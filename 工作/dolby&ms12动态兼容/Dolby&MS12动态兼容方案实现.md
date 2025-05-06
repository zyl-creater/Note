# 文档历史发放及记录

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|序号|变更（+/-）说明|作者|版本号|日期|批准|
|1|初始版本，基于Android S|朱岳霖|V1.0|2024.01.29||
|||||||
|||||||
|||||||
|||||||
|||||||
|||||||
|||||||

# 一、概述

## 1.1 基本信息

### Dolby Digital Plus介绍

Dolby Digital Plus (DDP) 是杜比实验室开发的一种数字音频编码技术，旨在提供高质量的声音体验。它是Dolby Digital (AC-3)的升级版本，可以支持更高的位速和采样率，以及更多的声道数量。Dolby Digital Plus通常用于蓝光光盘、流媒体服务、电视广播和家庭影院系统中。

与Dolby Digital相比，Dolby Digital Plus具有更高的比特率，因此可以提供更高的音频质量。它支持最多8个全范围频带的声道，每个声道的最高采样率可达48kHz。Dolby Digital Plus还使用了一些新的编码技术，例如Spectral Band Replication和Dialogue Normalization，这些技术可以提高音频的清晰度和对话的可听度。

### MS12介绍

MS12是微软公司开发的一种音频编解码技术，主要用于数字电视和互联网视频等领域。它是Dolby Digital Plus技术的基础上进行了改进和优化，旨在提供更高质量、更低延迟、更低比特率的音频编码方案。

与传统的Dolby Digital技术相比，MS12采用了更为先进的编码算法和信号处理技术。通过抑制噪声和失真，优化动态范围和声音清晰度，MS12能够将高品质的音频数据压缩到更小的文件大小，以便更快地下载和流式传输。

### Dolby Vision介绍

Dolby Vision是杜比实验室开发的一种高动态范围（HDR）技术，旨在提供更为逼真、更为透彻、更为生动的影像。与传统的电视格式不同，Dolby Vision支持更高的亮度水平，更深的黑色和更广泛的颜色范围，以便更好地显示现实世界中的强烈对比和鲜艳颜色。

Dolby Vision采用了一项名为“动态元数据”的技术，使影片制作者可以根据特定场景或具体场景的要求来调整画面的亮度、对比度和颜色。这些元数据被嵌入到Dolby Vision视频流中，并由兼容的硬件播放器自动解码并应用于屏幕上的每一帧图像。该过程确保每一帧都可以获得最佳的图像效果，无论是在明亮的户外环境还是在暗淡的影院中观看。

## 1.2 场景/背景

旧的版本配置方案是在打包时通过设置LICENSE_SUPPORT宏的方式，使用不同的库和配置文件，每次配置的一版软件只支持一种配置，基线验证大版本更新时，至少需要配置五个不同版本的软件进行验证。分别有以下7种值：

1：none：ddp、dv、ms12都不支持

2：ddp：支持ddp，不支持dv、ms12

3：ddp_dv：支持ddp、dv，不支持ms12

4：dpp_ms12-v1：支持ddp和ms12 v1版本，不支持dv

5：ddp_ms12-v2：支持ddp和ms12 v2版本，不支持dv

6：ddp_ms12-v1_dv：支持ddp、ms12 v1版本、dv

7：ddp_ms12-v2_dv：支持ddp、ms12 v2版本、dv

设置了这个属性后，会将打包工程的tools/license目录下的内容拷贝到oem中。

每个取值对应的内容都不同，对于x4和y4也会有不同的配置文件。

## 1.3 Lisence下的内容

audio_policy_configuration.xml

libHwAudio_dcvdec.so

media_codecs.xml

media_codecs_performance.xml

dovi.ko

dovi_fw.bin

libdolbyms12.so

## 1.4 参考资料

[Amlogic DolbyVision Release to SDMC.pdf](https://alidocs.dingtalk.com/i/nodes/gvNG4YZ7Jnev2e7XSB0oP5e1V2LD0oRE?utm_scene=person_space)

svn://10.10.61.22/package_r/project/technology_docs/03.经验总结/Dolby Vision & MS12 集成方案/Dolby Vision & MS12 集成方案.pdf

svn://10.10.61.22/package_r/project/technology_docs/03.经验总结/SELinux 配置总结

svn://10.10.61.22/package_r/project/technology_docs/03.经验总结/init分析

# 二、判断芯片的方式

## 2.1 Dolby支持判断

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NybEnBr8YYwRnP13/img/3a3be296-2807-45fd-b4f8-042f2131899a.png)

