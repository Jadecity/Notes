## getBottomView STEPS
1. 判断是否开启了safe_landing标志，没有则reset_landing and return 0. resetLanding是清空Context 和paramIn的状态，turn on path planning，然后启用cspace和rada。
2. 如果有摇杆推高，并且已经判断安全，则停止降落过程，reset landing后设置速度为0，退出。等待APP的确认。
3. 如果APP发送了强制landing的命令，则reset landing。继续降落过程。
4. 如果飞行状态为降落，并且之前不是降落，则
	4.1. 等待左图像，
	4.2. 速度设为0,
	4.3. 停止pathPlaning；
	4.4. 禁cspace和rada；
5. 开启bottom camera perception.
6. 检查飞行状态，如果不是降落，reset landing,退出。
7. 如果已经是safe状态，退出。
8. 如果Context的状态是等待左图像，那那么获取左图像；设状态为等DISP_MAP
9. 如果Context的状态是等待DISP—MAP，则获取DISP——IMG；设状态为等左图像。把数据放在left_n_disp_data里面，返回1，退出。



## landing_closeout STEPS
1. 如果飞行模式不是降落，则reset_landing(),退出；
2. 如果safeLanding已经结束（gstLandingContext.is_finish），清理内存，计算时间。
CASE：
	SAFE_LAND:设置already_safe_switch.
	NOT_SAFE_LAND:停止飞机。
	UNCERTAIN_LAND:停止飞机。
	IN_PROGRESS: DO NOTHING.

## flightmode 取值
0:已经停止
1:
2:landing

##特别事项
1. 只看到flightmode的声明，没有看到具体的定义和赋值的地方呢？
	答：flightmode是外部共享变量，由飞控模块直接赋值，由遥控器实施控制。在外部的名称和算法内部的名称不同，因此全局代码搜索搜不到。
2. _alg_init注册的函数，只被调用一次；_alg_main注册的函数，在加电之后，被系统循环调用，直至关机。
