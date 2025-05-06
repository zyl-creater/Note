## 开启kernel打印
```
setenv loglevel 8
saveenv
```

## 关闭kernel打印
```
setenv loglevel 0
saveenv
```

## 开放所有打印
```
setenv loglevel 8
setenv initargs $initargs printk.devkmsg=on
saveenv
```

## 重启
```
reset
```

## 进入系统
```
run bootcmd
run storeboot
```

## 查看环境变量
```
printenv

printenv otg_device
```

## 修改环境变量
```
setenv 
setenv otg_device 1
```
**注意：每次修改环境变量后都要保存修改才能生效**

## 保存修改
```
saveenv
```

## 还原环境
```
env default -a
saveenv
```

## 关闭SE
```
setenv EnableSelinux enforcing
setenv EnableSelinux permissive
saveenv
```

## 开启SE
```
setenv EnableSelinux enforcing
saveenv
```

## 替换bootloader
```
usb_update bootloader bootloader.img
```

## 查看dts
```
fdt addr $dtb_mem_addr
fdt print
```

## 修改dts
```
fdt set /cvbsout status "okay"
fdt print /cvbsout
run bootcmd
```
**注意：修改dts后不能使用reset重启**

## 查看MAC
```
keyman read mac ${loadaddr} str
print mac
```

