1. 核心内核组件
	
    1. arch 按CPU架构区分的硬件相关代码（如ARM/AArch64/X86），包含启动代码、中断处理、内存管理。
        
        1. arch/arm64
            
            1. boot/ 引导与设备树
                
            2. configs/ 硬件平台预设配置
                
            3. crypto/ 加密算法优化
                
            4. hyperv/ Hyper-V虚拟化支持
                
            5. include/ 架构相关头文件
                
            6. kernel/ 核心功能（进程/中断/调度）
                
            7. kvm/ KVM虚拟化模块
                
            8. mm/ 内存管理（页表/虚拟内存）
                
            9. tools/ 调试与分析工具
                
            10. xen/ Xen虚拟化接口
                
    2. block 块设备层核心逻辑（I/O调度、请求队列管理）。
        
    3. drivers 设备驱动集合（如GPU、USB、网络设备驱动）。
        
        1. 核心硬件支持
            
            1. acpi 高级配置与电源接口（ACPI）驱动，用于管理电源、热插拔和硬件配置。
                
            2. amba ARM AMBA总线设备驱动（如PrimeCell外设）。
                
            3. base 核心基础设施驱动（如设备模型、sysfs、固件加载）。
                
            4. clk 时钟管理驱动，控制硬件时钟源和分频。
                
            5. dma 直接内存访问（DMA）控制器驱动，管理内存与设备间的高速数据传输。
                
            6. gpio 通用输入输出（GPIO）控制器驱动，控制引脚电平。
                
            7. i2c I²C总线驱动，支持传感器、EEPROM等I²C设备。
                
            8. spi SPI总线驱动，用于Flash存储、显示屏控制器等SPI设备。
                
            9. pci PCI/PCIe总线驱动，管理PCI设备（如显卡、网卡）。
                
            10. usb USB主机控制器和外设驱动（如U盘、摄像头）。
                
        2. 存储设备
            
            1. ata ATA/SATA控制器驱动（如硬盘、光驱）。
                
            2. block 块设备驱动框架（如磁盘、RAID）。
                
            3. md 多磁盘驱动（软件RAID实现）。
                
            4. mmc SD/MMC卡控制器驱动（如手机存储卡）。
                
            5. nvme NVMe协议驱动，支持高速固态硬盘（SSD）。
                
            6. scsi SCSI/SAS控制器驱动，用于企业级存储设备。
                
            7. ufs 通用闪存存储（UFS）驱动，常见于移动设备。
                
        3. 网络与通信
            
            1. bluetooth 蓝牙协议栈及硬件驱动。
                
            2. net 网络设备驱动（以太网卡、Wi-Fi、虚拟网卡）。
                
            3. nfc 近场通信（NFC）芯片驱动。
                
            4. w1 单总线（1-Wire）协议驱动。
                
            5. isdn 综合业务数字网（ISDN）设备驱动。
                
        4. 多媒体与显示
            
            6. dma-buf DMA缓冲区共享框架，用于GPU和显示驱动。
                
            7. gpu 图形处理器（GPU）驱动（如DRM框架）。
                
            8. media 多媒体设备驱动（摄像头、视频采集卡、TV调谐器）。
                
            9. sound 音频设备驱动（声卡、DSP、编解码器）。
                
            10. video 显示控制器和帧缓冲驱动（如LCD、HDMI）。
                
        5. 输入与交互
            
            1. hid 人机接口设备（HID）驱动（如键盘、鼠标、游戏手柄）。
                
            2. input 输入子系统驱动（触摸屏、传感器、按键）。
                
            3. leds LED灯控制驱动（如电源指示灯）。
                
            4. rtc 实时时钟（RTC）驱动，用于系统时间同步。
                
        6. 电源与能耗
            
            1. power 电源管理框架（休眠、唤醒、电池管理）。
                
            2. thermal 温度控制驱动（散热风扇、温控传感器）。
                
            3. regulator 电压/电流调节器驱动（如CPU供电管理）。
                
        7. 虚拟化与容器
            
            1. virt 虚拟化驱动（如KVM、Xen的前端/后端设备）。
                
            2. vfio 用户态设备直通框架（用于虚拟化环境）。
                
            3. vhost 用户态网络和存储加速驱动（如vhost-net）。
                
        8. 特殊用途设备
            
            4. char 字符设备驱动（随机数生成器、内存设备）。
                
            5. misc 杂项设备驱动（如FPGA配置、硬件监控）。
                
            6. tty 串口和终端驱动（如UART、虚拟控制台）。
                
            7. watchdog 硬件看门狗驱动，防止系统死锁。
                
        9. 平台与架构相关
            
            1. of 设备树（Device Tree）支持代码。
                
            2. platform 平台设备驱动（如SoC集成外设）。
                
            3. soc 芯片系统（SoC）特定驱动（如时钟、电源管理）。
                
        10. 调试与测试
            
            4. edac 错误检测与纠正（EDAC）驱动（内存错误监控）。
                
            5. hwtracing 硬件追踪驱动（如Intel PT、CoreSight）。
                
            6. perf 性能监控工具支持代码。
                
            7. staging 暂存区驱动（尚未稳定的实验性代码）。
                
        11. 其他关键目录
            
            1. android Android特有驱动（如Binder IPC、ION内存分配器）。
                
            2. firmware 固件加载接口（如更新设备固件）。
                
            3. memory 内存控制器驱动（如DDR时序配置）。
                
            4. target SCSI目标模式驱动（用于存储虚拟化）。
                
    4. fs 文件系统实现（Ext4/XFS/Btrfs等）和VFS虚拟文件系统。
        
    5. net 网络协议栈（TCP/IP、套接字、网卡驱动）。
        
    6. mm 内存管理（物理内存分配、虚拟内存、NUMA支持）。
        
    7. kernel 核心子系统（进程调度、信号处理、定时器）。
        
