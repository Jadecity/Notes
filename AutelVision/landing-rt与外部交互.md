//降落过程中运动控制，给这个变量里赋值控制指令，然后把gLandingControlCmdflag 置为1即可
extern volatile fmu_movidus_control_cmd_t gstLandingMvControlCmd; 

//控制指令执行标志，赋值为1则飞控执行指令
extern volatile uint32_t gLandingControlCmdflag;

//手柄的状态，有按钮、模式、摇杆、roll、pitch、yaw
extern volatile fmu_vision_rc_info_t gstFmuRcInfoForAll;

//声纳数据，有valid distance
extern volatile sonar_dat_t gstSonar_dat;

//矫正后的相机参数
extern volatile LOS_LRT_CMX_DAT stStereoRecti *gRectiParams;

//是否log bottom的标志
extern volatile u8 gRequestLogBottom;

//向bottom view拷贝数据，以用于log
extern volatile u8 *gBottomView;

//是否启用path plan的标志，当safe landing启动时，停止path plan（设置为0）
extern volatile uint32_t gIsPathplanRun;

//飞行模式，2为降落模式，也就是手柄按了智能降落
extern volatile int flightmode;

//强制降落标志，当APP有危险提示时，如果人为确认继续降落，这个标志为1
extern volatile uint16_t gForceLandingFlag;

//其他模块启用禁用的标志，每一位代表的含义尚不清楚
extern volatile uint32_t g_mov_func_enable_flag;

//向APP和手柄发送信息，配合gMvStatusReportReady使用，gMvStatusReportReady置为1则代表消息已经准备好
extern volatile mv_status_report_t gstMvStatusReportData;
extern volatile int gMvStatusReportReady;