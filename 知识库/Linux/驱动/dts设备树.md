[[平台驱动模型]]
[[字符驱动模型]]

/proc/device-tree -> /sys/firmware/devicetree/base  链接 文件 指向设备树


## 5、DTS设备树

### 5.1 设备树简介

在内核源码中，存在大量对板级细节信息描述的代码。这些代码充斥在/arch/arm/plat-xxx和/arch/arm/mach-xxx目录，对内核而言这些platform设备、resource、i2c_board_info、spi_board_info以及各种硬件的platform_data绝大多数纯属垃圾冗余代码。为了解决这一问题，ARM内核版本3.x之后引入了原先在Power PC等其他体系架构已经使用的Flattened Device Tree（设备树）。

设备树的定义：一种描述硬件资源的数据结构，它通过bootloader将硬件资源传给内核，使得内核和硬件资源描述相对独立。

设备树由一系列被命名的节点（Node）和属性（Property）组成，而节点本身可包含子节点。所谓属性，其实就是成对出现的名称和值。
设备树可以描述的信息：
- CPU的数量和类别
- 内存基地址和大小
- 总线和桥
- 外设连接
- 中断控制器和中断使用情况
- GPIO控制器和GPIO使用情况
- Clock控制器和Clock使用情况。

**注意**：设备树对于可热插拔的设备不进行具体描述，它只描述用于控制该热插拔设备的控制器。

设备树的主要优势：对于同一SOC的不同主板，只需更换设备树文件.dtb即可实现不同主板的无差异支持，而无需更换内核文件。
（ 注意：要使得3.x之后的内核支持使用设备树，除了内核编译时需要打开相对应的选项外，bootloader也需要支持将设备树的数据结构传给内核。）

### 5.2 设备树的组成和结构
设备树主要包含DTC（device tree compiler），DTS（device tree source）和DTB（device tree blob）。其对应关系如下图所示：
![[Pasted image 20220829213832.png]]

#### DTS和DTSI（源文件）

.dts文件是一种ASCII文本对 Device Tree 的描述，放置在内核的 `/arch/arm/boot/dts` 目录。一般而言，一个.dts文件对应一个 ARM 的 machine。

由于一个SOC可能有多个不同的电路板（ .dts文件为板级定义， .dtsi文件为SoC级定义），而每个电路板拥有一个 .dts。这些dts势必会存在许多共同部分，为了减少代码的冗余，设备树将这些共同部分提炼保存在.dtsi文件中，供不同的dts共同使用。.dtsi的使用方法，类似于C语言的头文件，在dts文件中需要进行include .dtsi文件。当然，dtsi本身也支持include 另一个dtsi文件。

.dts的结构：
```dts
 /{
	node1{
		a - string - property = "A string";
		a - string - list - property = "first string", 
		"second string";
		a - byte - data - property =[0x01 0x23 0x34 0x56];
		child - node1
			{
			first - child - property;
			second - child - property = < 1 >;
			a - string - property = "Hello, world";
			};
		child - node2
			{
			};
		};
	node2{
		an - empty - property;
		a - cell - property = < 1 2 3 4 >;			/* each number (cell) is a uint32 */
		child - node1
			{
			};
		};
	};
```
- 1个root节点: “/”；
- root节点下面含一系列子节点，为node1和node2；
- 节点node1下又含有一系列子节点，本例中为child-node1和child-node2；

各节点都有一系列属性。这些属性可能为空，如an-empty-property；可能为字符串如a-string-property；可能为字符串数组，如a-string-list-property；可能为Cells（由u32整数组成），如second-child-property；可能为二进制数，如a-byte-data-property。

#### DTC（编译工具）

DTC为编译工具，dtc编译器可以把dts文件编译成为dtb，也可把dtb编译成为dts文件。在3.x内核版本中，DTC的源码位于内核的scripts/dtc目录，内核选中CONFIG_OF，编译内核的时候，主机可执行程序DTC就会被编译出来。 即scripts/dtc/Makefile中hostprogs-y := dtc always := $(hostprogs-y) 这一hostprogs的编译目标。

