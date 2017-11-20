# Android Driver
## 1.Android C/C++ 层Log打印
- Android log 打印等级： Verbose，Debug，Info，Warn，Error。
- 按打印等级分：ALOGV,ALOGD,ALOGI,ALOGW,ALOGE;
- Info、Warn、Error等级的Log禁止作为普通的调试信息使用
- 使用方式
	- 包含头文件`#include <cutils/log.h>`；
	- 定义LOG_TAG宏：`#define LOG_TAG "audio_hw_primary"`
	- 在Android.mk中增加 `LOCAL_SHARED_LIBRARIES += libcutils `,动态库依赖
- 各等级log打印控制：
```c
#define LOG_NDEBUG 0		//打开ALOGV
#define LOG_NDDEBUG 0 		//打开ALOGD
#define LOG_NIDEBUG 0		//打开ALOGI
#define NDEBUG 0		//打开全部LOG
```

## 2.printk 打印控制
- 在头文件 <include/linux/kern_levls.h>中规定了预定的八种内核log级别
```c
#define KERN_EMERG	KERN_SOH "0"	/* 用于紧急事件消息*/
#define KERN_ALERT	KERN_SOH "1"	/* 用于需要立即采取行动的情况 */
#define KERN_CRIT	KERN_SOH "2"	/*  临界状态，通常涉及严重的硬件或软件操作失败 */
#define KERN_ERR	KERN_SOH "3"	/* 用于报告错误 */
#define KERN_WARNING	KERN_SOH "4"	/* warning conditions */
#define KERN_NOTICE	KERN_SOH "5"	/* 有必要进行提示的正常情况 */
#define KERN_INFO	KERN_SOH "6"	/* 提示性信息，例如驱动可以在启动的时候以这个级别来打印找到的硬件信息 */
#define KERN_DEBUG	KERN_SOH "7"	/* 用于调试 */

#define KERN_DEFAULT	KERN_SOH "d"	/* the default kernel loglevel */

```
- 内核对打印级别的控制：`cat /proc/sys/kernel/printk/` 可以查看，默认值受一般为：
	> 7 4 1 7  
	> 依次为：  
	> 7：控制台日志级别，优先级高于该级别的（不包括），被打印至控制台，例如串口；  
	> 4：默认消息级别，没有指定消息级别的消息，会被赋予此权限；  
	> 1：控制台级别可被设置的最小值；  
	> 7：控制台级别的缺省值；
- 以上值由`kernel/printk.c`中：
```c
/* printk's without a loglevel use this.. */
/* CONFIG_DEFAULT_MESSAGE_LOGLEVEL 一般会在默认配置文件中定义 */
#define DEFAULT_MESSAGE_LOGLEVEL CONFIG_DEFAULT_MESSAGE_LOGLEVEL 

/* We show everything that is MORE important than this.. */
#define MINIMUM_CONSOLE_LOGLEVEL 1 /* Minimum loglevel we let people use */
#define DEFAULT_CONSOLE_LOGLEVEL 7 /* anything MORE serious than KERN_DEBUG */

int console_printk[4] = {
	DEFAULT_CONSOLE_LOGLEVEL,	/* console_loglevel */
	DEFAULT_MESSAGE_LOGLEVEL,	/* default_message_loglevel */
	MINIMUM_CONSOLE_LOGLEVEL,	/* minimum_console_loglevel */
	DEFAULT_CONSOLE_LOGLEVEL,	/* default_console_loglevel */
};
```
- dmesg:好像不受以上级别控制，都会打印；
## 3.dev_dbg
- dev_dbg：内核中我们常用`dev_dbg`来控制输出，这个函数实质是调用 `printk(KERN_DEBUG)`来打印输出； 
- 使用方式：1.包含头文件`<linux/device.h>`或 `<linux/platform_device.h>`, 2.在包含该头文件之前，使用`#define DEBUG 1`来打开开关；
- dev_dbg参数:第一个为`struct device *dev`类型,可以利用`dev->name`来检索log信息，第二个为需要打印的信息；
```c
#define dev_info(dev, fmt, arg...) _dev_info(dev, fmt, ##arg)

#if defined(CONFIG_DYNAMIC_DEBUG)
#define dev_dbg(dev, format, ...)		     \
do {						     \
	dynamic_dev_dbg(dev, format, ##__VA_ARGS__); \
} while (0)

#elif defined(DEBUG)

#define dev_dbg(dev, format, arg...)		\
	dev_printk(KERN_DEBUG, dev, format, ##arg)
#else
#define dev_dbg(dev, format, arg...)				\
({								\
	if (0)							\
		dev_printk(KERN_DEBUG, dev, format, ##arg);	\
	0;							\
})

#endif
```
## 4.跟打印log相关的预定义宏和预定义标识符
```c
/* 预定义宏 */
__DATE__    //运行预处理的时间
__FILE__    //当前源文件的名字
__LINE__    //当前源文件代码中的行号的整数常量
__TIME__    //源文件的编译时间，格式为"hh: mm: ss"
/* 预定义标识符 */
__func__    //代表为 "函数名" 的字符串
```