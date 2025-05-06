## HDMI调试命令
在`/dev/dri/` 目录下可以看到驱动注册的各个显卡，`DRM`设备节点为 `/dev/dri/cardX`，`X`为`0-15`的数值。

```shell
console:/ # ls -l /dev/dri/
total 0
crw-rw-rw- 1 root graphics 226,   0 1969-12-31 19:00 card0
crw-rw-rw- 1 root graphics 226, 128 1969-12-31 19:00 renderD128

console:/ # ls -l /sys/class/drm
total 0
lrwxrwxrwx 1 root root    0 2024-09-20 03:39 card0 -> ../../devices/platform/drm-subsystem/drm/card0
lrwxrwxrwx 1 root root    0 2024-09-20 03:39 card0-HDMI-A-1 -> ../../devices/platform/drm-subsystem/drm/card0/card0-HDMI-A-1
lrwxrwxrwx 1 root root    0 2024-09-20 03:39 card0-Writeback-1 -> ../../devices/platform/drm-subsystem/drm/card0/card0-Writeback-1
lrwxrwxrwx 1 root root    0 2024-09-20 03:39 renderD128 -> ../../devices/platform/drm-subsystem/drm/renderD128
lrwxrwxrwx 1 root root    0 2024-09-20 03:39 ttm -> ../../devices/virtual/drm/ttm
-r--r--r-- 1 root root 4096 2024-09-20 03:39 version
```

`sysfs`文件系统中的`card0-HDMI-A-1`代表的是`hdmi`显示设备，是由`drm_sysfs_connector_add`函数创建的。查看`card0-HDMI-A-1`目录结构；

```shell
console:/ # ls -l /sys/class/drm/card0-HDMI-A-1/
total 0
lrwxrwxrwx 1 root root    0 2024-09-20 03:40 device -> ../../card0
-r--r--r-- 1 root root 4096 2024-09-20 03:40 dpms
-r--r--r-- 1 root root    0 2024-09-20 03:40 edid
-r--r--r-- 1 root root 4096 2024-09-20 03:40 enabled
-r--r--r-- 1 root root 4096 2024-09-20 03:40 modes
drwxr-xr-x 2 root root    0 2024-09-20 03:40 power
-rw-r--r-- 1 root root 4096 2024-09-20 03:40 status
lrwxrwxrwx 1 root root    0 2024-09-20 03:40 subsystem -> ../../../../../../class/drm
-rw-r--r-- 1 root root 4096 2024-09-20 03:40 uevent
-r--r--r-- 1 root root 4096 2024-09-20 03:40 waiting_for_supplier
```

其中：
- `device`：指向`card0`；
- `edid`：存储`hdmi`显示器的扩展显示标识数据；
- `enabled`：`hdmi`接口是否被启用或禁用；
- `modes`：连接的`hdmi`显示器以及当前`hdmi`控制器同时支持的分辨率列表；
- `status`：`hdmi`接口连接状态的信息；

需要注意：下文中提到的分辨率，指的是就是显示模式。

### 查看`hdmi`使能状态

查看`hdmi`输出使能状态：

```shell
console:/ # cat /sys/class/drm/card0-HDMI-A-1/enabled
enabled
```

如果将`hdmi`线拔掉：

```shell
console:/ # cat /sys/class/drm/card0-HDMI-A-1/enabled
disabled
```

在使用`cat`命令读取`enabled`文件时调用`enabled_show`方法；

```c
static ssize_t enabled_show(struct device *device,
                            struct device_attribute *attr,
                           char *buf)
{
        struct drm_connector *connector = to_drm_connector(device);
        bool enabled;

        enabled = READ_ONCE(connector->encoder);

        return sysfs_emit(buf, enabled ? "enabled\n" : "disabled\n");
}
```

### 查看`hdmi`连接状态

查看`hdmi`的插拔连接状态；

```shell
console:/ # cat /sys/class/drm/card0-HDMI-A-1/status
connected
```

如果将`hdmi`线拔掉：

```shell
console:/ # cat /sys/class/drm/card0-HDMI-A-1/status
disconnected
```

在使用`cat`命令读取`status`文件时调用`status_show`方法；

```c
static ssize_t status_show(struct device *device,
                           struct device_attribute *attr,
                           char *buf)
{
        struct drm_connector *connector = to_drm_connector(device);
        enum drm_connector_status status;

        status = READ_ONCE(connector->status);

        return sysfs_emit(buf, "%s\n",
                          drm_get_connector_status_name(status));
}

```

### 查看`edid`

