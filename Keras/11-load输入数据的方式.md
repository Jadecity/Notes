输入数据，有两种方式

1. 一次性将数据load到内存
2. 批次将数据load到内存
   1. 自己写一个loadData的函数，循环调用；
   2. 使用TF提供的dataset模块，能够充分利用TF的queue runner模型。

推荐使用2.2的方式。

实例代码：

~~~python
# Create the dataset and its associated one-shot iterator.
dataset = Dataset.from_tensor_slices((x_train, y_train))
dataset = dataset.repeat()
dataset = dataset.shuffle(buffer_size)
dataset = dataset.batch(batch_size)
iterator = dataset.make_one_shot_iterator()

# Model creation using tensors from the get_next() graph node.
inputs, targets = iterator.get_next()
~~~

