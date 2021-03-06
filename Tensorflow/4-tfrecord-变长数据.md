~~~python
变长tfrecord Writer
# -*- coding:utf-8 -*- 

import tensorflow as tf
import numpy as np
from tqdm import tqdm

'''tfrecord 写入序列数据，每个样本的长度不固定。
和固定 shape 的数据处理方式类似，前者使用 tf.train.Example() 方式，而对于变长序列数据，需要使用 
tf.train.SequenceExample()。 在 tf.train.SequenceExample() 中，又包括了两部分：
context 来放置非序列化部分；
feature_lists 放置变长序列。

refer: 
https://github.com/tensorflow/magenta/blob/master/magenta/common/sequence_example_lib.py
https://github.com/dennybritz/tf-rnn
http://leix.me/2017/01/09/tensorflow-practical-guides/
https://github.com/siavash9000/im2txt_demo/blob/master/im2txt/im2txt/ops/inputs.py
'''

# **1.创建文件
writer1 = tf.python_io.TFRecordWriter('../../data/seq_test1.tfrecord')
writer2 = tf.python_io.TFRecordWriter('../../data/seq_test2.tfrecord')

# 非序列数据
labels = [1, 2, 3, 4, 5, 1, 2, 3, 4]
# 长度不固定的序列
frames = [[1], [2, 2], [3, 3, 3], [4, 4, 4, 4], [5, 5, 5, 5, 5],
          [1], [2, 2], [3, 3, 3], [4, 4, 4, 4]]


writer = writer1
for i in tqdm(xrange(len(labels))):  # **2.对于每个样本
    if i == len(labels) / 2:
        writer = writer2
        print('\nThere are %d sample writen into writer1' % i)
    label = labels[i]
    frame = frames[i]
    # 非序列化
    label_feature = tf.train.Feature(int64_list=tf.train.Int64List(value=[label]))
    # 序列化
    frame_feature = [
        tf.train.Feature(int64_list=tf.train.Int64List(value=[frame_])) for frame_ in frame
    ]

    seq_example = tf.train.SequenceExample(
        # context 来放置非序列化部分
        context=tf.train.Features(feature={
            "label": label_feature
        }),
        # feature_lists 放置变长序列
        feature_lists=tf.train.FeatureLists(feature_list={
            "frame": tf.train.FeatureList(feature=frame_feature),
        })
    )

    serialized = seq_example.SerializeToString()
    writer.write(serialized)  # **4.写入文件中

print('Finished.')
writer1.close()
writer2.close()
~~~


~~~python
变长tfrecord Reader
# -*- coding:utf-8 -*-

import tensorflow as tf
import math

QUEUE_CAPACITY = 100
SHUFFLE_MIN_AFTER_DEQUEUE = QUEUE_CAPACITY // 5

"""
读取变长序列数据。
和固定shape的数据读取方式不一样，在读取变长序列中，我们无法使用 tf.train.shuffle_batch() 函数，只能使用
tf.train.batch() 函数进行读取，而且，在读取的时候，必须设置 dynamic_pad 参数为 True, 把所有的序列 padding
到固定长度（该batch中最长的序列长度），padding部分为 0。
此外，在训练的时候为了实现 shuffle 功能，我们可以使用 RandomShuffleQueue 队列来完成。详见下面的 _shuffle_inputs 函数。
"""


def _shuffle_inputs(input_tensors, capacity, min_after_dequeue, num_threads):
    """Shuffles tensors in `input_tensors`, maintaining grouping."""
    shuffle_queue = tf.RandomShuffleQueue(
        capacity, min_after_dequeue, dtypes=[t.dtype for t in input_tensors])
    enqueue_op = shuffle_queue.enqueue(input_tensors)
    runner = tf.train.QueueRunner(shuffle_queue, [enqueue_op] * num_threads)
    tf.train.add_queue_runner(runner)

    output_tensors = shuffle_queue.dequeue()

    for i in range(len(input_tensors)):
        output_tensors[i].set_shape(input_tensors[i].shape)

    return output_tensors


