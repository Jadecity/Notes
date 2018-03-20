##CMX
大小：2M
组成：可认为是16x128K
特别事项：
1. 每个SHAVE访问“自己的”local slice时，带宽更高，耗电更少。
2. SHAVE和slice的对应关系，就是按照编号来的，12以后的没有对应任何的SHAVE，可自由使用。
3. 访问自己slice内部的数据会比访问其他slice的数据更低功耗，因为不需要slice 间 routing。
