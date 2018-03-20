在transfer learning中，需要使用已经训练好的模型进行fine-tune。
以inception-v3 为例，在keras中可以这么做:
先加载inception-v3模型：
~~~python
from keras.applications.inception_v3 import InceptionV3
from keras.preprocessing import image
from keras.models import Model
from keras.layers import Dense, GlobalAveragePooling2D
from keras import backend as K

# create the base pre-trained model
base_model = InceptionV3(weights='imagenet', include_top=False)
~~~
没有包含top 是为了重新训练全连接层。

然后加上新的后几层:
~~~python
# add a global spatial average pooling layer
x = base_model.output
x = GlobalAveragePooling2D()(x)# let's add a fully-connected layer as first layer

x = Dense(1024, activation='relu')(x)# and a logistic layer with 200 classes as last layer
predictions = Dense(200, activation='softmax')(x)# model to train
model = Model(input=base_model.input, output=predictions)
~~~

然后冻结pre-trained好的卷积层：

~~~python
# that is, freeze all convolutional InceptionV3 layers
for layer in base_model.layers: layer.trainable = False
~~~

最后模型重新编译，然后训练
~~~python
# we use SGD with a low learning rate
from keras.optimizers import SGD
model.compile(optimizer=SGD(lr=0.0001, momentum=0.9), loss='categorical_crossentropy')

# we train our model again (this time fine-tuning the top 2 inception blocks)
# alongside the top Dense layers
model.fit_generator(...)
~~~