adb reboot bootloader   （会停留在开机logo界面）
fastboot oem oem_unlock
fastboot flashing unlock_critical
fastboot flashing unlock
fastboot reboot			（执行完后机器会重启）
adb root				（进入系统后，再执行此命令）
adb disable-verity
adb reboot				（执行完后机器会重启）
adb root				（进入系统后，再执行此命令）
adb shell setenforce 0
adb remount