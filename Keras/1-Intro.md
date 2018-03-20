**结构**

- Layers
- activate functions
- loss functions
- Optimizer 

**模型以及参数的序列化和反序列化**

![odel-save-loa](/home/autel/Notes/Keras/imgs/model-save-load.png)

**模型类型**

1. Sequential：就是forward模型，各层只是简单线性叠加。
2. Graph: 
   1. 在所有的output上进行优化
   2. 允许多个独立网络同时运行，最后归并
   3. 允许多个独立输入输出
   4. 有不同的merging layer

**问题点**

1. Load and save model 
2. early stop
3. history saving: fit 函数的返回就是history.
4. checkpointing: 也有对应的callback可以用
5. interactions with TensorBoard and Quiver: 可以用这两个可视化分析软件。

**组合模型的方式**

Keras有两种model，sequential model and functional model.

* sequential
  * 各module按照顺序添加，组成一个stack 或者queue
  ~~~
    model = Sequential()
    model.add(Dense(N_HIDDEN, input_shape=(784,)))
    model.add(Activation('relu'))
    model.add(Dropout(DROPOUT))
    model.add(Dense(N_HIDDEN))
    model.add(Activation('relu'))
    model.add(Dropout(DROPOUT))
    model.add(Dense(nb_classes))
    model.add(Activation('softmax'))
    model.summary()
  ~~~

* functional

  * 各module通过functional api添加，可以构造复杂的模型，比如多个output、共享layer等。

  * Model是可以直接call的，像layer一样: 

    ~~~python
    x = Input(shape=(784,))
    # This works, and returns the 10-way softmax we defined above.
    y = model(x)
    ~~~


**模型输入**

如果数据集无法一次性加载到内存， 则有两种方法：

1. 用model.train_on_batch(x, y), 或者model.test_on_batch(x, y)
2. 自己写一个data generator，然后用model.fit_generator(data_generator, steps_per_epoch, epochs)来训练。

参见mnist-cnn的例子。

**Early stop**

如果想在loss不再降的时候停止训练，可以使用early stop callback

```python
from keras.callbacks import EarlyStopping
early_stopping = EarlyStopping(monitor='val_loss', patience=2)
model.fit(x, y, validation_split=0.2, callbacks=[early_stopping])
```
**在开发环境禁用随机的方法，用于验证算法本身**

```python
import numpy as np
import tensorflow as tf
import random as rn

# The below is necessary in Python 3.2.3 onwards to
# have reproducible behavior for certain hash-based operations.
# See these references for further details:
# https://docs.python.org/3.4/using/cmdline.html#envvar-PYTHONHASHSEED
# https://github.com/keras-team/keras/issues/2280#issuecomment-306959926

import os
os.environ['PYTHONHASHSEED'] = '0'

# The below is necessary for starting Numpy generated random numbers
# in a well-defined initial state.

np.random.seed(42)

# The below is necessary for starting core Python generated random numbers
# in a well-defined state.

rn.seed(12345)

# Force TensorFlow to use single thread.
# Multiple threads are a potential source of
# non-reproducible results.
# For further details, see: https://stackoverflow.com/questions/42022950/which-seeds-have-to-be-set-where-to-realize-100-reproducibility-of-training-res

session_conf = tf.ConfigProto(intra_op_parallelism_threads=1, inter_op_parallelism_threads=1)

from keras import backend as K

# The below tf.set_random_seed() will make random number generation
# in the TensorFlow backend have a well-defined initial state.
# For further details, see: https://www.tensorflow.org/api_docs/python/tf/set_random_seed

tf.set_random_seed(1234)

sess = tf.Session(graph=tf.get_default_graph(), config=session_conf)
K.set_session(sess)

# Rest of code follows ...
```
**Callback**

在调用model.fit时可传入一个callback对象列表，可用于在训练开始、期间、结束时记录一些东西。