在Linux内核的arch/arm/boot/dts/Makefile中，描述了当某种SoC被选中后，哪些.dtb文件会被编译出来，如与VEXPRESS对应的.dtb包括：
```makefile
dtb-$(CONfiG_ARCH_VEXPRESS) += vexpress-v2p-ca5s.dtb \
vexpress-v2p-ca9.dtb \
vexpress-v2p-ca15-tc1.dtb \
vexpress-v2p-ca15_a7.dtb \
xenvm-4.2.dtb
```

在Linux下，我们可以单独编译设备树文件。当我们在Linux内核下运行make dtbs时，若我们之前选择了ARCH_VEXPRESS，上述.dtb都会由对应的.dts编译出来，因为arch/arm/Makefile中含有一个.dtbs编译目标项目。

DTC除了可以编译.dts文件以外，其实也可以“反汇编”.dtb文件为.dts文件，其指令格式为：
```shell
./scripts/dtc/dtc -I dtb -O dts -o xxx.dts arch/arm/boot/dts/xxx.dtb
```

#### DTB（二进制文件）

DTC编译.dts生成的二进制文件（.dtb），bootloader在引导内核时，会预先读取.dtb到内存，进而由内核解析。

Dts编译生成dtb
`./dtc -I dts -O dtb -o B_dtb.dtb A_dts.dts \\把A_dts.dts编译生成B_dtb.dtb`

Dtb编译生成dts
`./dtc -I dtb -O dts -o A_dts.dts A_dtb.dtb \\把A_dtb.dtb反编译生成为A_dts.dts`

#### Binding(绑定）

对于设备树中的节点和属性具体是如何来描述设备的硬件细节的，一般需要文档来进行讲解，文档的后缀名一般为.txt。在这个.txt文件中，需要描述对应节点的兼容性、必需的属性和可选的属性。

这些文档位于内核的Documentation/devicetree/bindings目录下，其下又分为很多子目录。譬如，Documentation/devicetree/bindings/i2c/i2c-xiic.txt描述了Xilinx的I2C控制器，其内容如下：
```
Xilinx IIC controller:
Required properties:
- compatible : Must be "xlnx,xps-iic-2.00.a"
- reg : IIC register location and length
- interrupts : IIC controller unterrupt
- #address-cells = <1>
- #size-cells = <0>
Optional properties:
- Child nodes conforming to i2c bus binding
Example:
axi_iic_0: i2c@40800000 {
compatible = "xlnx,xps-iic-2.00.a";
interrupts = < 1 2 >;
reg = < 0x40800000 0x10000 >;
#size-cells = <0>;
#address-cells = <1>;
};
```

设备树绑定文档的主要内容包括：
- 关于该模块最基本的描述。
- 必需属性（Required Properties）的描述。
- 可选属性（Optional Properties）的描述。
- 一个实例：
    Linux内核下的scripts/checkpatch.pl会运行一个检查，如果有人在设备树中新添加了compatible字符串，而没有添加相应的文档进行解释，checkpatch程序会报出警告：UNDOCUMENTED_DT_STRINGDTcompatible string xxx appears un-documented，因此程序员要养成及时写DT Binding文档的习惯。

#### Bootloader

Uboot设备从v1.1.3开始支持设备树，其对ARM的支持则是和ARM内核支持设备树同期完成。

为了使能设备树，需要在编译Uboot的时候在config文件中加入：
`#define CONfiG_OF_LIBFDT`

在Uboot中，可以从NAND、SD或者TFTP等任意介质中将.dtb读入内存，假设.dtb放入的内存地址为0x71000000，之后可在Uboot中运行fdt addr命令设置.dtb的地址，如：
`UBoot> fdt addr 0x71000000`

fdt的其他命令就变得可以使用，如 fdt resize、fdt print 等。
对于ARM来讲，可以通过 bootz kernel_addr initrd_address dtb_address 的命令来启动内核，即dtb_address 作为 bootz 或者 bootm 的最后一个参数，第一个参数为内核映像的地址，第二个参数为initrd的地址，若不存在initrd，可以用 “-” 符号代替。

### 5.3 设备树结构解析

#### 根节点兼容性

在.dts文件中，根节点"/"的兼容属性compatible=“acme，coyotes-revenge”；定义了整个系统（设备级别）的名称，它的组织形式为：`<manufacturer>`，`<model>`。

