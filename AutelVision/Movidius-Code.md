## Movidius 系统操作代码

指定代码区：
components/Alg/common/algCommDefine.h
#define CMX_DMA          __attribute__((section(".cmx.cdmaDescriptors")))
#define CMX_NEW_DMA      // __attribute__((section(".cmx_direct.bss")))
#define CMX_DIRECT_DATA  __attribute__((section(".cmx_direct.data")))
#define CMX_DATA_GLOBAL  __attribute__((section(".cmx.data.global")))
#define DDR_BSS          __attribute__((section(".ddr.bss")))
#define DDR_BSS_UNCACHED __attribute__((section(".ddr_direct.bss")))

shared/common_def.h
#define DDR_AREA            SECTIONT_DDR_CACHED_BSS
#define DDR_UNCACHED        SECTIONT_DDR_UNCACHED_BSS
#define DDR_DIRECT_DAT      SECTIONT_DDR_UNCACHED_DAT
#define LOS_LRT_CMX_DAT     SECTIONT_CMX_UNCACHED_DAT
#define CMX_DMA_DESCRIPTORS SECTIONT_CMX_DMA_DESCRIPTORS

components/shared/section.h
#define SECTIONT_CMX_DMA_DESCRIPTORS	__attribute__((section(".cmx.cdmaDescriptors")))
#define SECTIONT_CMX_UNCACHED_BSS       __attribute__((section(".cmx_direct.bss")))
#define SECTIONT_CMX_UNCACHED_DAT       __attribute__((section(".cmx_direct.data")))
#define SECTIONT_DDR_UNCACHED_BSS       __attribute__((section(".ddr_direct.bss")))
#define SECTIONT_DDR_UNCACHED_DAT       __attribute__((section(".ddr_direct.data")))
#define SECTIONT_DDR_CACHED_BSS         __attribute__((section(".ddr.bss")))
#define SECTIONT_DDR_CACHED_DAT         __attribute__((section(".ddr.data")))

对齐：
ALIGNED(8):
#define ALIGNED(x) __attribute__((aligned(x)))

计时：
DrvTimerStartTicksCount(&tic2); //开始计时
DrvTimerGetElapsedTicks(&tic_msg_start, &tic_msg_end); //结束之前的计时
DrvTimerTicksToMs(ticN); //把tic转成ms。

异步启动模块：
swcResetShave(LANDING_SHV_ID);
swcSetAbsoluteDefaultStack(LANDING_SHV_ID);
swcStartShaveAsyncCC(LANDING_SHV_ID, pLandingShaveEnt,
                       (irq_handler)&hdlLandingShaveEnd, "ii", &stParamIn,
                       &stParamOut);
最后一个函数的说明：
/// @brief Write the value to a IRF/SRF/VRF Registers from a specific Shave
/// @param[in] shave_num - shave number to read T-Register value from
/// @param[in] pc       - function called from the pc
/// @param[in] function - function to call when shave finished execution
/// @param[in] *fmt     - string containing i, s, or v according to irf, srf or vrf ex. "iisv"
/// @param[in] ...      - variable number of params according to fmt
/// @return void

缓存使用：
//清空L2缓存，参数是partition ID
DrvShaveL2CachePartitionFlushAndInvalidate(0);

//清空Leon数据缓存
swcLeonDataCacheFlush();

停止SHAVE:
SHAVE_HALT




