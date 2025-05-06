## userdbug

### build bootloader
cd bootloader/uboot-repo/
./mk s7d_bm202 --avb2 --vab --userdebug --fastboot-write


### 拷贝带签名的bootloader
cp build/u-boot.bin.device.signed ../../device/sdmc/coffey/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/coffey/upgrade/
cp build/s7d_bm202-u-boot.aml.zip ../../device/sdmc/coffey/upgrade/


#### (可选)更新bootloader编译记录，不能清除中间产物
cd ../../
./device/amlogic/common/scripts/get_bootloader_version.sh bootloader/uboot-repo/bl33/v2023/ device/sdmc/coffey

#### (可选)清除bootloader编译中间产物
cd bootloader/uboot-repo/bl33/v2023
make distclean

### build Kernel
#### 全编

./mk clean
./mk coffey -v common14-5.15

# 编译产物路径
cd common/common14-5.15/out
cat ./bazel/output_user_root/5770733ef6858b09b81240213df38bf3/sandbox/linux-sandbox/46/execroot/__main__/out/android14-5.15/common/.config | grep CONFIG_DEBUG_FS


#### 单编驱动
特别注意 --modules $(相对路径)
./mk coffey -v common14-5.15 --modules common/driver_modules/wifi_bt/wifi/realtek/8822cs/rtl88x2CS

### build aosp

#### 单编oem
make custom_images -j16
cp out/target/product/coffey/oem.img device/sdmc/coffey/oem/
cp out/target/product/coffey/oem.img vendor/sdmc/oem_release_google/coffey/


#### 单编module

### 单编module
make <module> -j64
# 清obj/package以及其他安装文件，各种img、各种apk、各种bin、各种so等
make installclean
# 单清module的中间编译文件，清obj中的各种intermediates文件
make clean-<module>


#### 编amlogic所有模块
tools/bazel build //common:amlogic_dist --sandbox_debug --verbose_failures --lto=thin
#### 编外置模块
tools/bazel build @//driver_modules/gpioctrl:gpioctrl --sandbox_debug --verbose_failures --lto=thin
tools/bazel build @//driver_modules/media_modules:media --sandbox_debug --verbose_failures --lto=thin




#### 全编aosp

source build/envsetup.sh && lunch coffey-userdebug
make -j64
make recoveryimage -j64
make otapackage -j64
make installclean && make -j64 && make otapackage -j64

#### 清除中间文件

make installclean

#### 清除obj中间文件

make clean-<target>


---

## user版本

### build bootloader
cd bootloader/uboot-repo/
./mk s7d_bm202 --avb2 --vab --fastboot-write
./mk s7d_bm202 --avb2 --vab --fastboot-write --product


# 拷贝带签名的bootloader
cp build/u-boot.bin.device.signed ../../device/sdmc/coffey/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.usb.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.sd.bin.device.signed ../../device/sdmc/coffey/upgrade/
cp build/u-boot.bin.usb.device.signed ../../device/sdmc/coffey/upgrade/





#### (可选)更新bootloader编译记录，不能清除中间产物
croot
./device/amlogic/common/scripts/get_bootloader_version.sh bootloader/uboot-repo/bl33/v2023/ device/sdmc/coffey

#### (可选)清除bootloader编译中间产物
cd bl33/v2023
make distclean

### build Kernel

./mk clean
./mk coffey -v common14-5.15 -t user


### build aosp

source build/envsetup.sh
lunch coffey-user
make -j64
make recoveryimage -j64
make otapackage -j64
make installclean && make -j64 && make otapackage -j64



S905X5M

# cd bootloader/uboot-repo

$ ./fip/s7d/generate-device-keys/gen_all_device_key.sh --key-dir ./dv_scs_keys --rsa-size 4096 --project s905x5m --rootkey-index 0 --template-dir ./soc/templates/s7d --out-dir ./device-keys

$ ./fip/s7d/generate-device-keys/export_signing_keys_and_sign_template.sh --rootkey-index 0 --key-dir ./dv_scs_keys --project s905x5m --template-dir ./soc/templates/s7d --out-dir ./device-keys --arb-config ./bl33/v2023/board/amlogic/s7d_bm202/fw_arb.cfg

usb password:

#生成dvuk, 用于生成usb password.bin

$ mkdir -p ./dv_scs_keys/root/dvuk/s905x5m/

$ ./fip/s7d/generate-device-keys/bin/dvuk_gen.sh  ./dv_scs_keys/root/dvuk/s905x5m/dvuk

$ ./fip/s7d/binary-tool/vendor-keytool gen-usb-passwd --chipset=SC2 --mrk-file=./dv_scs_keys/root/dvuk/s905x5m/dvuk.bin | xxd -r -p > password.bin

$ ./fip/s7d/bin/efuse-gen.sh \

    --enable-device-vendor-scs true \

    --device-roothash ./dv_scs_keys/root/rsa/s905x5m/roothash/hash-device-rootcert.bin \

    --dvgk ./dv_scs_keys/root/dvgk/s905x5m/dvgk.bin \

    --dvuk ./dv_scs_keys/root/dvuk/s905x5m/dvuk.bin \

    --enable-usb-password true \

    --enable-dif-password true \

    --enable-dvuk-derive-with-cid true \

    -o pattern.efuse


