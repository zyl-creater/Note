不能直接放在init的hw目录下
要有其他文件引用
可以直接放在init目录下



![[Pasted image 20220825192115.png]]
可以放到这个目录下，相应的修改ohm.mk文件，添加相应的映射
![[Pasted image 20220825192228.png]]



**注意，当我们的 .rc 文件放在 init 目录下时，去修改我的们的文件名和文件后缀，还是能够识别到，要想文件不生效，只能移出文件夹备份到其他地方**
原因好像是，init下的文件都会存在内存中，内核会以流的形式去读取内容，再去进行判断



## 8、Android系统启动 init.rc分析

![[Android启动流程.png]]


![[Android系统启动_简略方法.png]]

### 8.1 Android系统大致启动流程

第一步：启动电源以及系统启动
当电源按下，引导芯片代码开始从预定义的地方（固化在ROM）开始执行。加载引导程序到RAM，然后 执行引导程序。

第二步：引导程序
引导程序是在Android操作系统开始运行前的一个小程序。引导程序是运行的第一个程序，因此它是针 对特定的主板与芯片的。设备制造商要么使用很受欢迎的引导程序比如redboot、uboot、qi bootloader或者开发自己的引导程序，它不是Android操作系统的一部分。引导程序是OEM厂商或者运 营商加锁和限制的地方。
引导程序分两个阶段执行。
第一个阶段，检测外部的RAM以及加载对第二阶段有用的程序；
第二阶段，引导程序设置网络、内存等等。这些对于运行内核是必要的，为了达到特殊的目标，引导程 序可以根据配置参数或者输入数据设置内核。
Android引导程序可以在`\bootable\bootloader\legacy\usbloader` 找到。传统的加载器包含两个文件， 需要在这里说明：
init.s初始化堆栈，清零BBS段，调用main.c的_main()函数；
main.c初始化硬件（闹钟、主板、键盘、控制台），创建linux标签

第三步：内核
Android内核与桌面linux内核启动的方式差不多。内核启动时，设置缓存、被保护存储器、计划列表， 加载驱动。当内核完成系统设置，它首先在系统文件中寻找”init”文件，然后启动root进程或者系统的第 一个进程

第四步：init进程
init进程是Linux系统中用户空间的第一个进程，进程号固定为1。Kernel启动后，在用户空间启动init进 程，并调用init中的main()方法执行init进程的职责。

第五步：启动zygote进程，通过zygote启动SystemServer，当服务都启动完成之后，启动Lancher App，然后退出开机动画。我们就可以看到桌面应用，此时启动过程就结束了。

### 8.2 启动init进程
init进程是Android系统中及其重要的第一个进程，是由内核拉起来的第一个用户进程。

init进程主要完成了三件事情。
- 创建和挂载启动所需要的文件目录
- 初始化和启动属性服务
- 解析init.rc配置文件并启动Zygote进程

此处自行查阅源码
/system/core/init/init.cpp

### 8.3 解析init.rc

init.rc是一个非常重要的配置文件，它是由Android初始化语言（Android Init Language）编写的脚本，它主要包含五种类型语句：Action（Action中包含了一系列的Command）、Commands（init语言中的命令）、Services（由init进程启动的服务）、Options（对服务进行配置的选项）和Import（引入其他配置文件）。init.rc的配置代码如下所示。

```
on init
    setprop persist.vendor.zyl 1

service zyl_daemon /vendor/bin/daemon_led
    class late_start
    user root
    group root
    disabled
    oneshot

on property:sys.boot_completed=1 #上电开机
    #insmod /vendor/lib/modules/dts_sys_led.ko

on property:persist.vendor.zyl=1
    insmod /vendor/lib/modules/dts_sys_led.ko
    start zyl_daemon

on property:persist.vendor.zyl=0
    rmmod /vendor/lib/modules/dts_sys_led.ko
```

#### Action
Action： 通过触发器trigger，即以on开头的语句来决定执行相应的service的时机，具体有如下时机：
- on early-init; 在初始化早期阶段触发；
- on init; 在初始化阶段触发；
- on late-init; 在初始化晚期阶段触发；
- on boot/charger： 当系统启动/充电时触发，还包含其他情况，此处不一一列举；
- on property:=: 当属性值满足条件时触发

#### Service
服务Service，以 service开头，由init进程启动，一般运行在init的一个子进程，所以启动service前需要判断对应的可执行文件是否存在。init生成的子进程，定义在rc文件，其中每一个service在启动时会通过 fork方式生成子进程。
例如： service servicemanager /system/bin/servicemanager代表的是服务名为 servicemanager，服务执行的路径为/system/bin/servicemanager。

#### Command
下面列举常用的命令:
- class_start <service_class_name>： 启动属于同一个class的所有服务；
- start <service_name>： 启动指定的服务，若已启动则跳过；
- stop <service_name>： 停止正在运行的服务
- setprop ：设置属性值
- mkdir ：创建指定目录
- symlink <sym_link>： 创建连接到的<sym_link>符号链接；
- write ： 向文件path中写入字符串；
- exec： fork并执行，会阻塞init进程直到程序完毕；
- exprot：设定环境变量；
- loglevel ：设置log级别

#### Options
Options是Service的可选项，与service配合使用
- disabled: 不随class自动启动，只有根据service名才启动；
- oneshot: service退出后不再重启；
- user/group： 设置执行服务的用户/用户组，默认都是root；
- class：设置所属的类名，当所属类启动/退出时，服务也启动/停止，默认为default； onrestart:当服务重启时执行相应命令；
- socket: 创建名为 /dev/socket/的socket
- critical: 在规定时间内该service不断重启，则系统会重启并进入恢复模式
- default: 意味着disabled=false，oneshot=false，critical=false。

### 8.4 启动zygote

### 8.5 SystemServer启动

![[Pasted image 20220830171557.png]]

### 8.6 启动FallbcakHome和Launcher