Linux内核通过根节点"/"的兼容属性即可判断它启动的是什么设备。这个顶层设备兼容属性一般包括两个或者两个以上的兼容性字符串，首个兼容性字符串是板子级别的名字，后面一个兼容性是芯片级别（或者芯片系列级别）的名字。比如：
`compatible = "insignal,origen","samsung,exynos4210","samsung,exynos4";`
- 第一个字符串是板子名字（很特定）;
- 第2个字符串是芯片名字（比较特定）；
- 第3个字段是芯片系列的名字（比较通用）。

在Linux 2.6内核中，ARM Linux针对不同的电路板会建立由 MACHINE_START 和 MACHINE_END 包围起来的针对这个设备的一系列回调函数：
```c
MACHINE_START(VEXPRESS, "ARM-Versatile Express")
  .atag_offset = 0x100, 
  .smp = smp_ops(vexpress_smp_ops), 
  .map_io = v2m_map_io, 
  .init_early = v2m_init_early, 
  .init_irq = v2m_init_irq, 
  .timer = &v2m_timer, 
  .handle_irq = gic_handle_irq, 
  .init_machine = v2m_init, 
  .restart = vexpress_restart, 
MACHINE_END
```

这些不同的设备会有不同的MACHINE ID，Uboot在启动Linux内核时会将MACHINE ID存放在r1寄存器，Linux启动时会匹配Bootloader传递的MACHINE ID和MACHINE_START声明的MACHINE ID，然后执行相应设备的一系列初始化函数。

ARM Linux 3.x在引入设备树之后，MACHINE_START变更为DT_MACHINE_START，其中含有一个.dt_compat成员，用于表明相关的设备与.dts中根节点的兼容属性兼容关系。如果Bootloader传递给内核的设备树中根节点的兼容属性出现在某设备的.dt_compat表中，相关的设备就与对应的兼容匹配，从而引发这一设备的一系列初始化函数被执行。
一个典型的DT_MACHINE：
```c
static const char * const v2m_dt_match[] __initconst = {
 "arm,vexpress", "xen,xenvm",NULL };
 
DT_MACHINE_START(VEXPRESS_DT,"ARM-Versatile Express")
 .dt_compat = v2m_dt_match,
 .smp = smp_ops(vexpress_smp_ops),
   .map_io = v2m_dt_map_io,
 .init_early = v2m_dt_init_early,
 .init_irq = v2m_dt_init_irq,
 .timer = &v2m_dt_timer,
 .init_machine = v2m_dt_init,
 .handle_irq = gic_handle_irq,
 .restart = vexpress_restart,
MACHINE_END
```

Linux倡导针对多个 SoC、多个电路板的通用 DT 设备，即一个 DT 设备的 .dt_compat 包含多个电路板.dts文件的根节点兼容属性字符串。之后，如果这多个电路板的初始化序列不一样，可以通过`Int of_machine_is_compatible (const char*compat)` API判断具体的电路板是什么。在Linux内核中，常常使用如下API来判断根节点的兼容性：
`int of_machine_is_compatible(const char *compat);`
此API判断目前运行的板子或者SoC的兼容性，它匹配的是设备树根节点下的兼容属性。

#### 设备节点兼容性

在.dts文件的每个设备节点中，都有一个兼容属性，兼容属性用于驱动和设备的绑定。兼容属性是一个字符串的列表，列表中的第一个字符串表征了节点代表的确切设备，形式为`<manufacturer>，<model>`，其后的字符串表征可兼容的其他设备。可以说前面的是特指，后面的则涵盖更广的范围。
设备节点的兼容性和根节点的兼容性是类似的，都是“从具体到抽象”。
使用设备树后，驱动需要与.dts中描述的设备节点进行匹配，从而使驱动的probe（）函数执行。对于platform_driver而言，需要添加一个OF匹配表，如前文的.dts文件的 "acme，a1234-i2c-bus" 兼容 I2C 控制器节点的 OF 匹配表：
platform设备驱动中的of_match_table：
```c
static const struct of_device_id a1234_i2c_of_match[] =
{
	{.compatible = "acme,a1234-i2c-bus"},
	{ },
};
MODULE_DEVICE_TABLE(of, a1234_i2c_of_match);

static struct platform_driver i2c_a1234_driver =
{
	.driver = {
		.name = "a1234-i2c-bus", 
		.owner = THIS_MODULE, 
		.of_match_table = a1234_i2c_of_match, 
	},
	.probe = i2c_a1234_probe, 
	.remove = i2c_a1234_remove, 
};
module_platform_driver(i2c_a1234_driver);
```

