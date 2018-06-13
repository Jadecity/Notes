input_fn 的设定：

> A tuple (features, labels): Where `features` is a `Tensor` or a
>dictionary of string feature name to `Tensor` and `labels` is a
>`Tensor` or a dictionary of string label name to `Tensor`. Both
>`features` and `labels` are consumed by `model_fn`. They should
>satisfy the expectation of `model_fn` from inputs.

model_fn 的设定：

> model_fn: Model function. Follows the signature:
>
>         * Args:
>     
>           * `features`: This is the first item returned from the `input_fn`
>                  passed to `train`, `evaluate`, and `predict`. This should be a
>                  single `Tensor` or `dict` of same.
>           * `labels`: This is the second item returned from the `input_fn`
>                  passed to `train`, `evaluate`, and `predict`. This should be a
>                  single `Tensor` or `dict` of same (for multi-head models). If
>                  mode is `ModeKeys.PREDICT`, `labels=None` will be passed. If
>                  the `model_fn`'s signature does not accept `mode`, the
>                  `model_fn` must still be able to handle `labels=None`.
>           * `mode`: Optional. Specifies if this training, evaluation or
>                  prediction. See `ModeKeys`.
>           * `params`: Optional `dict` of hyperparameters.  Will receive what
>                  is passed to Estimator in `params` parameter. This allows
>                  to configure Estimators from hyper parameter tuning.
>           * `config`: Optional configuration object. Will receive what is passed
>                  to Estimator in `config` parameter, or the default `config`.
>                  Allows updating things in your `model_fn` based on
>                  configuration such as `num_ps_replicas`, or `model_dir`.
>     
>         * Returns:
>           `EstimatorSpec`

feature_column作用：

​	feature_column 定义了一个从原始数据到模型实际用到的数据之间的转换，例如one-hot encoding. 由于自定义的estimator不能接受一个feature_column参数，因此需要在param参数中传入，然后再由model_fn解析并显示地使用:

~~~python
# Feature columns describe how to use the input.
my_feature_columns = []
for key in train_x.keys():
    my_feature_columns.append(tf.feature_column.numeric_column(key=key))
    
# Define model function    
def my_model(
   features, # This is batch_features from input_fn
   labels,   # This is batch_labels from input_fn
   mode,     # An instance of tf.estimator.ModeKeys
   params):  # Additional configuration
    
   # Use `input_layer` to apply the feature columns.
    net = tf.feature_column.input_layer(features, params['feature_columns'])
    
# Create estimator    
classifier = tf.estimator.Estimator(
    model_fn=my_model,
    params={
        'feature_columns': my_feature_columns,
        # Two hidden layers of 10 nodes each.
        'hidden_units': [10, 10],
        # The model must choose between 3 classes.
        'n_classes': 3,
    })




~~~



