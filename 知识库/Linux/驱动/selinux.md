在uboot下使用命令
开启selinux  `setenv EnableSelinux enforcing; saveenv;reset`
关闭selinux  `setenv EnableSelinux permissive; saveenv;reset`





对相应的文件添加权限
配置selinux权限文件    .te  文件
```te
# daemon_led seclabel is specified in daemon_led.rc
type daemon_led, domain;#domain是域
type daemon_led_exec, exec_type, vendor_file_type, file_type;
init_daemon_domain(daemon_led)
allow daemon_led input_device:chr_file { read write open entrypoint getattr};
allow daemon_led input_device:dir { search};
allow daemon_led sysfs:file { read write open};
```

进入  `system/sepolicy`  下 mm 编译 te 规则文件
在 ` out\target\product\ohm\vendor\etc\selinux ` 目录下会生成两个文件
vendor_file_contexts
vendor_sepolicy.cil

在 ` out\target\product\ohm\odm\etc\selinux ` 目录下会生成一个二进制文件
precompiled_sepolicy

将这三个文件推入到相应目录下


![[TE规则]]

