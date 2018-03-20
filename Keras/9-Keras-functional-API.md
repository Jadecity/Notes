## Keras functional API

一个例子：

Sequential model:

~~~python
from keras.models import Sequential
from keras.layers.core import dense, Activation
model = Sequential([
    dense(32, input_dim=784),
    Activation("sigmoid"),
    dense(10),
    Activation("softmax"),
])
model.compile(loss="categorical_crossentropy", optimizer="adam")
~~~

同样模型的Functional API实现

~~~python
from keras.layers import Input
from keras.layers.core import dense
from keras.models import Model
from keras.layers.core import Activation

inputs = Input(shape=(784,))
x = dense(32)(inputs)
x = Activation("sigmoid")(x)
x = dense(10)(x)
predictions = Activation("softmax")(x)
model = Model(inputs=inputs, outputs=predictions)
model.compile(loss="categorical_crossentropy", optimizer="adam")
~~~

可以认为functional API是seq API的实现基础。

以下三种模型只能用functional API实现：

* Models with multiple inputs and outputs
* Models composed of multiple submodels
* Models that used shared layers

**多个输入输出**

~~~python
直接把输入和输出作为列表传进Model的参数
model = Model(inputs=[input1, input2], outputs=[output1, output2])
~~~

**Customizing Keras**

如果要新加自己的层，或者用不同的距离函数，LOSS等，就得自己写，底层就是调TF或者Theano的东西。