根据表里面的信息，可以根据两个特定的节点值去判断是什么后缀的芯片，支持Dolby到哪个级别

根据表格中的验证信息，可以通过/sys/class/amaudio/dolby_enable 和/sys/class/amvecm/reg 这两个节点来判断cpu支持Dolby的级别

## 2.2 MS12支持判断

应为MS12属于纯软件支持，ms12的判断方式只能通过检测盒子是否烧录了ms12的efuse来区分项目是否支持ms12

可以使用下面命令来进行判断

```
echo get_lock AUDIO_VENDOR_ID > /sys/class/efuse/efuse_obj
cat /sys/class/efuse/efuse_obj
```

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YvenvKd6PDg0qoyZ/img/8de877d9-67b0-4822-8832-378d3b42bdd8.png)

如果返回值为01，表示烧录了ms12 efuse，则判断是支持ms12；如果返回值为空，表示不支持ms12

# 三、配置文件的加载位置

## audio_policy_configuration.xml

system/media/audio/include/system/audio_config.h

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/ee64f700-5e0c-4d3c-b6d5-c605889db798.png)

## libdolbyms12.so 和 libHwAudio_dcvdec.so

hardware/amlogic/audio/audio_hal/dolby_lib_api.h

hardware/amlogic/legacy/legacy_v10/audio/audio_hal/dolby_lib_api.c

hardware/amlogic/audio/libms12/src/dolby_ms12.cpp

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/21cf1117-316a-41f7-b9d5-f98ac94c3ca5.png)

## media_codecs.xml 和 media_codecs_performance.xml

frameworks/av/media/libstagefright/xmlparser/include/media/stagefright/xmlparser/MediaCodecsXmlParser.h

frameworks/av/media/libstagefright/xmlparser/MediaCodecsXmlParser.cpp

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/d25bb12e-4f34-46e1-8b54-7331b29ac3a9.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/e1939814-d50c-4224-9544-b51b98aacbec.png)

## dovi.ko

验证一下加载也并不影响功能，但是要考虑不同芯片要加载的ko不一样怎么处理

device/amlogic/ohm/init.amlogic.board.rc

vendor/amlogic/reference/tv/tvserver/libtv/tvsetting/TvKeyData.h

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/46176403-3a82-46c7-824b-6f8a38b4fb8c.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/854acd3b-0699-4ab8-bf96-a4a5309ec172.png)

## dovi_fw.bin

bootloader/uboot-repo/bl33/v2019/board/amlogic/configs/

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/1b86e2af-cc16-4269-9f8f-81e390ac2274.png)

dovi_fw.bin文件主要改动在bootloader下面

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/c9a0d616-4281-499c-8957-0234dd2294bd.png)

下面是加载dovi_fw.bin文件的代码

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/6f0d60ad-6954-45b3-bb1d-72de8da04eb4.png)

# 四、优化实现的方式

设计思路：

1. 在软件编译的阶段，将所有不同模式所有需要的配置文件全部集成在软件中，放到vendor/etc/license目录下
    
2. 在软件开机启动阶段
    
    1. 通过执行程序去识别芯片所支持的级别
        
        1. 如果ro.odm.sdmc.license_suffix已经默认配置，就会使用ro.odm.sdmc.license_suffix中的配置
            
        2. 如果ro.odm.sdmc.license_suffix没有配置，执行程序会根据CPU支持级别来进行配置
            
    2. 在rc阶段，rc脚本根据配置的属性去拷贝对应的支持文件到指定的目录下
        

仿照LICENSE_SUPPORT，通过配置 ro.odm.sdmc.license_suffix 属性支持以下7种模式：

1. none：ddp、dv、ms12都不支持
    
2. ddp：支持ddp，不支持dv、ms12
    
3. ddp_dv：支持ddp、dv，不支持ms12
    
4. dpp_ms12-v1：支持ddp和ms12 v1版本，不支持dv
    
