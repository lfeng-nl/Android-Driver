#  本文档介绍 Android.mk 文件

简介：Android.mk是Android提供的一种makefile文件，用来指定诸如编译生成so库名、引用的头文件目录、需要编译的.c/.cpp文件和.a静态库

##1.模块类型

### apk程序
- Android程序，编译打包生成apk文件
- 生成路径 out/target/product/{product-name}/obj/APPS/XXX_intermediates

### java库
- java库，编译打包生成jar文件
- 生成路径 out/target/product/{product-name}/obj/JAVA_LIBRARIES/XXX_intermediates

### c\c++应用程序
- 可执行的c\c++应用程序
- 生成路径 out/target/product/{product-name}/obj/EXECUTABLE/XXX_intermediates

### c\c++静态库
- 编译生成c\c++静态库，并打包成.a文件
- 生成路径 out/target/product/{product-name}/obj/STATIC_LIBRARY/XXX_static_intermediates

### c\c++共享库
- 编译生成共享库（动态链接库），并打包成.so文件，会被整合到apk包中。
- 生成路径 out/target/product/{product-name}/obj/obj/SHARED_LIBRARY/XXX_shared_intermediates
## 2. 编写Android.mk

### a.概述

- Setting.apk的Android.mk内容

  ```make
  LOCAL_PATH:= $(call my-dir)
  include $(CLEAR_VARS)
  # 一个Android.mk以这两行开始，其作用是
  # 设置当前模块的编译路径为当前文件夹路径。
  # 清理（可能由其他模块设置过的）编译环境中用到的变量。

  LOCAL_JAVA_LIBRARIES := bouncycastle conscrypt telephony-common ims-common
  LOCAL_STATIC_JAVA_LIBRARIES := android-support-v4 android-support-v13 jsr305

  LOCAL_MODULE_TAGS := optional

  LOCAL_SRC_FILES := $(call all-java-files-under, src)

  LOCAL_PACKAGE_NAME := Settings
  LOCAL_CERTIFICATE := platform

  include $(BUILD_PACKAGE)

  # Use the folloing include to make our test apk.
  include $(call all-makefiles-under,$(LOCAL_PATH))
  ```
- 编译环境变量

  | 变量                          | 作用                                       |
  | :-------------------------- | :--------------------------------------- |
  | LOCAL_MODULE                | 当前模块的名称，这个名称应当是唯一的，模块间的依赖关系就是通过这个名称来引用的  |
  | LOCAL_MODULE_CLASS          | 模块所属类型，根据该类型将生成的模块放到指定的目录下，<br> 如：EXECUTABLES 放到system/bin目录下 |
  | LOCAL_PACKAGE_NAME          | 当前apk应用的名称                               |
  | LOCAL_SRC_FILES             | 当前模块包含的所有源代码文件                           |
  | LOCAL_JAVA_LIBRARIES        | 当前模块依赖的 Java 共享库                         |
  | LOCAL_STATIC_JAVA_LIBRARIES | 当前模块依赖的 Java 静态库                         |
  | LOCAL_C_INCLUDES            | C 或 C++ 语言需要的头文件的路径                      |
  | LOCAL_STATIC_LIBRARIES      | 当前模块在静态链接时需要的库的名称                        |
  | LOCAL_SHARED_LIBRARIE       | 当前模块在运行时依赖的动态库的名称                        |
  | LOCAL_CFLAGS                | 提供给 C/C++ 编译器的额外编译参数                     |
  | LOCAL_CERTIFICATE           | 签署当前应用的证书名称。                             |
  | LOCAL_MODULE_TAG            | 当前模块所包含的标签，一个模块可以包含多个标签。标签的值可能是 debug, eng,user, development或optional。其中，optional 是默认标签。 |
> 标签是提供给编译类型使用的。不同的编译类型会安装包含不同标签的模块