通过如下命令可以查看`edid`信息，一共256个字节；
```
console:/ # cat /sys/class/amhdmitx/amhdmitx0/rawedid                          
00ffffffffffff006318000000000000091e0103800000780ad7a5a2594a9624145054a3080081c00101010101010101010101010101662156aa51001e30468f33003f432100001e023a801871382d40582c45003f432100001a000000fd001e4c1e5a1e000a202020202020000000fc004141410a2020202020202020200121020324715090050403070206011f141312161115202309070366030c0010000083010000011d007251d01e206e285500c48e2100001e011d8018711c1620582c2500c48e2100009e8c0ad08a20e02d10103e9600138e2100001800000000000000000000000000000000000000000000000000000000000000000000000000ee

console:/ # cat /sys/class/amhdmitx/amhdmitx0/edid                             
Rx Manufacturer Name: XXX
Rx Product Code: 0000
Rx Serial Number: 00000000
Rx Product Name: AAA
Manufacture Week: 9
Manufacture Year: 2020
Physcial size(mm): 708 x 398
EDID Version: 1.3
EDID block number: 0x1
blk0 chksum: 0x21
Source Physical Address[a.b.c.d]: 1.0.0.0
native Mode 71, VIC (native 16):
ColorDeepSupport 0
16 5 4 3 7 2 6 1 31 20 19 18 22 17 21 32 
Audio {format, channel, freq, cce}
{1, 1, 7, 3}
Speaker Allocation: 1
Vendor: 0xc03 ( HDMI device)
MaxTMDSClock1 150 MHz
vLatency:  Invalid/Unknown
aLatency:  Invalid/Unknown
i_vLatency:  Invalid/Unknown
i_aLatency:  Invalid/Unknown
SCDC: 0
RR_Cap: 0
LTE_340M_Scramble: 0

checkvalue: 0x21ee0000
```

或者使用
```shell
console:/ # cat /sys/class/drm/card0-HDMI-A-1/edid > /data/edid.bin
```