5. ddp_ms12-v2：支持ddp和ms12 v2版本，不支持dv
    
6. ddp_ms12-v1_dv：支持ddp、ms12 v1版本、dv
    
7. ddp_ms12-v2_dv：支持ddp、ms12 v2版本、dv
    

默认不设置属性的话，就会通过芯片后缀来配置默认属性

# 五、代码分析

## 5.1 拷贝所有文件

事先在编译中，将所有需要用到的文件拷贝到 /vendor/etc/lisence 目录下分为八个目录：

none：xml文件

ddp：xml文件

ddp_dv：xml文件

dpp_ms12-v1：xml文件

ddp_ms12-v2：xml文件

ddp_ms12-v1_dv：xml文件

ddp_ms12-v2_dv：xml文件

ext：所有公用的相同的文件，so库，ko，bin文件等

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/2e53ae4d-3500-4070-9887-b0257c0e319f.png)

这里有一个问题，y4-J平台使用的ms12是s4d的so，这里对y4平台拷贝时多拷贝了一份

## 5.2 配置驱动节点

新增一些可以直接返回值的节点，可以减少可执行程序访问操作是的se权限问题

### 5.2.1 Dolby节点修改

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/61462194-7808-4a0f-99b0-a8bd1fedd12e.png)

驱动中是加入一个输出的节点，直接访问这个节点，就可以了获取芯片的Dolby的节点值

解决了原生的/sys/class/amvecm/reg节点需echo指定数据，而且只有打印没有返回值

### 5.2.2 MS12节点修改

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/YvenvKd6PDg0qoyZ/img/e4605fe1-94bb-490a-ac69-c449ead1d3c8.png)

增加可以直接访问问的ms12节点，可以直接查看是否烧录ms12 efuse

## 5.3 可执行程序

编译好可执行程序

在ro.odm.sdmc.license_suffix属性未定义的时候，去判断芯片的后缀；

在ro.odm.sdmc.license_suffix有属性的时候，根据属性去配置好相应的需要用的属性值

