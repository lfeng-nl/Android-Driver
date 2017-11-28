# ALSA和TinyALSA
## I ALSA
### 基础
- ALSA:Advanced Linux Sound Architecture；参考[ALSA](http://www.alsa-project.org)
- 架构：HW --> alsa-driver --> alsa-lib --> App；用户空间的 alsa-lib 对应用程序提供统一的API接口；
- 代码结构：ALSA代码位于 `/sound` 目录
### 1. snd_card
- 定义：include/sound/core.h
  ```c
  struct snd_card {
  	int number;                  /* number of soundcard (index to snd_cards) */
  	struct list_head devices;    /* 记录该声卡下所有的逻辑设备的链表，如PCM， control */
  	strcut list_head controls;   /* 记录该声卡下所有的控制单元的链表 */
  	void *private_data;          /* 声卡的私有数据 */
  };
  ```
### 2.声卡创建
- 1.通过`snd_card_create`创建一个 `snd_card` 实例
  ```c
  int snd_card_create(int idx, const char *xid,struct module *module, int extra_size,struct snd_card **card_ret)
  ```
  - idx：声卡编号；
  - xid：字符串，声卡的表示符号；

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
### 3. PCM设备创建
```c
struct snd_pcm {
	struct snd_card *card;         // 指向声卡
	struct list_head list;         
	int device;                    // device number 
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
};
```
- `snd_pcm`是挂在`snd_card`下的一个 snd_device;
- `snd_pcm`中字段：steram[2],指向两个`smd_pcm_str` 结构，分别代表`playback stream`和`capture stream`;
- `snd_pcm_substream`是pcm的中间层的核心，绝大部分任务都是在substream中处理，
- pcm设备会在`snd_card`注册时，调用`snd_pcm_dev_register()`注册;

### 4.Control接口
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
## II ASoC (ALSA System on Chip)
建立在ALSA驱动基础上，为了更好的支持嵌入式处理器和移动设备中的音频codec的一套软件体系。已经被整合到：sound/soc。 ASoC不能单独存在，他建立在标准的ALSA驱动之上，和标准的ALSA驱动结合才能工作。
- 通常，嵌入式设备的音频系统被划分为板载硬件（Machine）、Soc（Platform）、Codec三大部分；
  - Machine:具体的某个机器、开发板、手机
  - Platform:一般指某个Soc平台，
  - Codec:指机器上的编解码器。
###1.Platform

###2.Codec

作用：进行数模转换、对音频通路进行控制、对音频增益进行控制；

关键结构体：`snd_soc_codec`,`snd_soc_codec_driver`, `snd_soc_dai`；

#### a.codec注册过程

以`Platform driver`为载体，在`probe`中调用`snd_soc_register_codec()` 传入关键结构体 `snd_soc_codec_driver` 和`snd_soc_dai_driver `；在注册函数中申请`snd_soc_codec` 。

```c
static struct snd_soc_dai_ops lc1160_dai_ops = {
	.shutdown		= lc1160_shutdown,
	.hw_params		= lc1160_hw_params,
	.set_sysclk 	= lc1160_set_dai_sysclk,
	.set_fmt		= lc1160_set_dai_fmt,
};
static struct snd_soc_dai_driver lc1160_dai[] = {
{
	.name = "comip_hifi",
	.playback = {
		.stream_name = "Playback",
		.channels_min = 1,
		.channels_max = 2,
		.rates = COMIP_1160_RATES,
		.formats = COMIP_1160_FORMATS,
	},
	.capture = {
		.stream_name = "Capture",
		.channels_min = 1,
		.channels_max = 2,
		.rates = COMIP_1160_RATES,
		.formats = COMIP_1160_FORMATS,
	},
	.ops = &lc1160_dai_ops,
},
  ...
};
static struct snd_soc_codec_driver soc_codec_dev_lc1160 = {
	.probe =	lc1160_probe,
	.remove =	lc1160_remove,
	.read = lc1160_read_reg_cache,
	.write = lc1160_write,
	.set_bias_level = lc1160_set_bias_level,
	.reg_cache_size = sizeof(lc1160_codec_reg),
	.reg_word_size = sizeof(u8),
	.reg_cache_default = lc1160_codec_reg,
	.ignore_pmdown_time = true,
	.controls = lc1160_snd_controls,
	.num_controls = ARRAY_SIZE(lc1160_snd_controls),
	.dapm_widgets = lc1160_dapm_widgets,
	.num_dapm_widgets = ARRAY_SIZE(lc1160_dapm_widgets),
	.dapm_routes = intercon,
	.num_dapm_routes = ARRAY_SIZE(intercon),
};
```

snd_soc_register_codec()过程分析:

```c
int snd_soc_register_codec(struct device *dev,
			   const struct snd_soc_codec_driver *codec_drv,
			   struct snd_soc_dai_driver *dai_drv,
			   int num_dai)
{
 	codec = kzalloc(sizeof(struct snd_soc_codec), GFP_KERNEL); //实例化snd_soc_codec
	codec->name = fmt_single_name(dev, &codec->id);
	codec->write = codec_drv->write;
	codec->read = codec_drv->read;
	...//初始化各个字段
    list_add(&codec->list, &codec_list); //将codec添加到list
    snd_soc_register_dais(dev, dai_drv, num_dai);  //对codec dai进行注册  
}

```

snd_soc_register_dais() 过程分析：

```c
static int snd_soc_register_dais(struct device *dev,
		struct snd_soc_dai_driver *dai_drv, size_t count)
{
	struct snd_soc_codec *codec;
	struct snd_soc_dai *dai;
	int i, ret = 0;
	for (i = 0; i < count; i++) {
        /* 实例化 dai */
		dai = kzalloc(sizeof(struct snd_soc_dai), GFP_KERNEL);
		/* dai 初始化 */
		dai->name = fmt_multiple_name(dev, &dai_drv[i]);
		dai->dev = dev;
		dai->driver = &dai_drv[i];
		...
          
		mutex_lock(&client_mutex);

		list_for_each_entry(codec, &codec_list, list) {
			if (codec->dev == dev) {
				/* 匹配codec */
				dai->codec = codec;
				break;
			}
		}
		/* 将dai 加入链表 */
		list_add(&dai->list, &dai_list);
	}
	return 0;
}

```

#### b.codec_drv->probe() 的调用 

在machine驱动的初始化，codec和dai的3注册都会调用snd_soc_instante_catds() 进行一次声卡和codec，dai，platform，的匹配绑定；按名字进行匹配，把匹配到的codec，dai，platform实例化赋值给声卡没对dai的snd_soc_runtime变量中。一旦绑定成功，将会使得codec和dai驱动的probe回调被调用。

### 3.Machine：
关键结构体`snd_soc_card`，`snd_soc_dai_link`，`snd_soc_ops`，作为platform_device的设备私有数据下，驱动在`sound/soc/soc-core.c`中;
- `soc_probe() --> snd_soc_register_card() --> snd_soc_instantiate_card()`
- 关键的几个链表：
    ```c
    static DEFINE_MUTEX(client_mutex);   //保护链表的锁
    static LIST_HEAD(dai_list);          //snd_soc_dai 的链表 
    static LIST_HEAD(platform_list);     //snd_soc_platform 的链表
    static LIST_HEAD(codec_list);        //snd_soc_codec 的链表
    static LIST_HEAD(component_list);    //snd_soc_component 的链表
    ```
- snd_soc_register_card解析
  ```c
  int snd_soc_register_card(struct snd_soc_card *card)
  {
  	for (i = 0; i < card->num_links; i++) {
  		struct snd_soc_dai_link *link = &card->dai_link[i];
  		/* 检查link 中的各项成员名字 */
  	}

  	dev_set_drvdata(card->dev, card);
  	/* 初始化card中的各个链表 */
  	snd_soc_initialize_card_lists(card);
  	/* 实例化rtd[]  num_links+num_aux_devs */
  	card->rtd = devm_kzalloc(card->dev,
  				 sizeof(struct snd_soc_pcm_runtime) *
  				 (card->num_links + card->num_aux_devs),
  				 GFP_KERNEL);
    	/* red 一些成员初始化 */
  	card->num_rtd = 0;
  	card->rtd_aux = &card->rtd[card->num_links];
  	for (i = 0; i < card->num_links; i++)
  		card->rtd[i].dai_link = &card->dai_link[i];

  	INIT_LIST_HEAD(&card->list);
  	INIT_LIST_HEAD(&card->dapm_dirty);
  	card->instantiated = 0;
  	/* 调用snd_soc_instantiate_card（） */
  	ret = snd_soc_instantiate_card(card);
  	return ret;
  }
  ```
- snd_soc_instantiate_card解析
    ```c
    int snd_soc_instantiate_card(struct snd_soc_card *card)
    {
      	for (i = 0; i < card->num_links; i++) {
        	/* 遍历dai_list,codec_list,platform_list链表，找到与snd_soc_dai_link中匹配的cpu_dai，codec_dai，codec，platform；放入rtd[]中； */
    		ret = soc_bind_dai_link(card, i);
      	}
      	/* 初始化缓存信息 */
      	ret = snd_soc_init_codec_cache(codec, compress_type);
      	/* 调用snd_card_create()创建声卡，保存于card->snd_card中 */
      	snd_card_create();
      	for (order = SND_SOC_COMP_ORDER_FIRST; order <= SND_SOC_COMP_ORDER_LAST; order++) {
    		for (i = 0; i < card->num_links; i++) {
              /* 探测并prob所有组建，重要！codec的prob（soc_probe_codec） */
    			ret = soc_probe_link_components(card, i, order);
    			}
    		}

      for (order = SND_SOC_COMP_ORDER_FIRST; order <= SND_SOC_COMP_ORDER_LAST; order++) {
          for (i = 0; i < card->num_links; i++) {
        /* probe all components used by DAI links on this card */
              ret = soc_probe_link_components(card, i, order);
          }
      }
      snd_soc_dapm_link_dai_widgets(card);
      
      snd_soc_dapm_new_widgets(&card->dapm);
      snd_card_register(card->snd_card);
    }
    ```
### 4.DAPM和widget

> DAPM:动态音频电源管理，Dynamic Audio Porer Management的缩写，DAPM是为了使基于linux的移动设备上的音频子系统，任何时候在最小功耗状态下。用户空间的应用程序无需对代码做出修改，也无需编译，DAPM根据当前激活的音频流（playback/capture）和声卡中mixer等的配置来决定哪些音频控件被打开或关闭。参见内核文档`DAPM.txt`  

>  widget：DAPM的基本单元，指音频系统中的一个部件；
#### a.Kcontrol

> kcontrol代表着一个mixer（混音器），或一个mux（多路开关），或者一个音量控制器等，`snd_kcontrol_new`为其基本结构：
>
> ```c
> struct snd_kcontrol_new {
> 	snd_ctl_elem_iface_t iface;	/* interface identifier */
> 	unsigned int device;		/* device/client number */
> 	unsigned int subdevice;		/* subdevice (substream) number */
> 	const unsigned char *name;	/* ASCII name of item */
> 	unsigned int index;		/* index of item */
> 	unsigned int access;		/* access rights */
> 	unsigned int count;		/* count of same elements */
> 	snd_kcontrol_info_t *info;	
> 	snd_kcontrol_get_t *get;	//获取当前值
> 	snd_kcontrol_put_t *put;	//设置控件状态值
> 	union {
> 		snd_kcontrol_tlv_rw_t *c;
> 		const unsigned int *p;
> 	} tlv;
> 	unsigned long private_value;
> };
> ```
- 对于每个控件，定义一个与他对应的`snd_kcontrol_new`结构，这些结构会在声卡初始化的初始阶段，通过`snd_soc_add_codec_contorls()`注册到系统中；

- 一般系统会提供一系列宏来辅助定义控件：

  - SOC_SINGLE：最简单的控件，只有一个开关；
    ```c
    /* reg:寄存器，shift：控制位位移，max可设置的最大值，invert：是否逻辑取反 */
    #define SOC_SINGLE(xname, reg, shift, max, invert)
    ```

  - SOC_SINGLE_TLV：一般用来做音量控制器，EQ均衡器；
    ```c
    /* tlv_array:需要利用下面的宏定义的元数据 */
    #define SOC_SINGLE_TLV(xname, reg, shift, max, invert, tlv_array)
    /*tlv_array:name, -1000:最小值(0.01db) 75:步长，0:mute */
    static DECLARE_TLV_DB_SCALE(tlv_array, -1000, 75, 0);
    ```

  - SOC_DOUBLE：与SOC_SINGLE类似，但同时控制两个开关（同一个寄存器的两个位），例如立体声的控制；
    ```c
    #define SOC_DOUBLE(xname, reg, shift_left, shift_right, max, invert)
    ```

  - SOC_DOUBLE_R_TLV：类似SOC_SINGLE_TLV，同时控制两个（两个相似寄存器的同一个位）
    ```c
    /* reg_left:左声道寄存器， reg_right：右声道寄存器*/
    #define SOC_DOUBLE_R_TLV(xname, reg_left, reg_right, xshift, xmax, xinvert, tlv_array)
    ```

  - Mixer控件：多输入单输出，一般可以自由的混合在一起，形成混合后输出。对于Mixer控件，可看作是多个简单控件的组合，也就是soc_kcontrol_new数组：
    ```c
    static const struct snd_kcontrol_new lc1160_spk_mixer_controls[] = {
    	SOC_DAPM_SINGLE("Right Playback Switch", LC1160_R72, 0, 1, 1),
    	SOC_DAPM_SINGLE("Left Playback Switch", LC1160_R72, 1, 1, 1),
    	SOC_DAPM_SINGLE("Mono Voice Switch", LC1160_R72, 2, 1, 1),
    	SOC_DAPM_SINGLE("Right Line Switch", LC1160_R72, 3, 1, 1),
    	SOC_DAPM_SINGLE("Left Line Switch", LC1160_R72, 4, 1, 1),
    };
    ```
  - Mux控件：与mixer类似，但同时只能有一个被选中
    ```c
    static const char *ls[] = {"MIC1", "HPMIC"};
    static const struct soc_enum mic1Pga_mux_enum = SOC_ENUM_SINGLE(LC1160_R71, 6, 2, ls);
    static const struct snd_kcontrol_new mic1Pga_mux_kctl = SOC_DAPM_ENUM("mic1Pga mux kctl", mic1Pga_mux_enum);

    /* xname:控件名字 xenum: soc_enum的地址*/
    #define SOC_DAPM_ENUM(xname, xenum)
    /* xmax:输入项个数 xtexts: 输入项名字列表*/
    #define SOC_ENUM_SINGLE(xreg, xshift, xmax, xtexts)
    ```

  - kcontrol 的注册：在`soc_probe_codec()`中调用`snd_soc_add_codec_controls(codec, driver->controls， number）`
#### b.widget
- 可以理解为对kcontrol的进一步升级和封装，同样指音频系统中的某个部件，如mixer，mux;
- 类型，`include/sound/soc-dapm.h`中定义很多`widget`类型，归纳为：
  > o Mixer           - 混合多个模拟信号到一个输入口.
  >  o Mux            - 从多个输入中选择一个.
  >  o PGA            - A programmable gain amplifier or attenuation widget.
  >  o ADC            - Analog to Digital Converter
  >  o DAC            - Digital to Analog Converter
  >  o Switch         - An analog switch
  >  o Input            - A codec input pin
  >  o Output         - A codec output pin
  >  o Headphone  - Headphone (and optional Jack)
  >  o Mic              - Mic (and optional Jack)
  >  o Line             - Line Input/Output (and optional Jack)
  >  o Speaker      - Speaker
  >  o Supply         - Power or clock supply widget used by other widgets.
  >  o Pre              - Special PRE widget (exec before all others)
  >  o Post            - Special POST widget (exec after all others)

- widget的基本结构体`sdn_soc_dapm_widget`

  ```c
  struct snd_soc_dapm_widget {
  	enum snd_soc_dapm_type id; // 类型
  	const char *name;		/* widget name */
  	...
  	/* dapm control */
  	int reg;			/* negative reg = no direct dapm */
  	unsigned char shift;		/* bits to shift */
  	unsigned int value;			/* widget current value */
  	unsigned int mask;			/* non-shifted mask */
  	...

  	int (*power_check)(struct snd_soc_dapm_widget *w);

  	/* external events */
  	unsigned short event_flags;		/* flags to specify event types */
  	int (*event)(struct snd_soc_dapm_widget*, struct snd_kcontrol *, int);
  	/* kcontrols that relate to this widget */
  	int num_kcontrols;
  	const struct snd_kcontrol_new *kcontrol_news;
  	struct snd_kcontrol **kcontrols;

  	/* widget input and outputs */
  	struct list_head sources;
  	struct list_head sinks;
  	...
  };
  ```
- widget的连接关系：snd_soc_dapm_route
  ```c
  struct snd_soc_dapm_route {
  	const char *sink;
  	const char *control;
  	const char *source;

  /* Note: currently only supported for links where source is a supply */
  	int (*connected)(struct snd_soc_dapm_widget *source,
  			 struct snd_soc_dapm_widget *sink);
  };
  ```
  - sink：指向到达端的widget名字；
  - source：指向起始端widget的名字；
  - control：指向负责控制该连接所对应的kcontrol名字字符串。
  - 最终利用`snd_soc_dapm_add_routes()`

- widget定义

  在soc-dapm.h中定义了大量的宏来辅助定义各种widget，根据widget所在的电源域，被分为几个域：

  - codec domain

      ```c
      #define SND_SOC_DAPM_VMID(wname)
      ```
  - platform domain：无寄存器控制，
      ```c
      #define SND_SOC_DAPM_SIGGEN(wname)
      #define SND_SOC_DAPM_INPUT(wname)
      #define SND_SOC_DAPM_OUTPUT(wname) 
      #define SND_SOC_DAPM_MIC(wname, wevent)
      #define SND_SOC_DAPM_HP(wname, wevent)
      #define SND_SOC_DAPM_SPK(wname, wevent)
      #define SND_SOC_DAPM_LINE(wname, wevent)
      ```
  - path domain：通常是对普通kcontrol控件的再封装，增加音频路径和电源管理功能，这些widegt的reg和shift字段需要赋值；包含一个或多个`kcontrol`,事先需要提前定义好相关的`kcontrol`；带`_E` 的版本可以定义事件处理回调函数；

      > 关于带`_E`的回调版本，一般可用来在当前widget上电、下电时回调某些函数，做到自动控制，状态检测等操作；

      ```c
      #define SND_SOC_DAPM_PGA(wname, wreg, wshift, winvert, wcontrols, wncontrols)
      #define SND_SOC_DAPM_OUT_DRV(wname, wreg, wshift, winvert, wcontrols, wncontrols)
      #define SND_SOC_DAPM_MIXER(wname, wreg, wshift, winvert,  wcontrols, wncontrols)
      #define SND_SOC_DAPM_MIXER_NAMED_CTL(wname, wreg, wshift, winvert,  wcontrols, wncontrols)
      #define SND_SOC_DAPM_MICBIAS(wname, wreg, wshift, winvert)
      #define SND_SOC_DAPM_SWITCH(wname, wreg, wshift, winvert, wcontrols) 
      #define SND_SOC_DAPM_MUX(wname, wreg, wshift, winvert, wcontrols) 
      #define SND_SOC_DAPM_VIRT_MUX(wname, wreg, wshift, winvert, wcontrols) 
      #define SND_SOC_DAPM_VALUE_MUX(wname, wreg, wshift, winvert, wcontrols)
      ​```
      ```

  - ...

- 建立widget和route

  - 第一步，利用辅助宏创建widget所需要的dapm kcontrol；

  - 第二步，定义真正的widget，包含第一步定义的kcontrol；

  - 第三步，定义widget的连接路径；

  - 第四步，在codec驱动的probe回调中注册这些widget和路径；

#### b.DAPM

> dapm把整个音频系统，按照功能和偏置电源级别，划分为若干个电源域，每个域包含各自的widget，每个域中的所有widget通常处于同一个偏置电压级别上，而一个电压域就是一个`dapm_context`；`snd_soc_dapm_context`通常会被内嵌到codec、platform、card、dai的结构体中；
- snd_soc_dapm_context
  ```c
  /* DAPM context */
  struct snd_soc_dapm_context {
  	enum snd_soc_bias_level bias_level;
  	enum snd_soc_bias_level suspend_bias_level;
  	struct delayed_work delayed_work;
  	unsigned int idle_bias_off:1; /* Use BIAS_OFF instead of STANDBY */
  	struct snd_soc_dapm_update *update;

  	void (*seq_notifier)(struct snd_soc_dapm_context *,
  			     enum snd_soc_dapm_type, int);

  	struct device *dev; /* from parent - for debug */
  	struct snd_soc_codec *codec; /* parent codec */
  	struct snd_soc_platform *platform; /* parent platform */
  	struct snd_soc_card *card; /* parent card */

  	/* used during DAPM updates */
  	enum snd_soc_bias_level target_bias_level;
  	struct list_head list;

  	int (*stream_event)(struct snd_soc_dapm_context *dapm, int event);
  };
  ```

- 注册widget

  一般，会根据音频硬件的组成，非别在声卡的codec驱动、platform驱动、和machine驱动中定义一组widget，这些widget用数组进行组织；例如codec中的注册：会将组织好的widget放到`codec_driver`中，当machine驱动匹配上该codec时，会检查codec_driver的相应位是否赋值，有则调用：`snd_soc_dapm_new_controls()`进行`create new dapm controls`；对于不能组织到数组中的widget可以在codec的probe中单独利用`snd_soc_dapm_new_controls()`进行`create new dapm controls`；创建好的`snd_soc_dapm_widget`加入`snd_soc_card->widgets`链表；

## III TinyALSA
### 1.介绍 
- Android系统上的音频框架：APP-->Farmwork-->Audio Lib --> HAL
  - APP：如各类音乐APP;
  - Farmwork:提供一系列的服务和API的接口;关于音频的有MediaPlayer 、MediaRecorder,Android系统还为我们控制音频系统提供了AudioManager、AudioService及AudioSystem类;
  - Audio Lib:AudioFlinger和AudioPolicyService。它们的代码放置在frameworks/av/services/audioflinger，生成的最主要的库叫做libaudioflinger;
    －HAL:硬件抽象层是AudioFlinger直接访问的对象;