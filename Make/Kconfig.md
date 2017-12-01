# 本文介绍Kconfig

## 1.简介

所有配置工具都是通过读取arch/$(ARCH)Kconfig文件来生成配置界面，这个文件就是所有配置的总入口，它会包含其他目录的Kconfig，Kconfig的作用是用来配置内核，它就是各种配置界面的源文件，内核的配置工具读取各个Kconfig文件，生成配置界面供开发人员配置内核，最后生成配置文件.config

默认的defconfig是基本框架，最终的.config 会根据defconfig和Kconfig生成；

## 2.语法

Kconfig按照一定的格式来书写，menuconfig程序可以识别这种格式，然后从中提取出有效信息组成menuconfig中的菜单项，每一行都以一个关键字开头，后面跟很多参数；

```makefile
config DTS_EAGLE
	bool "Enable DTS Eagle Support"
	depends on SND_SOC_MSM_QDSP6V2_INTF
	select SND_HWDEP
	help
	 To add DTS Eagle support on QDSP6 targets.
	 Eagle is a DTS pre/post processing
	 package that includes HeadphoneX. The configuration
	 includes sending tuning parameters of various modules.
```
- **config**：新的菜单标记项
- **bool**：y or n
- **tristate**：三态，y，n，m
- **source**：引入这个文件夹下的Kconfig
- **depends on**：意思是本配置项依赖于另一个配置项。如果那个依赖的配置项为Y或者M，则本配置项才有意义；如果依赖的哪个配置项本身被设置为N，则本配置项根本没有意义;
- **select**：表示depends on的值有效时，下面的select也会成立，将相应的内容选上。
- **default**：表示depends on的值有效时，下面的default也会成立，将相应的选项选上，有三种选项，分别对应y，n，m。
- **help**：帮助信息，解释这个配置项的含义，以及如何去配置他。