def get_padded_batch(file_list, batch_size, num_enqueuing_threads=4, shuffle=False):
    """Reads batches of SequenceExamples from TFRecords and pads them.
    Can deal with variable length SequenceExamples by padding each batch to the
    length of the longest sequence with zeros.
    Args:
      file_list: A list of paths to TFRecord files containing SequenceExamples.
      batch_size: The number of SequenceExamples to include in each batch.
      num_enqueuing_threads: The number of threads to use for enqueuing
          SequenceExamples.
      shuffle: Whether to shuffle the batches.
    Returns:
      labels: A tensor of shape [batch_size] of int64s.
      frames: A tensor of shape [batch_size, num_steps] of floats32s. note that
          num_steps is the max time_step of all the tensors.
    Raises:
      ValueError: If `shuffle` is True and `num_enqueuing_threads` is less than 2.
    """
    file_queue = tf.train.string_input_producer(file_list)
    reader = tf.TFRecordReader()
    _, serialized_example = reader.read(file_queue)

    context_features = {
        "label": tf.FixedLenFeature([], dtype=tf.int64)
    }
    sequence_features = {
        "frame": tf.FixedLenSequenceFeature([], dtype=tf.int64)
    }

    context_parsed, sequence_parsed = tf.parse_single_sequence_example(
        serialized=serialized_example,
        context_features=context_features,
        sequence_features=sequence_features
    )

    labels = context_parsed['label']
    frames = sequence_parsed['frame']
    input_tensors = [labels, frames]

    if shuffle:
        if num_enqueuing_threads < 2:
            raise ValueError(
                '`num_enqueuing_threads` must be at least 2 when shuffling.')
        shuffle_threads = int(math.ceil(num_enqueuing_threads) / 2.)

        # Since there may be fewer records than SHUFFLE_MIN_AFTER_DEQUEUE, take the
        # minimum of that number and the number of records.
        min_after_dequeue = count_records(
            file_list, stop_at=SHUFFLE_MIN_AFTER_DEQUEUE)
        input_tensors = _shuffle_inputs(
            input_tensors, capacity=QUEUE_CAPACITY,
            min_after_dequeue=min_after_dequeue,
            num_threads=shuffle_threads)

        num_enqueuing_threads -= shuffle_threads

    tf.logging.info(input_tensors)
    return tf.train.batch(
        input_tensors,
        batch_size=batch_size,
        capacity=QUEUE_CAPACITY,
        num_threads=num_enqueuing_threads,
        dynamic_pad=True,
        allow_smaller_final_batch=False)


def count_records(file_list, stop_at=None):
    """Counts number of records in files from `file_list` up to `stop_at`.
    Args:
      file_list: List of TFRecord files to count records in.
      stop_at: Optional number of records to stop counting at.
    Returns:
      Integer number of records in files from `file_list` up to `stop_at`.
    """
    num_records = 0
    for tfrecord_file in file_list:
        tf.logging.info('Counting records in %s.', tfrecord_file)
        for _ in tf.python_io.tf_record_iterator(tfrecord_file):
            num_records += 1
            if stop_at and num_records >= stop_at:
                tf.logging.info('Number of records is at least %d.', num_records)
                return num_records
    tf.logging.info('Total records: %d', num_records)
    return num_records


if __name__ == '__main__':
    tfrecord_file_names = ['../../data/seq_test1.tfrecord', '../../data/seq_test2.tfrecord']
    label_batch, frame_batch = get_padded_batch(tfrecord_file_names, 10, shuffle=True)
    config = tf.ConfigProto()
    config.gpu_options.allow_growth = True
    sess = tf.Session(config=config)
    tf.train.start_queue_runners(sess=sess)
    for i in xrange(3):
        _frames_batch, _label_batch = sess.run([frame_batch, label_batch])
        print('** batch %d' % i)
        print(_label_batch)
        print(_frames_batch)
