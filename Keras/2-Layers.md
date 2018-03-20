1. Regular dense layer：全连接层

   ~~~python
   keras.layers.core.Dense(units, activation=None, use_bias=True, kernel_initializer='glorot_uniform', bias_initia...
   ~~~

2. Conv and Pooling layer:卷积和pooling层

   ~~~python
   keras.layers.convolutional.Conv1D(filters, kernel_size, strides=1, padding='valid', dilation_rate=1, activation=
                                     
   keras.layers.convolutional.Conv2D(filters, kernel_size, strides=(1, 1), padding='valid', data_format=None, dilat
                                     
   keras.layers.pooling.MaxPooling1D(pool_size=2, strides=None, padding='valid')
                                     
   keras.layers.pooling.MaxPooling2D(pool_size=(2, 2), strides=None, padding='valid', data_format=None)
   ~~~

3. Regularization:正则化层

   ~~~
   kernel_regularizer:正则化权重矩阵
   bias_regularizer:正则化bias
   activity_regularizer:正则化激活函数

   #Dropout
   keras.layers.core.Dropout(rate, noise_shape=None, seed=None)
   ~~~

4. 归一化:Batch Norm

   ~~~
   keras.layers.normalization.BatchNormalization(axis=-1, momentum=0.99, epsilon=0.001, center=True, scale=True, ...
   ~~~

   ​

