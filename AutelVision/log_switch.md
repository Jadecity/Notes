要打开下视图像在landing过程的log，需要修改下面两个开关为1

1. shared/common_def.h: LOG_BOTTOM_FOR_LANDING
2. co. mponents/alg/landing/landing_common_def.h: LOG_SMART_LANDING
3. stereo_logger.c:  saveBottomView2SD函数
取消注释：MV_LOG("writing %d image\n", bottom_index);