2. 构建与配置系统
    
    8. build.config. 多平台编译配置：
        
        1. aarch64/arm/x86_64：不同架构的通用配置
            
        2. amlogic/rockpi4：芯片厂商专用配置
            
        3. gki：通用内核镜像（Android专用）
            
        4. kasan/khwasan：内存调试工具配置
            
    9. BUILD.bazel Bazel构建系统的入口文件，替代传统Makefile，声明全局构建规则。
        
    10. Kconfig 内核配置菜单的主定义文件，控制功能开关（如启用特定驱动）。
        
    11. Makefile 顶层Makefile，定义内核编译流程和模块依赖关系。
        
    12. scripts 构建和开发辅助脚本（如代码检查、配置生成）。
        
3. 安全与加密
    
    1. security 安全框架（SELinux、AppArmor）和权限控制。
        
    2. crypto 加密算法实现（AES/SHA）和硬件加速接口。
        
    3. certs 内核模块签名证书和黑名单管理。
        
4. 硬件平台扩展
    
    4. common_drivers 项目定制化的通用驱动（如Amlogic芯片的共享模块）。
        
        1. 构建系统配置
            
            1. amlogic.bzl Bazel构建规则的主文件，定义Amlogic平台驱动的编译目标（如哪些驱动编译为模块或内置）。
                
            2. amlogic_utils.bzl 构建工具函数库，提供自动化依赖分析、模块生成等Bazel辅助规则。
                
            3. build.config.amlogic Amlogic平台的构建配置：
                
            4. build.config.amlogic: ARM64架构通用配置
                
            5. build.config.amlogic32: 旧版ARM32架构配置
                
            6. build.config.amlogic.bazel: Bazel专用构建规则补充配置。
                
            7. modules.bzl 定义需要编译的内核模块列表（如视频解码驱动、GPIO驱动）。
                
            8. header_include.mk Makefile头文件包含规则，指定驱动头文件的搜索路径。
                
        2. 自动化脚本
            
            1. amlogic_utils.sh Shell工具脚本，用于：
                
            2. 环境变量配置（如交叉编译工具链路径）
                
            3. 自动打补丁（调用auto_patch/中的补丁）
                
            4. 清理临时文件。
                
            5. mk.sh 主构建脚本，执行以下操作：
                
            6. 选择平台（如amlogic_s905x3）
                
            7. 调用Bazel或Make编译内核和驱动
                
            8. 生成固件镜像和设备树文件（.dtb）。
                
            9. auto_patch/ 存放内核源码补丁文件（.patch），用于修复上游内核对Amlogic芯片的兼容性问题。
                
        3. 驱动与硬件支持
            
            1. drivers/ Amlogic定制化设备驱动代码，例如：
                
            2. drivers/amlogic/media/: 视频编解码硬件加速（H.265/VP9）
                
            3. drivers/amlogic/gpio/: 芯片专用GPIO控制器驱动
                
            4. drivers/amlogic/sound/: 音频编解码器驱动。
                
            5. include/ 驱动头文件和硬件寄存器定义，例如：
                
            6. include/linux/amlogic/registers.h: 定义Amlogic芯片寄存器地址
                
            7. include/uapi/linux/vdec.h: 用户态视频解码控制接口。
                
            8. sound/ 音频子系统扩展，支持Amlogic HDMI音频输出或数字音频接口（I2S/TDM）。
                
        4. 平台适配与项目配置
            
            1. arch/ Amlogc芯片的架构扩展代码：
                
            2. 启动代码（arch/arm/mach-meson）
                
            3. 电源管理（PM）和时钟控制。
                
            4. project/ 具体硬件项目的配置：
                
            5. 设备树文件（dts/amlogic/meson-g12a-u200.dts）
                
            6. 硬件配置文件（如Wi-Fi模块的校准数据）。
                
            7. Kconfig.ext 扩展内核配置选项，定义Amlogic特有功能的开关（如CONFIG_AMLOGIC_DEBUG_TOOLS）。
                
        5. 其他工具与辅助内容
            
            1. modules_rename.txt 模块重命名规则，解决内核符号冲突问题（如自定义驱动与上游驱动的重名问题）。
                
            2. samples/ Amlogc驱动的示例代码，演示如何调用驱动API或开发新功能。
                
            3. scripts/ 开发工具脚本：
                
            4. 性能分析工具
                
            5. 固件签名脚本
                
            6. 调试日志解析工具。
                
    5. sound 音频子系统（声卡驱动、编解码器支持）。
        
    6. gpu 图形驱动（DRM框架、显示控制器支持）。
        
5. 调试与工具
    
    1. tools 内核调试和性能分析工具（如perf、tracing工具）。
        
    2. Documentation 内核文档（配置指南、API说明、调试方法）。
        
    3. samples 示例代码（演示内核模块开发、BPF程序编写）。
        
6. 虚拟化与容器
    
    4. virt 虚拟化支持（KVM、Xen前端驱动）。
        
    5. io_uring 高性能异步I/O框架，用于容器和数据库场景。
        
7. 其他关键目录
    
    1. init 内核启动初始化代码（start_kernel入口）。
        
    2. ipc 进程间通信（消息队列、共享内存）。
        
    3. lib 内核通用库（字符串操作、CRC校验）。
        
    4. usr 用户空间工具（如initramfs生成）。