这里我们尝试通过`EDID Manager`工具去解析，首先需要去下载`EDID Manager`工具，然后将`edid.bin`下载到`windows`系统上，并加载文件解析内容如下：
```


			Time: 16:16:50
			Date: 2024年9月20日
			EDID Manager Version: 1.0.0.14
	___________________________________________________________________

	Block 0 (EDID Base Block), Bytes 0 - 127,  128  BYTES OF EDID CODE:

		        0   1   2   3   4   5   6   7   8   9   
		000  |  00  FF  FF  FF  FF  FF  FF  00  63  18
		010  |  00  00  00  00  00  00  09  1E  01  03
		020  |  80  00  00  78  0A  D7  A5  A2  59  4A
		030  |  96  24  14  50  54  A3  08  00  81  C0
		040  |  01  01  01  01  01  01  01  01  01  01
		050  |  01  01  01  01  66  21  56  AA  51  00
		060  |  1E  30  46  8F  33  00  3F  43  21  00
		070  |  00  1E  02  3A  80  18  71  38  2D  40
		080  |  58  2C  45  00  3F  43  21  00  00  1A
		090  |  00  00  00  FD  00  1E  4C  1E  5A  1E
		100  |  00  0A  20  20  20  20  20  20  00  00
		110  |  00  FC  00  41  41  41  0A  20  20  20
		120  |  20  20  20  20  20  20  01  21

(8-9)    	ID Manufacture Name : XXX
(10-11)  	ID Product Code     : 0000
(12-15)  	ID Serial Number    : 0
(16)     	Week of Manufacture : 9
(17)     	Year of Manufacture : 2020

(18)     	EDID Version Number : 1
(19)     	EDID Revision Number: 3

(20)     	Video Input Definition       : Digital
(21)     	Maximum Horizontal Image Size: 0 cm
(22)     	Maximum Vertical Image Size  : 0 cm
(23)     	Display Gamma                : 2.20
(24)     	Power Management and Supported Feature(s):
			RGB Color, Non-sRGB, Preferred Timing Mode

(25-34)  	Color Characteristics
			Red Chromaticity   :  Rx = 0.636  Ry = 0.345
			Green Chromaticity :  Gx = 0.290  Gy = 0.589
			Blue Chromaticity  :  Bx = 0.143  By = 0.080
			Default White Point:  Wx = 0.313  Wy = 0.329

(35)     	Established Timings I

			720 x 400 @ 70Hz (IBM, VGA)
			640 x 480 @ 60Hz (IBM, VGA)
			800 x 600 @ 56Hz (VESA)
			800 x 600 @ 60Hz (VESA)

(36)     	Established Timings II

			1024 x 768 @ 60Hz (VESA)

(37)     	Manufacturer's Timings (Not Used)

(38-53)  	Standard Timings

			1280x720 @ 60 Hz (16:9 Aspect Ratio)

(54-71)  	Detailed Descriptor #1: Preferred Detailed Timing (1366x768 @ 60Hz)

			Pixel Clock            : 85.5 MHz
			Horizontal Image Size  : 575 mm
			Vertical Image Size    : 323 mm
			Refresh Mode           : Non-interlaced
			Normal Display, No Stereo

			Horizontal:
				Active Time     : 1366 Pixels
				Blanking Time   : 426 Pixels
				Sync Offset     : 70 Pixels
				Sync Pulse Width: 143 Pixels
				Border          : 0 Pixels
				Frequency       : 47 kHz

			Vertical:
				Active Time     : 768 Lines
				Blanking Time   : 30 Lines
				Sync Offset     : 3 Lines
				Sync Pulse Width: 3 Lines
				Border          : 0 Lines

			Digital Separate, Horizontal Polarity (+), Vertical Polarity (+)

			Modeline: "1366x768" 85.500 1366 1436 1579 1792 768 771 774 798 +hsync +vsync

(72-89)  	Detailed Descriptor #2: Detailed Timing (1920x1080 @ 60Hz)

			Pixel Clock            : 148.5 MHz
			Horizontal Image Size  : 575 mm
			Vertical Image Size    : 323 mm
			Refresh Mode           : Non-interlaced
			Normal Display, No Stereo

			Horizontal:
				Active Time     : 1920 Pixels
				Blanking Time   : 280 Pixels
				Sync Offset     : 88 Pixels
				Sync Pulse Width: 44 Pixels
				Border          : 0 Pixels
				Frequency       : 67 kHz

			Vertical:
				Active Time     : 1080 Lines
				Blanking Time   : 45 Lines
				Sync Offset     : 4 Lines
				Sync Pulse Width: 5 Lines
				Border          : 0 Lines

			Digital Separate, Horizontal Polarity (+), Vertical Polarity (-)

			Modeline: "1920x1080" 148.500 1920 2008 2052 2200 1080 1084 1089 1125 +hsync -vsync

(90-107) 	Detailed Descriptor #3: Monitor Range Limits

			Horizontal Scan Range: 30kHz-90kHz
			Vertical Scan Range  : 30Hz-76Hz
			Supported Pixel Clock: 300 MHz
			Secondary GTF        : Not Supported

(108-125)	Detailed Descriptor #4: Monitor Name

			Monitor Name: AAA

(126-127)	Extension Flag and Checksum

			Extension Block(s)  : 1
			Checksum Value      : 33

	___________________________________________________________________

	Block 1 ( CEA-861 Extension Block), Bytes 128 - 255,  128  BYTES OF EDID CODE:

		        0   1   2   3   4   5   6   7   8   9   
		128  |  02  03  24  71  50  90  05  04  03  07
		138  |  02  06  01  1F  14  13  12  16  11  15
		148  |  20  23  09  07  03  66  03  0C  00  10
		158  |  00  00  83  01  00  00  01  1D  00  72
		168  |  51  D0  1E  20  6E  28  55  00  C4  8E
		178  |  21  00  00  1E  01  1D  80  18  71  1C
		188  |  16  20  58  2C  25  00  C4  8E  21  00
		198  |  00  9E  8C  0A  D0  8A  20  E0  2D  10
		208  |  10  3E  96  00  13  8E  21  00  00  18
		218  |  00  00  00  00  00  00  00  00  00  00
		228  |  00  00  00  00  00  00  00  00  00  00
		238  |  00  00  00  00  00  00  00  00  00  00
		248  |  00  00  00  00  00  00  00  EE

(128-130)	Extension Header

			Revision Number    :	3
			DTD Starting Offset:	36

(131)    	Display Support

			Basic Audio, YCbCr 4:4:4, YCbCr 4:2:2
			Number of Native Formats: 1

(132-148)	Video Data Block

			1920x1080p @ 59.94/60Hz - HDTV (16:9, 1:1) [Native]
			1920x1080i @ 59.94/60Hz - HDTV (16:9, 1:1)
			1280x720p @ 59.94/60Hz - HDTV (16:9, 1:1)
			720x480p @ 59.94/60Hz - EDTV (16:9, 32:27)
			720(1440)x480i @ 59.94/60Hz - SDTV (16:9, 32:27)
			720x480p @ 59.94/60Hz - EDTV (4:3, 8:9)
			720(1440)x480i @ 59.94/60Hz - SDTV (4:3, 8:9)
			640x480p @ 59.94/60Hz - EDTV (4:3, 1:1)
			1920x1080p @ 50Hz - HDTV (16:9, 1:1)
			1920x1080i @ 50Hz - HDTV (16:9, 1:1)
			1280x720p  @ 50Hz - HDTV  (16:9, 1:1)
			720x576p @ 50Hz - EDTV (16:9, 64:45)
			720(1440)x576i @ 50Hz - SDTV (16:9, 64:45)
			720x576p @ 50Hz - EDTV (4:3, 16:15)
			720(1440)x576i @ 50Hz - SDTV (4:3, 16:15)
			1920x1080p @ 23.97/24Hz - HDTV(16:9, 1:1)

(149-152)	Audio Data Block

			Audio Format #1    : LPCM, 2-Channel, 20-Bit, 16-Bit
			Sampling Frequency : 48 kHz, 44.1 kHz, 32 kHz

(153-159)	Vendor Specific Data Block (VSDB)

			IEEE Registration Identifier: 0x000C03
			CEC Physical Address        : 0x0010

(160-163)	Speaker Allocation Data Block (SADB)

			Front Left/Front Right Audio Channel (FL/FR)

(164-181)	Detailed Descriptor #1: Detailed Timing (1280x720 @ 60Hz 16:9 Apsect Ratio)

			Pixel Clock            : 74.25 MHz
			Horizontal Image Size  : 708 mm
			Vertical Image Size    : 398 mm
			Refresh Mode           : Non-interlaced
			Normal Display, No Stereo

			Horizontal:
				Active Time     : 1280 Pixels
				Blanking Time   : 370 Pixels
				Sync Offset     : 110 Pixels
				Sync Pulse Width: 40 Pixels
				Border          : 0 Pixels
				Frequency       : 45 kHz

			Vertical:
				Active Time     : 720 Lines
				Blanking Time   : 30 Lines
				Sync Offset     : 5 Lines
				Sync Pulse Width: 5 Lines
				Border          : 0 Lines

			Digital Separate, Horizontal Polarity (+), Vertical Polarity (+)

			Modeline: "1280x720" 74.250 1280 1390 1430 1650 720 725 730 750 +hsync +vsync

(182-199)	Detailed Descriptor #2: Detailed Timing (1920x1080i @ 60Hz 16:9 Apsect Ratio)

			Pixel Clock            : 74.25 MHz
			Horizontal Image Size  : 708 mm
			Vertical Image Size    : 398 mm
			Refresh Mode           : Interlaced
			Normal Display, No Stereo

			Horizontal:
				Active Time     : 1920 Pixels
				Blanking Time   : 280 Pixels
				Sync Offset     : 88 Pixels
				Sync Pulse Width: 44 Pixels
				Border          : 0 Pixels
				Frequency       : 33 kHz

			Vertical:
				Active Time     : 540 Lines
				Blanking Time   : 22 Lines
				Sync Offset     : 2 Lines
				Sync Pulse Width: 5 Lines
				Border          : 0 Lines

			Digital Separate, Horizontal Polarity (+), Vertical Polarity (+)

			Modeline: "1920x1080i" 74.250 1920 2008 2052 2200 1080 1084 1094 1124 interlace +hsync +vsync

(200-217)	Detailed Descriptor #3: Detailed Timing (720x480 @ 60Hz 4:3 Apsect Ratio)

			Pixel Clock            : 27 MHz
			Horizontal Image Size  : 531 mm
			Vertical Image Size    : 398 mm
			Refresh Mode           : Non-interlaced
			Normal Display, No Stereo

			Horizontal:
				Active Time     : 720 Pixels
				Blanking Time   : 138 Pixels
				Sync Offset     : 16 Pixels
				Sync Pulse Width: 62 Pixels
				Border          : 0 Pixels
				Frequency       : 31 kHz

			Vertical:
				Active Time     : 480 Lines
				Blanking Time   : 45 Lines
				Sync Offset     : 9 Lines
				Sync Pulse Width: 6 Lines
				Border          : 0 Lines

			Digital Separate, Horizontal Polarity (-), Vertical Polarity (-)

			Modeline: "720x480" 27.000 720 736 798 858 480 489 495 525 -hsync -vsync

(218-235)	Detailed Descriptor #4: Defined by Manufacturer

(236-253)	Detailed Descriptor #5: Defined by Manufacturer

(254)    	Post DTD Padding

			Residual Byte Padding: 00

(255)    	Checksum Value: 238

	___________________________________________________________________

```
