# 开机logo

GTV wm项目中提供了新的开机logo实现方案：将开机logo集成在oem中
原本的集成方案是将logo分区集成在odm_ext中。当Googlebuild出软件时，子项目需要每次都在做包的过程中替换logo分区

新logo集成方案的改动
```diff
diff --git a/board/amlogic/configs/s4_ap222.h b/board/amlogic/configs/s4_ap222.h
index 1a49054355..8195340b67 100644
--- a/board/amlogic/configs/s4_ap222.h
+++ b/board/amlogic/configs/s4_ap222.h
@@ -118,7 +118,7 @@
         "active_slot=normal\0"\
         "boot_part=boot\0"\
         "vendor_boot_part=vendor_boot\0"\
-        "board_logo_part=odm_ext\0" \
+        "board_logo_part=oem\0" \
         "board=onn_4k_gtv\0"\
                "rollback_flag=0\0"\
        "boot_flag=0\0"\
```

`board_logo_part` 参数会影响后面加载logo文件的位置
```c++
/board/amlogic/configs/s4_ap222.h
		......
        "load_bmp_logo="\
            "if rdext4pic ${board_logo_part} $loadaddr; then bmp display $logoLoadAddr; " \
            "else if imgread pic logo bootup $loadaddr; then bmp display $bootup_offset; fi; fi;" \
            "\0"\
		......
```


rdext4pic 在全局中搜到的解释是
```c
cmd/amlogic/imgread.c



#if defined(CONFIG_CMD_EXT4) && defined(CONFIG_MMC_MESON_GX)
/*"if ext4load mmc 1:x ${dtb_mem_addr} /recovery/dtb.img; then echo cache dtb.img loaded; fi;"\*/
static int do_load_logo_from_ext4(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
{
    if (3 > argc) {
        errorP("argc(%d) < 3 illegle\n", argc);
        return CMD_RET_USAGE;
    }
    int iRet = 0;
    const char* ext4Part = argv[1];
    void* loadaddr = (void*)simple_strtoul(argv[2], NULL, 16);
	int autoSelectSlot = 1;//auto detect if need add _a/_b

    if (argc > 3) {
        env_set("ext4LogoPath", argv[3]);
    } else {
        env_set("ext4LogoPath", "/logo_files/bootup.bmp");
    }
	if (argc > 4) {
		const char *paraAutoSel = argv[4];

		autoSelectSlot = !memcmp(paraAutoSel, "true", 5);
		if (!autoSelectSlot && strcmp(paraAutoSel, "false")) {
			errorP("illegal para4 %s\n", paraAutoSel);
			return CMD_RET_FAILURE;
		}
	}

    if (!loadaddr) {
        errorP("illgle loadaddr %s\n", argv[2]);
        return CMD_RET_FAILURE;
    }

    if (BOOT_EMMC != store_get_type() ) {
        errorP("only support emmc, but store type %d\n", store_get_type() );
        return CMD_RET_FAILURE;
    }

    env_set("bootLogoPart", ext4Part);
	if (autoSelectSlot)
		run_command("if test ${active_slot} != normal; then setenv bootLogoPart ${bootLogoPart}${active_slot}; printenv bootLogoPart; fi", 0);
    const int partIndex = get_partition_num_by_name(env_get("bootLogoPart"));
    //获取分区索引。如果找不到，打印错误信息并返回`CMD_RET_FAILURE`。
    if (partIndex < 0) {
        errorP("fail find part index for name(%s)\n", env_get("bootLogoPart")); return CMD_RET_FAILURE;
    }
    env_set_hex("logoPart", partIndex);
    env_set_hex("logoLoadAddr", (ulong)loadaddr);
    env_set("ext4logoLoadCmd", "ext4load mmc 1:${logoPart} ${logoLoadAddr} ${ext4LogoPath}");
    iRet = run_command("printenv ext4logoLoadCmd; run ext4logoLoadCmd", 0);
    if (iRet) {
        errorP("Fail in load logo cmd\n"); return CMD_RET_FAILURE;
    }
    MsgP("load bmp from ext4 part okay\n");
    run_command("setenv ext4LogoSz ${filesize}", 0);
    const int bmpSz = env_get_hex("filesize", 0);
    if (bmpSz <= 0) {
        errorP("err bmp sz\n"); return CMD_RET_FAILURE;
    }

#if defined(CONFIG_GZIP)
    if (memcmp(loadaddr, gzip_magic, sizeof(gzip_magic))) {
        return CMD_RET_SUCCESS;
    }
    MsgP("gunzip bmp logo\n");
    void* uncompress = (char*)loadaddr + (((bmpSz + 0xf)>>4)<<4);
    unsigned long uncompSz = 0;
    iRet = imgread_uncomp_pic((unsigned char*)loadaddr, bmpSz, (unsigned char*)uncompress,
            CONFIG_MAX_PIC_LEN, (unsigned long*)&uncompSz);
    if (iRet) {
        errorP("Fail in uncomp pic,rc[%d]\n", iRet); return __LINE__;
    }
    if (uncompSz <= 0) {
        errorP("Fail uncompress logo bmp\n"); return CMD_RET_FAILURE;
    }
    memmove(loadaddr, uncompress, uncompSz);
    env_set_hex("ext4LogoSz", uncompSz);
#endif//#if defined(CONFIG_GZIP)

    return CMD_RET_SUCCESS;
}

U_BOOT_CMD_COMPLETE(
   rdext4pic,                   //read ext4 picture
	5,                           //maxargs
   0,                           //repeatable
   do_load_logo_from_ext4,      //command function
   "read logo bmp from ext4 part",           //description
   "    argv: rdext4pic <partName> <memAddr> <logoPath>\n"   //usage
   "    load bmp picture from <logoPath> of <partName> to <memAddr>.\n",
   var_complete
);
#endif// #if defined(CONFIG_CMD_EXT4) && defined(CONFIG_MMC_MESON_GX)
```
这段代码是U-Boot命令行接口的一部分，用于从EXT4分区加载BMP格式的Logo图片。以下是代码的详细解释：
- `/*"if ext4load mmc 1:x ${dtb_mem_addr} /recovery/dtb.img; then echo cache dtb.img loaded; fi;"\*/`：
    - 这行代码被注释掉了，但它展示了一个示例命令，用于从MMC设备加载文件。
