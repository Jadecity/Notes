保存模型训练时的log数据

```
keras.callbacks.TensorBoard(log_dir='./logs', histogram_freq=0,
write_graph=True, write_images=False)
```

使用TensorBoard可视化

~~~shell
tensorboard --logdir=/full_path_to_your_logs
~~~

