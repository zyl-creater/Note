adnl setkey password.bin
adnl bl1_boot -f u-boot.bin.usb.signed 
adnl bl2_boot -f u-boot.bin.usb.signed

adnl oem "disk_initial"

adnl oem "store boot_read bootloader 0x1080000 0 0x3f0000"
adnl upload -m mem -p 0x1080000 -z 0x3f0000 -f user-bootdump

adnl oem "store boot_read bootloader 0x1080000 1 0x3f0000"
adnl upload -m mem -p 0x1080000 -z 0x3f0000 -f boot0.dump

adnl oem "store boot_read bootloader 0x1080000 2 0x3f0000"
adnl upload -m mem -p 0x1080000 -z 0x3f0000 -f boot1.dump

adnl upload -p boot_a（分区名）-f D:\boot_a.dump -z 64M（分区大小）