```
#define LOG_NDEBUG 0
#define LOG_TAG "dynamic_license"

#include <stdio.h>
#include <unistd.h>
#include "cutils/log.h"
#include <string.h>
#include <errno.h>
#include "cutils/properties.h"

#define ENABLE_TRIGGLE  "ro.odm.sdmc.enable_dynamic_license"
#define SUPPORT_LICENSE_TYPE  "ro.odm.sdmc.license_suffix"
#define SOC_SUFFIX  "ro.board.platform"
#define DCVDEC_SUFFIX  "ro.odm.sdmc.dcvdec_suffix"
#define MS12_SUFFIX  "ro.odm.sdmc.ms12_suffix"
#define DOVI_SUFFIX  "ro.odm.sdmc.dovi_suffix"
#define DOLBY_AUDIO_ENABLE  "/sys/class/amaudio/dolby_enable"
#define DOLBY_VISION_ENABLE  "/sys/class/amvecm/dolby_vpu"
#define MS12_VERSION_ENABLE  "/sys/class/efuse/ms12_node"

#define MAX_STR_LEN         100

#define NONE  0
#define DDP  2
#define DDP_MS12  3
#define DDP_DV  4
#define DDP_DV_MS12  5

static int sdmc_get_sysfs_str(const char *path, char *value) {
    int fd;
    int result = 0;
    char bcmd[MAX_STR_LEN] = { 0 };
    fd = open(path, O_RDONLY);
    if (fd >= 0) {
        read(fd, bcmd, sizeof(bcmd));
        close(fd);
        strcpy(value, bcmd);
    } else {
        ALOGE("unable to open file %s,err: %s", path, strerror(errno));
        result = -1;
    }
    return result;
}

static inline int get_the_type_of_chip(char* buf) {
    //根据文件信息和节点信息判断是什么芯片
    //读取文件信息

    int result = 0;

    int ms12 = sdmc_get_sysfs_str(MS12_VERSION_ENABLE, buf);
    if (ms12 < 0){
        ALOGE("fialed to read ms12_node file");
        return ms12;
    }
    if (strncmp("01", buf, 2) == 0) {
        result = result + 1;
    }

    int ddp_dv = sdmc_get_sysfs_str(DOLBY_VISION_ENABLE, buf);
    if (ddp_dv < 0){
        ALOGE("fialed to read dolby vision file");
        return ddp_dv;
    }
    if (strncmp("0x00000018", buf, 10) == 0) {
        //是-j芯片
        result = result + 4;
    } else {
        int ddp = sdmc_get_sysfs_str(DOLBY_AUDIO_ENABLE, buf);
        if (ddp < 0){
            ALOGE("fialed to read dolby audio fs");
            return ddp;
        }
        if (strncmp("0x1", buf, 3) == 0) {
            result = result + 2;
        }
    }

    switch (result)
    {
    case 0:
        return NONE;
    case 2:
        return DDP;
    case 3:
        return DDP_MS12;
    case 4:
        return DDP_DV;
    case 5:
        return DDP_DV_MS12;
    default:
        break;
    }
    return -1;
}

int main() {

    char buf[MAX_STR_LEN] = { 0 };
    char SOC_buf[MAX_STR_LEN] = { 0 };
    //读取是否启用功能的属性
    property_get(ENABLE_TRIGGLE, buf, "");
    if (strncmp("1", buf, 1) == 0) {
        ALOGV("function disabled");
        return 0;
    }

    //获取soc信息
	property_get(SOC_SUFFIX, SOC_buf, "");

    //判断最终属性是否存在，如果存在就自动结束
    property_get(SUPPORT_LICENSE_TYPE, buf, "");
    if (strlen(buf) > 0) {
        if(strstr(buf, "none") != NULL) {
			property_set(DCVDEC_SUFFIX, "_none");
			property_set(DOVI_SUFFIX, "_none");
			property_set(MS12_SUFFIX, "_none");
		} else {
			property_set(DCVDEC_SUFFIX, "_ok");
		}
		if(strstr(buf, "dv") != NULL) {
			property_set(DOVI_SUFFIX, "_ok");
		} else {
			property_set(DOVI_SUFFIX, "_none");
		}
		if(strstr(buf, "v1") != NULL) {
			property_set(MS12_SUFFIX, "_v1");
		} else if(strstr(buf, "v2") != NULL) {
			//如果是y4-J的芯片
			if(strstr(SOC_buf, "s4") != NULL && strstr(buf, "dv") != NULL) {
				property_set(MS12_SUFFIX, "_v2_s4d");
			} else {
				property_set(MS12_SUFFIX, "_v2");
			}
		} else {
			property_set(MS12_SUFFIX, "_none");
		}
		ALOGV("ro.odm.sdmc.license_suffix exists: %s", buf);
        return 0;
    }

    int result = get_the_type_of_chip(buf);
    ALOGV("get_the_type_of_chip： %d",result);
    if (result < 0) {
        ALOGE("failed to get the type of chip");
        result = NONE;
    }
	
	// 1: none ; 2: ddp ; 3: ddp_dv
    if (result == DDP_DV_MS12) {
        property_set(SUPPORT_LICENSE_TYPE, "ddp_ms12-v2_dv");
        property_set(DCVDEC_SUFFIX, "_ok");
		property_set(DOVI_SUFFIX, "_ok");
        //如果是y4-J的芯片
		if(strstr(SOC_buf, "s4") != NULL ) {
			property_set(MS12_SUFFIX, "_v2_s4d");
		} else {
			property_set(MS12_SUFFIX, "_v2");
		}
    }
    if (result == DDP_DV) {
        property_set(SUPPORT_LICENSE_TYPE, "ddp_dv");
        property_set(DCVDEC_SUFFIX, "_ok");
		property_set(DOVI_SUFFIX, "_ok");
		property_set(MS12_SUFFIX, "_none");
    }
    if (result == DDP_MS12) {
        property_set(SUPPORT_LICENSE_TYPE, "ddp_ms12-v2");
		property_set(DCVDEC_SUFFIX, "_ok");
		property_set(DOVI_SUFFIX, "_none");
		property_set(MS12_SUFFIX, "_v2");
    }
	if (result == DDP) {
        property_set(SUPPORT_LICENSE_TYPE, "ddp");
		property_set(DCVDEC_SUFFIX, "_ok");
		property_set(DOVI_SUFFIX, "_none");
		property_set(MS12_SUFFIX, "_none");
    }
	if (result == NONE) {
        property_set(SUPPORT_LICENSE_TYPE, "none");
		property_set(DCVDEC_SUFFIX, "_none");
		property_set(DOVI_SUFFIX, "_none");
		property_set(MS12_SUFFIX, "_none");
    }
    return 0;
 
}
```

