# ALSA和TinyALSA
# I ALSA
## 基础
- ALSA:Advanced Linux Sound Architecture；参考[ALSA](http://www.alsa-project.org)
- 架构：HW --> alsa-driver --> alsa-lib --> App；用户空间的 alsa-lib 对应用程序提供统一的API接口；
- 代码结构：ALSA代码位于 `/sound` 目录
	- 
## a. struct snd_card
- 定义：include/sound/core.h
	```c
	struct snd_card {
		int number;                  /* number of soundcard (index to snd_cards) */
		struct list_head devices;    /* 记录该声卡下所有的逻辑设备的链表，如PCM， control */
		strcut list_head controls;   /* 记录该声卡下所有的控制单元的链表 */
		void *private_data;          /* 声卡的私有数据 */
	};
	```
## b. 声卡创建
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
	```c
	int snd_card_register(struct snd_card *card)
	{
		/* 创建sysfs 下的设备 */
		if (!card->card_dev) {
			card->card_dev = device_create(sound_class, card->dev,
							MKDEV(0, 0), card,
							"card%i", card->number);
			if (IS_ERR(card->card_dev))
				card->card_dev = NULL;
		}

		/* 通过 snd_device_register_all()注册所有挂在该声卡下的逻辑设备
		 *	snd_device_register_all() 会遍历整个设备链表，调用设备链表中的
		 * register() 函数,
		 */
		if ((err = snd_device_register_all(card)) < 0)
			return err;

		/* 其他 */
		return 0;
	}
	```
## c. PCM设备创建
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
- `snd_pcm`是挂在`snd_card`下的一个 snd_device;
- `snd_pcm`中字段：steram[2],指向两个`smd_pcm_str` 结构，分别代表`playback stream`和`capture stream`;
- `snd_pcm_substream`是pcm的中间层的核心，绝大部分任务都是在substream中处理，
- pcm设备会在`snd_card`注册时，调用`snd_pcm_dev_register()`注册;

## d.Control接口
Control接口主要是让用户空间的应用（alsa-lib）可以访问和控制音频codec芯片中的多路开关；
```c
struct snd_kcontrol_new {
    snd_ctl_elem_iface_t iface;   /* control 的类型 */
    const unsigned char *name;    /* ASCII name of item */
    unsigned int index;           /* 保存该control在该卡中的编号 */
    unsigned int access;          /* 该control 的访问类型,每bit表示一种访问类型 */
    snd_kcontrol_info_t *info;    /* 回调函数 */
    snd_kcontrol_get_t *get;
    snd_kcontrol_put_t *put;
    unsigned long private_value;  /*   */
};
```
- control的命名：一般为 源 -- 方向 -- 功能；
- 访问标志Access Flags：保存control的访问类型，默认类型 `SNDRV_CTL_ELEM_ACCESS_READWRITE`;
- 回调函数：`info(),get(),put()`
	- `info()`:用于获取control的详细信息；
	- `get()`:用于读取control的当前值；
	- `put()`:用于把应用程序的控制值设置到control中；
# II ASoC（ALSA System on Chip）
建立在ALSA驱动基础上，为了更好的支持嵌入式处理器和移动设备中的音频codec的一套软件体系。已经被整合到：sound/soc。 ASoC不能单独存在，他建立在标准的ALSA驱动之上，和标准的ALSA驱动结合才能工作。
- 通常，嵌入式设备的音频系统被划分为板载硬件（Machine）、Soc（Platform）、Codec三大部分；
	- Machine:具体的某个机器、开发板、手机
	- Platform:一般指某个Soc平台，
	- Codec:指机器上的编解码器。
- Machine：关键结构体`snd_soc_card`，`snd_soc_dai_link`，`snd_soc_ops`，驱动在`sound/soc/soc-core.c`中;
	```c
	platform_device "soc-audio"  |   platform_driver  "soc-audio"
		^                        |
		|                        |
	snd_soc_card                 |
		^                        |
		|                        |
	sdn_soc_dai_link             |
		^                        |
		|                        |
	snd_soc_ops                  |
	```
	- `soc_probe() --> snd_soc_register_card() --> snd_soc_instantiate_card()`
	- 关键的几个链表：
	```c
	static DEFINE_MUTEX(client_mutex);   //保护链表的锁
	static LIST_HEAD(dai_list);          //snd_soc_dai 的链表 
	static LIST_HEAD(platform_list);     //snd_soc_platform 的链表
	static LIST_HEAD(codec_list);        //snd_soc_codec 的链表
	static LIST_HEAD(component_list);    //snd_soc_component 的链表
	``` 


# III TinyALSA
## 1. 
- Android系统上的音频框架：APP-->Farmwork-->Audio Lib --> HAL
	- APP：如各类音乐APP;
	- Farmwork:提供一系列的服务和API的接口;关于音频的有MediaPlayer 、MediaRecorder,Android系统还为我们控制音频系统提供了AudioManager、AudioService及AudioSystem类;
	- Audio Lib:AudioFlinger和AudioPolicyService。它们的代码放置在frameworks/av/services/audioflinger，生成的最主要的库叫做libaudioflinger;
	－HAL:硬件抽象层是AudioFlinger直接访问的对象;