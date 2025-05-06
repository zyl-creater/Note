# 沃尔玛sdk下载
# 直接下载
repo init -u ssh://git@source.amlogic.com/wm-s805x2-s/platform/manifest.git -m google_gretzky_sdmc_wave1.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git

.repo/repo/repo sync -j8


# 下载一个mirror镜像
cd ~/workspace/s905y4/s/wm/google_gretzky_mirror/
repo init -u ssh://git@source.amlogic.com/wm-s805x2-s/platform/manifest.git -m google_gretzky_sdmc_wave1.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --mirror

.repo/repo/repo sync -j8

# 从mirror镜像拷贝
## 及时mirror是旧的，但是这样sync的代码也是可以sync到最新的版本
repo init -u ssh://git@source.amlogic.com/wm-s805x2-s/platform/manifest.git -m google_gretzky_sdmc_wave1.xml --repo-url=ssh://git@source.amlogic.com/tools/repo.git --reference=/home/jenkins/workspace/wm/gretzky_sdk_from_amlogic/wave1/wave1_mirror_server

.repo/repo/repo sync -j8

### 特别注意！！！！！！！
```shell
# 在下载完sdk后，需要手动revert这个patch；这个patch是删了几个文件，去掉会导致编译不过；
# 为什么去掉，这是amlogic D代码服务器的问题，据aml的描述是服务器的代码审核机制导致这几个文件过不了审；
# google编译的服务器和D代码服务器不是同一个；
cd vendor/amlogic/common
git revert d642f3a74973a933c58ec54ed17d7959bf9db6ff
```


---

## userdbug

### build bootloader
cd bootloader/uboot-repo/
./mk s4_ap222 --avb2 --vab --userdebug --fastboot-write

### 拷贝签过名的bootloader到当前sdk
cp build/u-boot.bin.device.signed ../../device/sdmc/YOC/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/YOC/upgrade/
cp build/s4_ap222-u-boot.aml.zip ../../device/sdmc/YOC/s4_ap222-u-boot.aml.zip


#### (可选)更新bootloader编译记录，不能清除中间产物
cd ../../
./device/amlogic/common/scripts/get_bootloader_version.sh bootloader/uboot-repo/bl33/v2019/ device/sdmc/YOC

../../sdk_patch/tool/sdmc_patch_creat.sh device/sdmc/YOC 0001-userdebug 1

#### (可选)清除bootloader编译中间产物
cd bootloader/uboot-repo/bl33/v2019
make distclean

### build Kernel
#### 全编
croot
rm -rf device/sdmc/YOC-kernel/5.4
rm -rf out/android12-5.4
./device/amlogic/common/kernelbuild/mk YOC -v 5.4

#### 单编驱动
特别注意 --modules $(相对路径)
./device/amlogic/common/kernelbuild/mk YOC -v 5.4 --modules vendor/wifi_driver/realtek/8822cs/rtl88x2CS

### build aosp

#### 单编oem
make custom_images -j8
cp -rfp out/target/product/YOC/oem.img device/sdmc/YOC/oem/

#### 单编module
```shell
# 单编module
make <module> -j64
# 清obj/package以及其他安装文件，各种img、各种apk、各种bin、各种so等
make installclean
# 单清module的中间编译文件，清obj中的各种intermediates文件
make clean-<module>
```

#### 全编aosp
source build/envsetup.sh
lunch onn_4k_gtv-userdebug
make -j64

make recoveryimage -j64
make otapackage -j64

#### 清除中间文件
make installclean

#### 清除obj中间文件

```shell
make clean-<target>
```

---

## user版本

### build bootloader
cd bootloader/uboot-repo/
./mk s4_ap222 --avb2 --vab --fastboot-write --product

```shell
# 拷贝签过名的bootloader到当前sdk
cp build/u-boot.bin.device.signed ../../device/sdmc/YOC/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/YOC/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/YOC/upgrade/
cp build/s4_ap222-u-boot.aml.zip ../../device/sdmc/YOC/s4_ap222-u-boot.aml.zip
```

#### (可选)更新bootloader编译记录，不能清除中间产物
croot
./device/amlogic/common/scripts/get_bootloader_version.sh bootloader/uboot-repo/bl33/v2019/ device/sdmc/YOC

../../sdk_patch/tool/sdmc_patch_creat.sh device/sdmc/YOC 0001-user 1

#### (可选)清除bootloader编译中间产物
cd bl33/v2019
make distclean

### build Kernel
```shell
rm -rf device/sdmc/YOC-kernel/5.4
rm -rf out/android12-5.4
./device/amlogic/common/kernelbuild/mk YOC -v 5.4 -t user
```


### build aosp

source build/envsetup.sh
lunch onn_4k_gtv-user

make -j64

make otapackage -j64