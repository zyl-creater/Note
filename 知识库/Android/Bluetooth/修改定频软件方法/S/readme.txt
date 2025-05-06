1. 关avb2
adb reboot bootloader
fastboot flashing unlock
fastboot reboot
adb root
adb remount
adb reboot

2. 关selinux
开机串口按enter进入uboot cmd
setenv EnableSelinux permissive;saveenv
res

3. 修改系统文件
adb root
adb remount
adb push vendor /
adb shell
	chmod 0777 /vendor/bin/rtlbtmp && chmod 0777 /vendor/bin/rtwpriv && chmod 0777 /vendor/lib/hw/btmp.default.so
	chmod 0777 /vendor/firmware/mp_rtl8822cs_config && chmod 0777  /vendor/firmware/mp_rtl8822c_fw
	cd /system/app/Bluetooth
	mv Bluetooth.apk Bluetooth.apk_bak
	exit
adb install RtkWiFiTest-2.6.6_20210329.apk
adb reboot