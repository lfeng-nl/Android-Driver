# Android Driver
## 1.Android C/C++ 层Log打印
- Android log 打印等级： Verbose,Debug,Info,Warn,Error。
- Info、Warn、Error等级的Log禁止作为普通的调试信息使用
- 需包含头文件`#include <cutils/log.h>`；
- 按打印等级分：ALOGV,ALOGD,ALOGI,ALOGW,ALOGE;
- 各等级log打印控制：
```c
#define LOG_NDEBUG 0		//打开ALOGV:  
#define LOG_NIDEBUG 0		//打开ALOGI：  
#define LOG_NDDEBUG 0 		//打开ALOGD： 
#define NDEBUG 0		//打开全部LOG：
```

## 2.