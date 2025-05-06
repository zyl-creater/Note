# 编译userdebug
## luncher debug
source build/envsetup.sh
lunch BVS_2-userdebug

## build bootloader debug
cd bootloader/uboot-repo/
./mk s4_ap222 --avb2 --vab --userdebug --fastboot-write

cp build/u-boot.bin.device.signed ../../device/sdmc/YHT/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/YHT/upgrade/
cp build/s4_ap222-u-boot.aml.zip ../../device/sdmc/YHT/s4_ap222-u-boot.aml.zip
cd -

==(可选)更新bootloader编译记录，不能清除中间产物==
./device/amlogic/common/scripts/get_bootloader_version.sh bootloader/uboot-repo/bl33/v2019/ device/sdmc/YHT

../../sdk_patch/tool/sdmc_patch_creat.sh device/sdmc/YHT 0001-userdebug 1

==(可选)清除bootloader编译中间产物==
cd bootloader/uboot-repo/bl33/v2019
make distclean

## build Kernel debug
### 全编
croot
rm -rf device/sdmc/YHT-kernel/5.4
rm -rf out/android12-5.4
./device/amlogic/common/kernelbuild/mk YHT -v 5.4

### 单编驱动
特别注意 --modules $(相对路径)
./device/amlogic/common/kernelbuild/mk YHT -v 5.4 --modules vendor/wifi_driver/realtek/8822es/rtl88x2ES

## build aosp debug
### 单编oem
make custom_images -j32
cp -rfp out/target/product/YHT/oem.img device/sdmc/YHT/oem/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/BVS_2/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/MAG/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/Dcolor/

### 单编module
```sh
### 单编module
make <module> -j64

### 清obj/package以及其他安装文件，各种img、各种apk、各种bin、各种so等
make installclean

### 单清module的中间编译文件，清obj中的各种intermediates文件
make clean-<module>
```

### 全编aosp
source build/envsetup.sh
lunch BVS_2-userdebug
make -j64

make recoveryimage -j64
make otapackage -j64

---
# 编译user
## luncher user
source build/envsetup.sh
lunch BVS_2-user

## build bootloader user
cd bootloader/uboot-repo/
./mk s4_ap222 --avb2 --vab --fastboot-write
./mk s4_ap222 --avb2 --vab --fastboot-write --product

cp build/u-boot.bin.device.signed ../../device/sdmc/YHT/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/YHT/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/YHT/upgrade/
cp build/s4_ap222-u-boot.aml.zip ../../device/sdmc/YHT/s4_ap222-u-boot.aml.zip
cd -

## build Kernel user
### 全编
croot
rm -rf device/sdmc/YHT-kernel/5.4
rm -rf out/android12-5.4
./device/amlogic/common/kernelbuild/mk YHT -v 5.4 -t user

## build aosp user
### 单编oem
make custom_images -j32
cp -rfp out/target/product/YHT/oem.img device/sdmc/YHT/oem/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/BVS_2/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/MAG/
cp -rfp out/target/product/YHT/oem.img vendor/sdmc/oem_release_google/Dcolor/

### 全编aosp
source build/envsetup.sh
lunch BVS_2-user
make -j64

make otapackage -j64