- `static int do_load_logo_from_ext4(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])`：
    - 这是一个静态函数，用于处理`rdext4pic`命令。`cmd_tbl_t`是U-Boot中用于定义命令的结构体。
- `if (3 > argc) { ... }`：
    - 检查命令行参数的数量是否至少为3个。如果不是，打印错误信息并返回`CMD_RET_USAGE`。
- `const char* ext4Part = argv[1]; void* loadaddr = (void*)simple_strtoul(argv[2], NULL, 16);`：
    - 从命令行参数中提取EXT4分区的名称和内存地址。
- `if (argc > 3) { ... } else { ... }`：
    - 如果提供了第4个参数，将其设置为Logo文件的路径。如果没有提供，使用默认路径`/logo_files/bootup.bmp`。
- `if (!loadaddr) { ... }`：
    - 检查内存地址是否有效。如果无效，打印错误信息并返回`CMD_RET_FAILURE`。
- `if (BOOT_EMMC != store_get_type() ) { ... }`：
    - 确保存储类型是eMMC。如果不是，打印错误信息并返回`CMD_RET_FAILURE`。
- `env_set("bootLogoPart", ext4Part); ...`：
    - 设置环境变量，用于存储Logo分区的名称和加载地址。
- `env_set("ext4logoLoadCmd", "ext4load mmc 1:${logoPart}${logoLoadAddr} ${ext4LogoPath}"); ...`：
    - 构建用于加载Logo的命令，并执行它。

整体而言，这段代码定义了一个U-Boot命令，用于从EXT4分区加载BMP格式的Logo图片到指定的内存地址。

知识点：
## ext4load
ext4load 是一个用于从 ext4 文件系统中加载文件到内存的命令，通常在 U-Boot 环境下使用。该命令的基本语法如下：
```
ext4load <interface> [<dev[:part]> [addr [filename [bytes [pos]]]]]
```
其中，`<interface>` 是指设备接口，例如 mmc（MMC卡）；`<dev>` 是设备标识符，例如 0 或 1；`<part>` 是分区标识符，例如 1 或 2；`addr` 是目标内存地址；`filename` 是要加载的文件名；`bytes` 是要加载的字节数；`pos` 是文件的起始位置。

