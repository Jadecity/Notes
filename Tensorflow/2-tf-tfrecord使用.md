Five steps to build tfrecord dataset

1. Use `tf.python_io.TFRecordWriter` to open the tfrecord file and start writing.
2. Before writing into tfrecord file, the image data and label data should be
  converted into proper datatype. (byte, int, float)
3. Now the datatypes are converted into `tf.train.Feature`
4. Finally create an Example Protocol Buffer using `tf.Example` and use the
  converted features into it. Serialize the Example using `serialize()` function.
5. Write the serialized Example .

~~~python
import tensorflow as tf
import numpy as np
import glob
from PIL import Image

# Converting the values into features
# _int64 is used for numeric values
def _int64_feature(value):
return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))

# _bytes is used for string/char values
def _bytes_feature(value):
return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))

# Loading the location of all files - image dataset
# Considering our image dataset has apple or orange
# The images are named as apple01.jpg, apple02.jpg .. , orange01.jpg .. etc.
images = glob.glob('data/*.jpg')
tfrecord_filename = 'something.tfrecords'

# Initiating the writer and creating the tfrecords file.
writer = tf.python_io.TFRecordWriter(tfrecord_filename)

# Write to tfrecord file
for image in images[:1]:
    img = Image.open(image)
    img = np.array(img.resize((32,32)))
    label = 0 if 'apple' in image else 1
    feature = { 'label': _int64_feature(label),
    		    'image': _bytes_feature(img.tostring()) }

    # Create an example protocol buffer
	example = tf.train.Example(features=tf.train.Features(feature=feature))
    
    # Writing the serialized example.
    writer.write(example.SerializeToString())
    
writer.close()
~~~

----

**Read Back**

~~~python
import tensorflow as tf 
import glob


filenames = glob.glob('*.tfrecords')
filename_queue = tf.train.string_input_producer(
filenames)

reader = tf.TFRecordReader()
_, serialized_example = reader.read(filename_queue)
feature_set = { 'image': tf.FixedLenFeature([], tf.string),
			    'label': tf.FixedLenFeature([], tf.int64)}

features = tf.parse_single_example( serialized_example, features= feature_set
)

label = features['label']
~~~

