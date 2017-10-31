# ALSA和TinyALSA
# I ALSA
## 1.基础
- ALSA:Advanced Linux Sound Architecture；参考[ALSA](http://www.alsa-project.org)
- 架构：HW --> alsa-driver --> alsa-lib --> App；用户空间的 alsa-lib 对应用程序提供统一的API接口；
- 代码结构：ALSA代码位于 `/sound` 目录
	- 
### a. struct snd_card
- 定义：include/sound/core.h
```c
struct snd_card {
	int number;                  /* number of soundcard (index to snd_cards) */
	struct list_head devices;    /* 记录该声卡下所有的逻辑设备的链表， */
	strcut list_head controls;   /* 记录该声卡下所有的控制单元的链表 */
	void *private_data;          /* 声卡的私有数据 */
};
```
### b. 声卡创建
- 1.通过 `snd_card_creats() `创建一个 `snd_card` 实例
	- index：声卡编号；
	- id：字符串，声卡的表示符号；
- 2.创建声卡的芯片专有数据：用于存放声卡的资源信息，如中断、io、dma等，存放于 `private_data`；然后把专有数据注册为声卡的低阶设备：`snd_device_new()` ,用于自动销毁内存;
- 3.初始化 `snd_card` 的其他字段；
- 4.创建声卡的功能部件，如PCM，TIMER，CONTROL等；然后存放于 `snd_card` 的 `struct list_head devices` 中；
	- 例如PCM设备：调用 `snd_pcm_new()` 参数：
	```c
	truct snd_card *card；  // 声卡
	const char *id；        // 名称
	int device;             // 设备号
	int playback_count;     // 
	int capture_count;      // 
	truct snd_pcm **rpcm;   // 返回的pcm
	```
	- 对于pcm设备，主要就是定义操作函数，创建stream,调用`snd_device_new`封装；
	```c
	struct snd_pcm {
		struct snd_card *card;    // 指向声卡
		struct list_head list;    
		int device;               // device number 
		struct snd_pcm_str streams[2];  //snd_pcm_new()中创建
	};
	```
   - 最后调用`snd_card_register()`注册
### c. PCM设备创建
```c
struct snd_pcm {
	struct snd_card *card;         // 指向声卡
	struct list_head list;         //
	int device;                    /* device number */
	unsigned int info_flags;
	unsigned short dev_class;
	unsigned short dev_subclass;
	char id[64];
	char name[80];
	struct snd_pcm_str streams[2]; // 两个stream, playback capture
	struct mutex open_mutex;
	wait_queue_head_t open_wait;
	void *private_data;
	void (*private_free) (struct snd_pcm *pcm);
	struct device *dev;            /* actual hw device this belongs to */
	bool internal;                 /* pcm is for internal use only */
#if defined(CONFIG_SND_PCM_OSS) || defined(CONFIG_SND_PCM_OSS_MODULE)
	struct snd_pcm_oss oss;
#endif
};
```
- `snd_pcm`是挂在`snd_card`下的一个snd_device;
- `snd_pcm`中字段：steram[2],指向两个`smd_pcm_str` 结构，分别代表`playback stream`和`capture stream`;
- `snd_pcm_substream`是pcm的中间层的核心，绝大部分任务都是在substream中处理，

# II TinyALSA
## 1. 
- Android系统上的音频框架：APP-->Farmwork-->Audio Lib --> HAL
	- APP：如各类音乐APP;
	- Farmwork:提供一系列的服务和API的接口;关于音频的有MediaPlayer 、MediaRecorder,Android系统还为我们控制音频系统提供了AudioManager、AudioService及AudioSystem类;
	- Audio Lib:AudioFlinger和AudioPolicyService。它们的代码放置在frameworks/av/services/audioflinger，生成的最主要的库叫做libaudioflinger;
	－HAL:硬件抽象层是AudioFlinger直接访问的对象;