- 编译类型的说明

  | 名称        | 说明                                       |
  | --------- | ---------------------------------------- |
  | eng       | 默认类型，该编译类型适用于开发阶段。<br>当选择这种类型时，编译结果将：<br>安装包含 eng, debug, user，development 标签的模块<br>安装所有没有标签的非 APK 模块<br>安装所有产品定义文件中指定的 APK 模块 |
  | user      | 该编译类型适合用于最终发布阶段。<br>当选择这种类型时，编译结果将：<br>安装所有带有 user 标签的模块 <br> 安装所有没有标签的非 APK 模块 <br> 安装所有产品定义文件中指定的 APK 模块，APK 模块的标签将被忽略 |
  | userdebug | 该编译类型适合用于 debug 阶段。<br>该类型和 user 一样，除了：<br>会安装包含 debug 标签的模块 <br>编译出的系统具有 root 访问权限 |
  - 编译系统中提供的函数

  ```makefile
  # 获取当前文件夹路径
  $(call my-dir)
  # 获取指定目录下的所有 Java 文件
  $(call all-java-files-under, <src>)
  # 获取指定目录下的所有 C 语言文件
  $(call all-c-files-under, <src>)
  # 获取指定目录下的所有 AIDL 文件
  $(call all-Iaidl-files-under, <src>)
  # 获取指定目录下的所有 Make 文件
  $(call all-makefiles-under, <folder>)
  # 获取 Build 输出的目标文件夹路径
  $(call intermediates-dir-for, <class>, <app_name>, <host or target>, <common?> )
  ```
- 编译方式常量

  ```makefile
  在/build/core/config.mk中定义一些常量，每个常量对应一种编译方式的Makefile文件
  BUILD_HOST_STATIC_LIBRARY:= $(BUILD_SYSTEM)/host_static_library.mk
  BUILD_HOST_SHARED_LIBRARY:= $(BUILD_SYSTEM)/host_shared_library.mk
  BUILD_STATIC_LIBRARY:= $(BUILD_SYSTEM)/static_library.mk
  BUILD_SHARED_LIBRARY:= $(BUILD_SYSTEM)/shared_library.mk
  BUILD_EXECUTABLE:= $(BUILD_SYSTEM)/executable.mk
  BUILD_HOST_EXECUTABLE:= $(BUILD_SYSTEM)/host_executable.mk
  BUILD_PACKAGE:= $(BUILD_SYSTEM)/package.mk
  BUILD_PHONY_PACKAGE:= $(BUILD_SYSTEM)/phony_package.mk
  BUILD_HOST_PREBUILT:= $(BUILD_SYSTEM)/host_prebuilt.mk
  BUILD_PREBUILT:= $(BUILD_SYSTEM)/prebuilt.mk
  BUILD_MULTI_PREBUILT:= $(BUILD_SYSTEM)/multi_prebuilt.mk
  BUILD_JAVA_LIBRARY:= $(BUILD_SYSTEM)/java_library.mk
  BUILD_STATIC_JAVA_LIBRARY:= $(BUILD_SYSTEM)/static_java_library.mk
  BUILD_HOST_JAVA_LIBRARY:= $(BUILD_SYSTEM)/host_java_library.mk
  ```
- 各种模块的编译方式的定义文件

  | 文件名                    | 说明                                       |
  | ---------------------- | ---------------------------------------- |
  | host_static_library.mk | 定义了如何编译主机上的静态库。                          |
  | host_shared_library.mk | 定义了如何编译主机上的共享库。                          |
  | static_library.mk      | 定义了如何编译设备上的静态库。                          |
  | shared_library.mk      | 定义了如何编译设备上的共享库。                          |
  | executable.mk          | 定义了如何编译设备上的可执行文件。                        |
  | host_executable.mk     | 定义了如何编译主机上的可执行文件。                        |
  | package.mk             | 定义了如何编译 APK 文件。                          |
  | prebuilt.mk            | 定义了如何处理一个已经编译好的文件 ( 例如 Jar 包 )。          |
  | multi_prebuilt.mk      | 定义了如何处理一个或多个已编译文件，该文件的实现依赖 prebuilt.mk。  |
  | host_prebuilt.mk       | 处理一个或多个主机上使用的已编译文件，该文件的实现依赖 multi_prebuilt.mk。 |
  | java_library.mk        | 定义了如何编译设备上的共享 Java 库。                    |
  | static_java_library.mk | 定义了如何编译设备上的静态 Java 库。                    |
  | host_java_library.mk   | 定义了如何编译主机上的共享 Java 库。                    |
### b.示例