- 对于I2C和SPI从设备而言，同样也可以通过of_match_table添加匹配的.dts中的相关节点的兼容属性。
- 对于I2C、SPI还有一点需要提醒的是，I2C和SPI外设驱动和设备树中设备节点的兼容属性还有一种弱式匹配方法，就是“别名”匹配。兼容属性的组织形式为，，别名其实就是去掉兼容属性中manufacturer前缀后的部分。

    一个驱动可以在of_match_table中兼容多个设备，在Linux内核中常常使用如下API来判断具体的设备是什么：
    `int of_device_is_compatible(const struct device_node *device,const char *compat);`
    此函数用于判断设备节点的兼容属性是否包含compat指定的字符串。这个API多用于一个驱动支持两个以上设备的时候。

- 当一个驱动支持两个或多个设备的时候，这些不同 .dts 文件中设备的兼容属性都会写入驱动 OF 匹配表。因此驱动可以通过 Bootloader 传递给内核设备树中的真正节点的兼容属性以确定究竟是哪一种设备，从而根据不同的设备类型进行不同的处理。
- 当一个驱动可以兼容多种设备的时候，除了 of_device_is_compatible ( ) 这种判断方法以外，还可以采用在驱动的 of_device_id 表中填充 .data 成员的形式。

#### 设备节点和label的命名

设备树的.dts文件中对于设备节点的命名，遵循的组织形式为 `<name>[@<unit-address>]`，`<>` 中的内容是必选项，`[ ]` 中的则为可选项。

name 是一个ASCII字符串，用于描述节点对应的设备类型；
如果一个节点描述的设备有地址，则应该给出@unit-address。多个相同类型设备节点的name可以一样，只要unit-address不同即可。

对于挂在内存空间的设备而言，@字符后跟的一般就是该设备在内存空间的基地址。
还可以给一个设备节点添加 label ，之后可以通过 &label 的形式访问这个 label ，这种引用是通过phandle (pointer handle) 进行的。

#### linux内核对硬件的描述方式

在以前的内核版本中，内核包含了对硬件的全部描述：
1. bootloader 会加载一个二进制的内核镜像，并执行它，比如 uImage 或者 zImage；
2. bootloader 会提供一些额外的信息，成为 ATAGS，它的地址会通过 r2 寄存器传给内核；
3. ATAGS 包含了内存大小和地址，kernel command line 等等；
4. bootloader 会告诉内核加载哪一款 board ，通过 r1 寄存器存放的 machine type integer；
5. U-Boot 的内核启动命令：bootm
6. Barebox 变量：bootm.image

现今的内核版本使用了Device Tree：
1. 内核不再包含对硬件的描述，它以二进制的形式单独存储在另外的位置：the device tree blob
2. bootloader 需要加载两个二进制文件：内核镜像和DTB
3. 内核镜像仍然是 uImage 或者 zImage；
4. DTB文件在 arch/arm/boot/dts 中，每一个 board 对应一个 dts 文件；
5. bootloader 通过 r2 寄存器来传递 DTB 地址，通过修改DTB可以修改内存信息，kernel command line，以及潜在的其它信息；
6. 不再有 machine type；
7. U-Boot 的内核启动命令：bootm -
8. Barebox 变量：bootm.image,bootm.oftree

