**类型**

1. 分类问题的准确率

   ~~~
   binary_accuracy: (mean accuracy rate across all predictions for binary classification problems)

   categorical_accuracy (mean accuracy rate across all predictions for multiclass classification problems)

   sparse_categorical_accuracy (useful for sparse targets)

   top_k_categorical_accuracy (success when the target class is within the top_k predictions provided)​
   ~~~

2. Hinge Loss: SVM用的，二分类LOSS

3. Class Loss: Cross-entropy loss

4. Error Loss: 回归问题用的LOSS，有

   * `mse (mean square error between predicted and target values)`
   * `rmse (root square error between predicted and target values)`
   * `mae (mean absolute error between predicted and target values)`
   * `mape (mean percentage error between predicted and target values)`
   * `msle (mean squared logarithmic error between predicted and target values)`