编译一个 APK 文件
```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# 获取所有子目录中的 Java 文件
LOCAL_SRC_FILES := $(call all-subdir-java-files)
# 当前模块依赖的静态 Java 库，如果有多个以空格分隔
LOCAL_STATIC_JAVA_LIBRARIES := static-library
# 当前模块的名称
LOCAL_PACKAGE_NAME := LocalPackage
# 编译 APK 文件
include $(BUILD_PACKAGE)
编译一个 Java 的静态库

OCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# 获取所有子目录中的 Java 文件
LOCAL_SRC_FILES := $(call all-subdir-java-files)
# 当前模块依赖的动态 Java 库名称
LOCAL_JAVA_LIBRARIES := android.test.runner
# 当前模块的名称
LOCAL_MODULE := sample
# 将当前模块编译成一个静态的 Java 库
include $(BUILD_STATIC_JAVA_LIBRARY)
```
FMRadio，编译jni动态库，并打包到apk中
packages/apps/FMRadio/jni/fmr/Android.mk

编译生成libfmjni.so
```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
# 定义源代码文件
LOCAL_SRC_FILES := \
    fmr_core.cpp \
    fmr_err.cpp \
    libfm_jni.cpp \
    common.cpp
# 引入头文件目录
LOCAL_C_INCLUDES := $(JNI_H_INCLUDE) \
    frameworks/base/include/media
# 依赖的动态库
LOCAL_SHARED_LIBRARIES := \
    libcutils \
    libdl \
    libmedia \
# 当前模块名称
LOCAL_MODULE := libfmjni
# 编译一个动态库
include $(BUILD_SHARED_LIBRARY)
packages/apps/FMRadio/Android.mk
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE_TAGS := optional
LOCAL_CERTIFICATE := platform

LOCAL_SRC_FILES := $(call all-java-files-under, src)
# apk应用名称
LOCAL_PACKAGE_NAME := FMRadio
# 定义要包含的jni库文件的名字，apk被安装后，jni库文件放在 [apk应用]/lib/目录下
LOCAL_JNI_SHARED_LIBRARIES := libfmjni

LOCAL_PROGUARD_ENABLED := disabled
LOCAL_PRIVILEGED_MODULE := true
# 依赖的静态库
LOCAL_STATIC_JAVA_LIBRARIES += android-support-v7-cardview
# 定义资源的路径
LOCAL_RESOURCE_DIR = $(LOCAL_PATH)/res frameworks/support/v7/cardview/res
# 定义aapt参数，aapt是打包apk的工具，执行aapt --help可以查看参数的含义
LOCAL_AAPT_FLAGS := --auto-add-overlay --extra-packages android.support.v7.cardview
# 编译一个apk
include $(BUILD_PACKAGE)
# 引入当前路径下的所有makefile文件
include $(call all-makefiles-under,$(LOCAL_PATH))
```
集成第三方软件(.jar，.so)

本例中apk使用百度地图提供的jar和so库文件
假设apk代码路径：packages/apps/MyMaps/
jar文件路径：packages/apps/MyMaps/libs/baidumapapi.jar
so文件路径：packages/apps/MyMaps/libs/armeabi/libBMapApiEngine_v1_3_1.so
Android.mk内容如下：

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
# 依赖静态库libbaidumapapi，这是baidumapapi.jar的别名
LOCAL_STATIC_JAVA_LIBRARIES := libbaidumapapi

LOCAL_SRC_FILES := $(call all-subdir-java-files)
# apk软件名称MyMaps
LOCAL_PACKAGE_NAME := MyMaps
# 编译一个apk
include $(BUILD_PACKAGE)

##################################################
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
# 定义一个预编译的java静态库，格式： <别名>:<文件路径>
# 注：以可以不设置别名
# 编译时，该jar会被放目录 out/target/product/{product-name}/obj/JAVA_LIBRARIES/libbaidumapapi_intermediates中
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES :=libbaidumapapi:libs/baidumapapi.jar
# 定义一个预编译的二进制库文件
# 该so将被放到/system/lib/目录下
LOCAL_PREBUILT_LIBS :=libBMapApiEngine_v1_3_1:libs/armeabi/libBMapApiEngine_v1_3_1.so
# 执行预编译
include $(BUILD_MULTI_PREBUILT)
##################################################

include $(callall-makefiles-under,$(LOCAL_PATH))
```
参考资料

http://android.mk/
https://developer.android.com/ndk/guides/android_mk.html
https://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/index.html
http://blog.csdn.net/huangxiaominglipeng/article/details/17839239
http://blog.csdn.net/thl789/article/details/7918093
http://blog.csdn.net/a462533587/article/details/46380795