## structure

1. define global parameter and flags using `tf.app.flags.DEFINE_*`
2. create global step: 这步是在做什么？
3. config dataset
4. get SSD network and anchors
5. load preprocessing function
6. Create a dataset provider and batches.
   7. preprocess inputs
7. define model and loss
8. build train tensor
9. config GPU and model saver
10. train