## 5.4 配置rc等启动文件

配置好RC文件，让系统起来后，让RC根据配置的属性去拷贝相应的文件到指定目录

### 5.4.1 拷贝相应的xml，so，ko，bin等文件到指定的目录

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/7ea52ad9-7671-4851-8313-6bff2dbc8b07.png)

这里是做一个软连接，我们的odm/etc 目录在 rc中无法直接进行操作，在编译的时候通过软连接的方式将我们的文件连接到无法操作的目录

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/e5427e6b-b6de-4acd-bea1-438d25f17744.png)

这个是我们的RC文件，在开机后加载，rc通过我们设定的不同属性来拷贝相应的文件到对应的目录下面

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/2ded0619-f590-4e41-86da-a66ece683bf4.png)

### 5.4.2 修改dovi.ko的加载路径

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/09fe511a-5496-4573-ae25-7ff7e0cf8681.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NybEnBr8YYwRnP13/img/24123205-61b8-4c58-a48e-5d103a956a61.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NybEnBr8YYwRnP13/img/dfda965b-0592-4da1-bf43-386c1db25abd.png)

这两个改动他的所编到的so库和可执行程序都是存在vendor下面，对gsi认证不会有影响

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/NybEnBr8YYwRnP13/img/57dde077-a2d9-48d6-bb30-7542a8b91178.png)

## 5.5 配置SE

配好SE权限

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/f7d7f052-2122-435e-8cf3-c2cec9d4bab1.png)

用于打开指定文件并设置其 SELinux 扩展属性。

- filp_open 函数用于打开指定的文件。在这个示例中，我们将 AML_DOVI_KO_BIN 定义为要打开的文件名，O_RDONLY | O_PATH 表示只读模式和路径模式，0 表示默认权限
    
- 如果打开文件成功且没有错误，则从文件中获取目录项，并将其存储在 entry 变量中。file_dentry 函数用于获取给定文件对象的目录项。
    
- vfs_setxattr 函数用于设置文件系统扩展属性。在这里，我们设置 SELinux 扩展属性，使用 XATTR_NAME_SELINUX 宏定义属性名称，AML_DOVI_KO_CON 定义属性内容，sizeof(AML_DOVI_KO_CON) 表示属性内容的大小，0 表示默认标志
    

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/vBPlNzpZPAX3ldG8/img/bae168ec-1977-4fd5-888d-6c7b7ba8dbfb.png)

# 六、相关难处理问题

## 6.1 oem分区的加载问题

在kernel起来后RC中想要向oem分区中拷贝文件，需要对oem分区进行挂载，但对oem分区进行挂载后会使原来的文件丢失无法访问。解决方法是取消对oem分区进行的操作，将需要使用的文件都拷贝到RC能够操作访问的目录 /odm/lib 和 /odm/bin 这两个目录。

## 6.2 文件访问的问题

因为需要不改动上层代码，所以文件还是需要放到原来所对应的位置，但在 RC 中拷贝文件的时候，很多文件原本的位置是无法直接操作的。解决办法，同一个分区下面的文件，在编译的时候创建软连接，这样软件烧录进盒子后，盒子对应的位置就会生成相应的软连接，我们只需将我们的文件放到指定的位置就行。

## 6.3 ms12 so的加载问题

当时直接拷贝ms12 的so，发现ms12的功能会有问题，AC4 的视频播放会有问题。后面排查的结果是，需要使用指定的 dolby_fw_dolbyms12 工具对 so进行操作，会对 so 进行打上 时间戳等相关的解密或加密等操作，会使得每次开机后，so 的 md5值都会发生变化

## 6.4 dolby_fw_dolbyms12签名失败

一般来说是两种可能

1. ms12的key没有烧
    
2. so库文件不匹配：举个例子 y4平台
    
    1. y4-B 芯片 ms12.so是 s4平台的
        
    2. y4-J 芯片 ms12.so是 s4d平台的