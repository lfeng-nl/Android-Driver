当前目录下编译执行，相当于在android目录下执行make 本文介绍Android的编译过程及各种mk文件的导入

## 1.编译

### a.编译步骤

- 清理

  ```sh
  $ make clobber
  ```

  ​

- 初始化环境

  ```sh
  $ source ./build/envetup.sh
  # 这个命令是用来将envsetup.sh里的所有用到的命令加载到环境变量里去
  ```

- 选择编译目标

  ```sh
  $ lunch aosp_arm-eng
  ```

- 编译

  ```sh
  $ make -j16
  ```

  > 参数N表示并行编译的任务数，一般N为cpu线程数的1-2倍.例如，在一台双核 E5520 计算机（2 个 CPU，每个 CPU 4 个内核，每个内核 2 个线程）上，要实现最快的编译速度，可以使用介于 make -j16 到 make -j32 之间的命令

### b.编译结果及目录

- **out/**  ：所有的编译产物都将位于out目录下，该目录下主要有以下几个子目录
- **-out/host/** ：该目录下包含了针对主机的 Android 开发工具的产物。即 SDK 中的各种工具，例如：emulator，adb，aapt 等
- **out/target/common/** ：该目录下包含了针对设备的共通的编译产物，主要是 Java 应用代码和 Java 库。
- **out/target/product/<product_name>/** ：包含了针对特定设备的编译结果以及平台相关的 C/C++ 库和二进制文件。其中，<product_name>是具体目标设备的名称。

### c.编译生成的主要镜像文件

**ramdisk.img**

  - 是一个最基础的小型文件系统，它包括了初始化系统所需要的全部核心文件；
  - 在启动时将被 Linux 内核挂载为只读分区，它包含了 /init 文件和一些配置文件。它用来挂载其他系统镜像并启动 init 进程

**boot.img**

  - 由boot header、kernel和ramdisk三部分打包在一起组成
  - 启动时，由bootloader解析后引导kernel启动

**system.img**

  - 被挂载到/system目录下
  - 包含了 Android OS 的系统文件，库，可执行文件以及预置的应用程序

 **userdata.img**

  - 被挂载到/data目录下
  - 包含了应用程序相关的数据以及和用户相关的数据
  - 可以预制一些可删除的应用软件
## 2.编译过程详解

###  a.source 过程

source命令：在当前bash环境下 **读取并执行** FileName中的命令；`$ source ./build/envsetup.sh` 读取envsetup.sh 中的函数，并执行脚本 。

- 加载的函数有

  | 函数              | 说明                                       |
  | --------------- | ---------------------------------------- |
  | m               | 当前目录下编译执行，相当于在android目录下执行make           |
  | mm              | 执行当前目录下最近的make文件                         |
  | mmm             | 在android目录下，执行某个文件夹下的make文件              |
  | mma             | 编译当前目录下所有的模块和他们的依赖                       |
  | make update-api | 更新 API。在 framework API 改动之后，需要首先执行该命令来更新 API |
  | printconfig     | 打印配置                                     |
  | lunch           | 配置lunch                                  |
  | croot           | 回到根目录                                    |
  | cgrep           | 在所有 C/C++ 文件上执行 grep                     |
  | jgrep           | 在所有 Java 文件上执行 grep                      |
  | resgrep         | 在所有 res/*.xml 文件上执行 grep                 |

- 执行脚本：1.执行add_lunch_combo xxxxx  2.检查shell是否为bash，3.查找 vendorsetup.sh，并 include ；4.输出所有的include文件 5，执行addcompletions() 检查bash版本;
### b. lunch过程
`lunch <product_name>-<build_variant>`，选择平台编译选项

### c. Make 过程

makefile的相关语法请参考Makefile语法一文或其他书籍。

入口文件是源码树根目录下名称为`Makefile`的文件，当在源代码根目录上调用 `make` 命令时，`make` 命令首先将读取该文件。`Makefile` 文件的内容只有一行：`include build/core/main.mk`。作用仅为包含 `build/core/main.mk `文件。在 `main.mk` 文件中又会去包含其他的文件，其他文件中又会包含更多的文件，这样就引入了整个 Build 系统。

![Makefile文件](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image004.png)

####  比较重要的make文件介绍：

devices/xxxxx/BoardConfig.mk，AndroidBoard.mk

BoardConfig.mk旨在包含硬件特定的配置和生成标志。
它应该定义可用于设备的低级硬件功能
它应该定义如何打包最终的输出以及包含的内容（启动加载器，内核等）
AndroidBoard.mk在HW板上配置Android平台
定义在构建过程中生成并包含在Android平台构建中的文件
定义各种Android分区的文件
定义要为您的产品打包的一组应用程序
可以定义需要成为产品构建部分的Android平台功能。