#### GPIO  
对于GPIO控制器而言，其对应的设备节点需声明gpio-controller属性，并设置#gpio-cells的大小。譬如，对于兼容性为fsl，imx28-pinctrl的pinctrl驱动而言，其GPIO控制器的设备节点类似于：
```c
pinctrl @ 80018000
{
	compatible			= "fsl,imx28-pinctrl", "simple-bus";
	reg 				= < 0x80018000 2000 >;
    gpio0:gpio @ 0
		{
		compatible			= "fsl,imx28-gpio";
		interrupts			= < 127 >;
		gpio - controller;
               #gpio -cells = <2>;
		interrupt - controller;
               #interrupt -cells = <2>;
		};
    gpio1:gpio @ 1
		{
		compatible			= "fsl,imx28-gpio";
		interrupts			= < 126 >;
		gpio - controller;
               #gpio - cells = <2>;
        interrupt - controller;
               #interrupt -cells = <2>;
		};
	...
};
```
其中，#gpio- cells为2，第1个cell为GPIO号，第2个为GPIO的极性。为0的时候是高电平有效，为1的时候则是低电平有效。  
使用GPIO的设备则通过定义命名xxx-gpios属性来引用GPIO控制器的设备节点，如：
```c
sdhci@c8000400 {
status = "okay";
cd-gpios = <&gpio01 0>;
wp-gpios = <&gpio02 0>;
power-gpios = <&gpio03 0>;
bus-width = <4>;
};
```
而具体的设备驱动则通过类似如下的方法来获取GPIO：
```c
cd_gpio = of_get_named_gpio(np,"cd-gpios", 0);
wp_gpio = of_get_named_gpio(np,"wp-gpios", 0);
power_gpio = of_get_named_gpio(np,"power-gpios", 0);

//of_get_named_gpio（）这个API的原型如下：
static inline int of_get_named_gpio(struct device_node *np,
const char *propname, int index);
```

在.dts和设备驱动不关心GPIO名字的情况下，也可以直接通过of_get_gpio（）获取GPIO，此函数原型为：
```c
static inline int of_get_gpio(struct device_node *np, int index);
```

### 5.4 设备树引发的BSP和驱动变更

**有了设备树后，不再需要大量的板级信息，譬如过去经常在arch/arm/plat-xxx和arch/arm/mach-xxx中实施的一些事情**：

1. 注册platform_device，绑定resource，即内存、IRQ等板级信息
    有了设备树之后，形如：
    ```c
    static struct resource xxx_resources[] = {
    [0] = {
    .start = …,
    .end = …,
    .flags = IORESOURCE_MEM,
    },
    [1] = {
    .start = …,
    .end = …,
    .flags = IORESOURCE_IRQ,
    },
    };
    static struct platform_device xxx_device = {
    .name = "xxx",
    .id = -1,
    .dev = {
    .platform_data = &xxx_data,
    },
    .resource = xxx_resources,
    .num_resources = ARRAY_SIZE(xxx_resources),
    };
    ```
    之类的platform_device代码都不再需要，其中platform_device会由内核自动展开。而这些resource实际来源于.dts中设备节点的reg、interrupts属性。
    典型的，大多数总线都与“simple_bus”兼容，而在与SoC对应的设备的.init_machine成员函数中，调用of_platform_bus_probe（NULL，xxx_of_bus_ids，NULL）；即可自动展开所有的platform_device。

2. 注册i2c_board_info或spi_board_info，指定IRQ等板级信息
    形如：
    ```c
    static struct i2c_board_info __initdata afeb9260_i2c_devices[] = {
    {
    I2C_BOARD_INFO("tlv320aic23", 0x1a),
    }, {
    I2C_BOARD_INFO("fm3130", 0x68),
    }, {
    I2C_BOARD_INFO("24c64", 0x50),
    },
    };
    ```

    之类的i2c_board_info代码目前不再需要出现，现在只需要把tlv320aic23、fm3130、24c64这些设备节点填充作为相应的I2C控制器节点的子节点：
    ```c
    i2c@0x1a{
    compatible = "acme,a1234-i2c-bus";
    …
    rtc@1a {
    compatible = "maxim,tlv320aic23";
    reg = <1a>;
    interrupts = < 7 3 >;
    };
    };
    ```
    
    设备树中的I2C客户端会通过在I2C host驱动的probe（）函数中调用的of_i2c_register_devices（&i2c_dev->adapter）；被自动展开。  
    而spi_board_info与i2c的变化类似。

3. 多个针对不同电路板的设备，以及相关的回调函数
    在过去，ARM Linux针对不同的电路板会建立由MACHINE_START和MACHINE_END包围的设备，引入设备树之后，MACHINE_START变更为DT_MACHINE_START，其中含有一个.dt_compat成员，用于表明相关的设备与.dts中根节点的兼容属性的兼容关系。
    采用设备树后，我们可以对多个SoC和板子使用同一个DT_MACHINE和板文件，板子和板子之间的差异更多只是通过不同的.dts文件来体现。

