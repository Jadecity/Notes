tf.data API可用于对大型数据集行进抽象。

1. tf.data.Dataset

   两种方式创建

   1. From source tensors: `Dataset.from_tensor_slices`
   2. from serveral datasets, applying a transformation: `Dataset.batch()`

   Dataset提供了几个函数用于对数据逐个进行变换。

2. tf.data.Iterator: 提供了从Dataset中读取数据的主要方式（through Iterator.get_next()）.

   目前提供四种Iterator，复杂度从低到高

   * one-shot: 从dataset 读取一遍，再调用get_next时会报OutOfRangeError

   * initializable: 建立iterator时可先用placeholder建，然后在sess.run时feed真正数据。

     ~~~python
     max_value = tf.placeholder(tf.int64, shape=[])
     dataset = tf.data.Dataset.range(max_value)
     iterator = dataset.make_initializable_iterator()
     next_element = iterator.get_next()

     # Initialize an iterator over a dataset with 10 elements.
     sess.run(iterator.initializer, feed_dict={max_value: 10})
     value = sess.run(next_element)
     ~~~

     ​

   * reinitializable: 可在运行时再从不同的dataset初始化，比如先从train dataset初始化，然后训练完成转成validation dataset初始化。

     ~~~python
     iterator = tf.data.Iterator.from_structure(training_dataset.output_types, training_dataset.output_shapes)
     next_element = iterator.get_next()

     training_init_op = iterator.make_initializer(training_dataset)
     validation_init_op = iterator.make_initializer(validation_dataset)

     # Run 20 epochs in which the training dataset is traversed, followed by the
     # validation dataset.
       # Initialize an iterator over the training dataset.
       sess.run(training_init_op)
       sess.run(next_element)

       # Initialize an iterator over the validation dataset.
       sess.run(validation_init_op)
       sess.run(next_element)
     ~~~

   * feedable: 定义了一个可以指向不同dataset的iterator，在sess.run的时候，根据传入的不同dataset的iterator_handle来确定从哪个dataset来取数据。

     ~~~python
     # Define training and validation datasets with the same structure.
     training_dataset = tf.data.Dataset.range(100).map(
         lambda x: x + tf.random_uniform([], -10, 10, tf.int64)).repeat()
     validation_dataset = tf.data.Dataset.range(50)

     # A feedable iterator is defined by a handle placeholder and its structure. We
     # could use the `output_types` and `output_shapes` properties of either
     # `training_dataset` or `validation_dataset` here, because they have
     # identical structure.
     handle = tf.placeholder(tf.string, shape=[])
     iterator = tf.data.Iterator.from_string_handle(
         handle, training_dataset.output_types, training_dataset.output_shapes)
     next_element = iterator.get_next()

     # You can use feedable iterators with a variety of different kinds of iterator
     # (such as one-shot and initializable iterators).
     training_iterator = training_dataset.make_one_shot_iterator()
     validation_iterator = validation_dataset.make_initializable_iterator()

     # The `Iterator.string_handle()` method returns a tensor that can be evaluated
     # and used to feed the `handle` placeholder.
     training_handle = sess.run(training_iterator.string_handle())
     validation_handle = sess.run(validation_iterator.string_handle())

     # Loop forever, alternating between training and validation.
     while True:
       # Run 200 steps using the training dataset. Note that the training dataset is
       # infinite, and we resume from where we left off in the previous `while` loop
       # iteration.
       for _ in range(200):
         sess.run(next_element, feed_dict={handle: training_handle})

       # Run one pass over the validation dataset.
       sess.run(validation_iterator.initializer)
       for _ in range(50):
         sess.run(next_element, feed_dict={handle: validation_handle})
     ~~~

   Dataset一个经典的使用方式是，对jpg文件列表进行处理：

   ~~~python
   # Reads an image from a file, decodes it into a dense tensor, and resizes it
   # to a fixed shape.
   def _parse_function(filename, label):
     image_string = tf.read_file(filename)
     image_decoded = tf.image.decode_image(image_string)
     image_resized = tf.image.resize_images(image_decoded, [28, 28])
     return image_resized, label

   # A vector of filenames.
   filenames = tf.constant(["/var/data/image1.jpg", "/var/data/image2.jpg", ...])

   # `labels[i]` is the label for the image in `filenames[i].
   labels = tf.constant([0, 37, ...])

   dataset = tf.data.Dataset.from_tensor_slices((filenames, labels))
   dataset = dataset.map(_parse_function)

   ~~~

   可看出Dataset.from_tensor_slices的重要能力，通过map, filter等操作，可以将最初的数据集转成训练师实际使用的数据集。