例如，以下命令从第0个存储设备的第2个分区的根目录读出 `uImage` 文件到内存地址 `0x40008000`：
`ext4load mmc 0:2 0x40008000 uImage`
这个命令会将 `uImage` 文件从指定的 ext4 分区加载到指定的内存地址，并且在 U-Boot 的提示符下执行该命令时，可以手动输入或者使用环境变量来指定文件名和地址。例如：

`ext4load mmc 0:2 ${loadaddr} /boot/uImage`

需要注意的是，ext4load 命令在使用时可能会遇到一些问题，例如文件大小不匹配或者加载失败的情况

# 开机动画
==本篇主要以GTV项目为例==
Android开机动画是在设备启动时显示的动画，通常包括品牌标志、加载进度条以及可能的品牌宣传元素。这个动画文件通常是放在设备的 `/product/media` 目录下，并且通常是一个名为 `bootanimation.zip` 的压缩文件。
## Google动画
Google动画在SDK中的位置：
```shell
vendor/google_gtvs_gtv/bootanimations/
├── bootanimation-gtv-4k.zip
└── bootanimation-gtv.zip
```

Google动画在系统中的加载位置：
vendor/google_gtvs_gtv/products/gtvs_gtv.mk
```mk

# Bootanimation
ifeq ($(GTV_BOOT_ANIMATION),4K)
    PRODUCT_COPY_FILES += vendor/google_gtvs_gtv/bootanimations/bootanimation-gtv-4k.zip:$(TARGET_COPY_OUT_PRODUCT)/media/bootanimation.zip
else
    PRODUCT_COPY_FILES += vendor/google_gtvs_gtv/bootanimations/bootanimation-gtv.zip:$(TARGET_COPY_OUT_PRODUCT)/media/bootanimation.zip
endif

```

## bootanimation.zip 内容解析
`bootanimation.zip` 文件包含以下内容：
1. 一个名为 `desc.txt` 的描述文件，它定义了动画的属性，比如帧率、循环次数以及动画的各个部分的顺序。
2. 动画帧，通常为PNG或JPEG格式的图片，按照 `desc.txt` 文件中的描述进行播放。

zip目录结构
```shell
bootanimation.zip
├── desc.txt
├── part0
│   ├── gtv_bootanim_000.png
│   ├── gtv_bootanim_001.png
│   ├── ...
└── part1
    ├── gtv_bootanim_025.png
    ├── gtv_bootanim_026.png
    ├── ...
```

`desc.txt` 文件的内容通常如下所示：
```txt
660 460 30
c 1 0 part0
f 0 0 part1 9
```
下面是对这些行的详细解释：
1. `660 460 30`：这一行定义了开机动画的分辨率和帧率。
    - `660`：动画的宽度（像素）。
    - `460`：动画的高度（像素）。
    - `30`：动画的帧率（每秒帧数）。
2. `c 1 0 part0`：这一行定义了一个动画部分，其中`c`表示该部分是“循环”的。
    - `1`：循环次数，这里设置为1次，表示这部分动画只会播放一次。
    - `0`：延迟时间（以毫秒为单位），这里设置为0，表示没有延迟。
    - `part0`：包含动画帧的目录名称。
3. `f 0 0 part1 9`：这一行定义了另一个动画部分，其中`f`表示该部分是“帧序列”。
    - `0`：循环次数，这里设置为0次，表示这部分动画会无限循环。
    - `0`：延迟时间，这里设置为0，表示没有延迟。
    - `part1`：包含动画帧的目录名称。
    - `9`：表示`part1`目录中的图片将以每秒9帧的速度播放。

根据这些定义，动画将首先播放`part0`目录中的图片一次，然后播放`part1`目录中的图片，并且会无限循环播放，播放速度为每秒9帧。

例如，如果你想要创建一个简单的开机动画，你可以按照以下步骤操作：
1. 准备动画帧：创建一系列PNG或JPEG图片，这些图片将按顺序播放形成动画。
2. 创建 `desc.txt` 文件：按照上述格式创建一个文本文件，描述动画的属性和播放顺序。
3. 创建 `bootanimation.zip`：将 `desc.txt` 文件和包含动画帧的目录一起压缩成 `bootanimation.zip` 文件。
4. 将 `bootanimation.zip` 文件 push 到设备的 `/product/media` 目录下，或者根据你的设备，可能需要放置到其他位置。
5. 重启设备，查看新的开机动画。

## 开机动画启动流程
[[BootAnimation启动流程]]

![[1729909429550_Pasted image 20240902155820.png]]