~~~

关于padded batch: 参考https://stackoverflow.com/questions/45955241/how-do-i-create-padded-batches-in-tensorflow-for-tf-train-sequenceexample-data-u

一个变长sequence tfrecord的padding_batch实例：

~~~python
写入和批量读取tfrecord样本，注意：padded_batch的参数shape，需要每个字段的padding shape，不需要paddding的字段也必须给出一个shape， [None]就行。

# -*- coding:utf-8 -*- 

import os
import tensorflow as tf
import json
import tensorlayer.visualize as vis
import numpy as np


def _int64List_feature(value):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=value.flatten()))

def _int64_feature(value):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))

def _int64_feature_list(value):
    """
    Return a list of feature.
    :param value: np array, shape[n, k]
    """
    return [_int64List_feature(v) for v in value]

# _bytes is used for string/char values
def _bytes_feature(value):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))



def test_tfrecord_create(repeat_num):
    writer = tf.python_io.TFRecordWriter('/tmp/tmp.tfrecords')

    for i in range(repeat_num):
        img_size = np.array([40, 50])
        if i == 0:
            bboxes = np.array([[1,2,3], [4,5,6]])
        else:
            bboxes = np.array([[1, 2, 3]])

        context_feature = {
            'size': _int64List_feature(img_size)
        }

        varlen_feature = {
            'bboxes': tf.train.FeatureList(feature=_int64_feature_list(bboxes))
        }

        # Create an example protocol buffer
        example = tf.train.SequenceExample(context=tf.train.Features(feature=context_feature),
                                           feature_lists=tf.train.FeatureLists(feature_list=varlen_feature))

        # Writing the serialized example.
        writer.write(example.SerializeToString())
    writer.close()

def test_tfrecord_decode():
    file_queue = tf.train.string_input_producer(['/tmp/tmp.tfrecords'])
    reader = tf.TFRecordReader()
    _, example = reader.read(file_queue)

    context_feature = {
        'size': tf.FixedLenFeature([2], dtype=tf.int64)
    }

    varlen_feature = {
        'bboxes': tf.FixedLenSequenceFeature([3], dtype=tf.int64)
    }

    context_parsed, sequence_parsed = tf.parse_single_sequence_example(
        serialized=example,
        context_features=context_feature,
        sequence_features=varlen_feature
    )

    labels = context_parsed['size']
    frames = sequence_parsed['bboxes']

    return labels, frames


def parse_func(example):
    """
    Parse tfrecord example.
    :param exam: one example instance
    :return: image, size, labels, bboxes
    """
    context_feature = {
        'size': tf.FixedLenFeature([2], dtype=tf.int64)
    }

    varlen_feature = {
        'bboxes': tf.FixedLenSequenceFeature([3], dtype=tf.int64)
    }

    context_parsed, sequence_parsed = tf.parse_single_sequence_example(
        serialized=example,
        context_features=context_feature,
        sequence_features=varlen_feature
    )

    labels = context_parsed['size']
    frames = sequence_parsed['bboxes']

    return labels, frames

if __name__ == '__main__':
    test_tfrecord_create(10)

    #
    # labels, frames = test_tfrecord_decode()
    # with tf.Session() as sess:
    #     tf.train.start_queue_runners(sess=sess)
    #
    #     labels_np, frames_np = sess.run([labels, frames])
    #     print(labels_np)
    #     print(frames_np)

    # test TFRecordDataset
    rcd_files = ['/tmp/tmp.tfrecords']

    dataset = tf.data.TFRecordDataset(rcd_files)
    dataset = dataset.map(map_func=parse_func)
    # dataset = dataset.batch(2)
    dataset = dataset.padded_batch(2, padded_shapes=([None], tf.TensorShape([None,3])))
    itr = dataset.make_one_shot_iterator()
    with tf.Session() as ss:
        print(ss.run(itr.get_next()))
~~~

