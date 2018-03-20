经常需要得到Top-k的元素的index，方法如下:

~~~python
top_k = 4
index = np.argpartition(prob_array, -top_k)[-top_k:]
top_k_probs = prob_array[index]
~~~

