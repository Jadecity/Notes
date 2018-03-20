设置checkpoint的方法

~~~
# save best model
checkpoint = ModelCheckpoint(filepath=os.path.join(MODEL_DIR, "model-{epoch:02d}.h5"))

model.fit(Xtrain, Ytrain, batch_size=BATCH_SIZE, nb_epoch=NUM_EPOCHS,
validation_split=0.1, callbacks=[checkpoint])
~~~

就是在训练的时候加上一个callback