4. 设备与驱动的匹配方式
    使用设备树后，驱动需要与在.dts中描述的设备节点进行匹配，从而使驱动的prob（）函数执行。新的驱动、设备的匹配变成了设备树节点的兼容属性和设备驱动中的OF匹配表的匹配。

5. 设备的平台数据属性化
    在Linux 2.6下，驱动习惯自定义platform_data，在arch/arm/mach-xxx注册platform_device、i2c_board_info、spi_board_info等的时候绑定platform_data，而后驱动通过标准API获取平台数据。
    在转移到设备树后，platform_data便不再喜欢放在arch/arm/mach-xxx中了，它需要从设备树的属性中获取。

### 5.5 常用的of API

除了前面接触到的 of_machine_is_compatible（）、of_device_is_compatible（）等常用函数以外，在Linux的BSP和驱动代码中，经常会使用到一些Linux中其他设备树的API，这些API通常被冠以 of_ 前缀，它们的实现代码位于内核的drivers/of 目录下：

#### 寻找节点

```c
struct device_node *of_find_compatible_node(struct device_node *from,const char *type, const char *compatible);
```
根据兼容属性，获得设备节点。遍历设备树中的设备节点，看看哪个节点的类型、兼容属性与本函数的输入参数匹配，在大多数情况下，from、type为NULL，则表示遍历了所有节点。

#### 读取属性

```c
int of_property_read_u8_array(const struct device_node *np,
const char *propname, u8 *out_values, size_t sz);
int of_property_read_u16_array(const struct device_node *np,
const char *propname, u16 *out_values, size_t sz);
int of_property_read_u32_array(const struct device_node *np,
const char *propname, u32 *out_values, size_t sz);
int of_property_read_u64(const struct device_node *np,
                              const char*propname, u64 *out_value);

```
读取设备节点np的属性名，为propname，属性类型为8、16、32、64位整型数组。对于32位处理器来讲，最常用的是of_property_read_u32_array（）。

除了整型属性外，字符串属性也比较常用，其对应的API包括：
```c
int of_property_read_string(struct device_node *np,
 const char*propname,const char **out_string);
int of_property_read_string_index(struct device_node *np, 
const char*propname,int index, const char **output);
```
前者读取字符串属性，后者读取字符串数组属性中的第index个字符串。

除整型、字符串以外的最常用属性类型就是布尔型，其对应的API很简单：
```c
static inline bool of_property_read_bool(const struct device_node *np, const char *propname);
```
如果设备节点np含有propname属性，则返回true，否则返回false。一般用于检查空属性是否存在。

#### 内存映射
```c
void __iomem *of_iomap(struct device_node *node, int index);
```
上述API可以直接通过设备节点进行设备内存区间的ioremap（），index是内存段的索引。若设备节点的reg属性有多段，可通过index标示要ioremap（）的是哪一段，在只有1段的情况，index为0。采用设备树后，一些设备驱动通过of_iomap（）而不再通过传统的ioremap（）进行映射，当然，传统的ioremap（）的用户也不少。

```c
int of_address_to_resource(struct device_node *dev, int index,struct resource *r);
```
上述API通过设备节点获取与它对应的内存资源的resource结构体。其本质是分析reg属性以获取内存基地址、大小等信息并填充到struct resource\*r参数指向的结构体中。

#### 解析中断
```c
unsigned int irq_of_parse_and_map(struct device_node *dev, int index);
```
通过设备树获得设备的中断号，实际上是从.dts中的interrupts属性里解析出中断号。若设备使用了多个中断，index指定中断的索引号。

#### 获取与节点对应的platform_device
```c
struct platform_device *of_find_device_by_node(struct device_node *np);
```
在可以拿到device_node的情况下，如果想反向获取对应的platform_device，可使用上述API。当然，在已知platform_device的情况下，想获取device_node则易如反掌，例如：
```c
static int sirfsoc_dma_probe(struct platform_device *op)
{
struct device_node *dn = op->dev.of_node;
…
}
```
