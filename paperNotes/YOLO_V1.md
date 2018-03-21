流程上，端对端训练。

输入：整幅图像
输出：bbox和类别

算法设计：
把图像分成SxS的格子，如果一个物体的中心落在哪个格子里，那个格子就负责检测这个物体。 
每个bbox有五个predictons:x, y, w, h, confidents. x, y是物体中心相对于grid cell的位置。w, h是相对于图像的宽高。
每个grid predicts C 个类别的条件概率， regardless of the number  of bbox B。
在VOC数据集，S=7, B=2，C=20。
x, y, w, h都是0-1的，因为都是相对值。

结构：
24个conv
2个dense

激活函数：最后一层：线性激活函数; 钱面，都用leaky rectified linear activation function.
LOSS: sum-squared error. 而且最后bbox的宽高，是square root之后的。

参数：
batch size:64
momentum: 0.9
decay: 0.0005
learning rate: 在不同阶段，用不同的Learning rate.

训练了1周。
