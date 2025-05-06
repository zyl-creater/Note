## 6、GPIO子系统

### 6.1 层次结构

GPIO作为一个常用的信息输入输出手段经常被用在各种场合

GPIO像I2C一样，要使用某个引脚，需要先把引脚配置为GPIO功能，这要使用Pinctrl子系统，只需要在设备树里指定就可以了。
在驱动代码上不需要做任何事情。GPIO本身需要确定引脚，这也需要在设备树里指定。
设备树节点会被内核转换为platform_device。
对应的，驱动代码中要注册一个platform_driver，在probe函数中：获得引脚、注测file_operations。
在file_operation中：设置方向、读写值。
![[Pasted image 20220811182702.png]]

**GPIO子系统** 相对于 **pin control subsystem** 来说是更加上层的子系统，它将 **引脚** 配置为 **GPIO** 并且控制其输入输出，所以从这方面来看，**GPIO** 和 **引脚** 在系统软件层面并不是一个概念

中间层有gpio lib
![[Pasted image 20220830154303.png]]

#### GPIOLIB向上提供的接口
|    descriptor-based       |    legacy                |        说明       |
| ---------------------- | --------------------- | ---------- |
| gpiod_get              | gpio_request          | 获得GPIO   |
| gpiod_get_index        |                       |            |
| gpiod_get_array        | gpio_request_array    |            |
| devm_gpiod_get         |                       |            |
| devm_gpiod_get_index   |                       |            |
| devm_gpiod_get_array   |                       |            |
| gpiod_direction_input  | gpio_direction_input  | 设置方向   |
| gpiod_direction_output | gpio_direction_output |            |
| gpiod_get_value        | gpio_get_value        | 读值、写值 |
| gpiod_set_value        | gpio_set_value        |            |
| gpio_free              | gpio_free             | 释放GPIO   |
| gpio_put               | gpio_free_array       |            |
| gpiod_put_array        |                       |            |
| devm_gpiod_put         |                       |            |
| devm_gpiod_put_array   |                       |            |

#### GPIOLIB向下提供的接口
```c
//用来注册gpio_chip
int gpiochip_add_data(struct gpio_chip *chip, void *data)
```

### 6.2 三个核心数据结构

#### gpio_device
每个 `GPIO Controller` 用一个 `gpio_device` 来表示：
-   每个gpio引脚用一个 `gpio_desc` 来表示
-   gpio引脚的函数，都放在 `gpio_chip` 里
![[Pasted image 20220830155646.png]]

![[Pasted image 20220830155735.png]]

一个GPIO Controller用 `gpio_device` 来描述，每个其中每个引脚，有用 `gpio_desc` 来描述。
```c
//drivers/gpio/gpiolib.h
struct gpio_device {
	int			id;					//它是系统中第几个gpio controller
	struct device		dev;
	struct cdev		chrdev;
	struct device		*mockdev;
	struct module		*owner;
	struct gpio_chip	*chip;			//含有各类操作函数
	struct gpio_desc	*descs;		//用来描述引脚，每个引脚对应一个gpio_desc
	int			base;		//这些GPIO的号码基值
	u16			ngpio;		//这些GPIO Controller支持多少个GPIO
	char			*label;		//标签、名字
	void			*data;
	struct list_head        list;

#ifdef CONFIG_PINCTRL
	/*
	 * If CONFIG_PINCTRL is enabled, then gpio controllers can optionally
	 * describe the actual pin range which they serve in an SoC. This
	 * information would be used by pinctrl subsystem to configure
	 * corresponding pins for gpio usage.
	 */
	struct list_head pin_ranges;
#endif
};
```

#### gpio_chip
编写驱动时需要创建gpio_chip
-   控制引脚的函数
-   中断相关的函数
-   引脚信息：支持多少引脚？各引脚名字？
```c
struct gpio_chip {
	const char		*label;
	struct gpio_device	*gpiodev;
	struct device		*parent;
	struct module		*owner;

//相关函数
	int			(*request)(struct gpio_chip *chip,
						unsigned offset);
	void			(*free)(struct gpio_chip *chip,
						unsigned offset);
	int			(*get_direction)(struct gpio_chip *chip,
						unsigned offset);
	int			(*direction_input)(struct gpio_chip *chip,
						unsigned offset);
	int			(*direction_output)(struct gpio_chip *chip,
						unsigned offset, int value);
	int			(*get)(struct gpio_chip *chip,
						unsigned offset);
	void			(*set)(struct gpio_chip *chip,
						unsigned offset, int value);
	void			(*set_multiple)(struct gpio_chip *chip,
						unsigned long *mask,
						unsigned long *bits);
	int			(*set_config)(struct gpio_chip *chip,
					      unsigned offset,
					      unsigned long config);
	int			(*to_irq)(struct gpio_chip *chip,
						unsigned offset);

	void			(*dbg_show)(struct seq_file *s,
						struct gpio_chip *chip);
	int			base;		//GPIO Controller中引脚的号码基值
	u16			ngpio;		//个数
	const char		*const *names; //每个引脚的名字
	bool			can_sleep;

	//....
};
```

#### gpio_desc
使用GPIO子系统时，先获得某个引脚对应的gpio_desc  
gpio_device表示一个GPIO Controller，里面支持多个GPIO  
在gpio_device中有一个`gpio_desc`数组，每一个引脚有一项`gpio_desc`
```c
struct gpio_desc {
	struct gpio_device	*gdev;		//属于哪个GPIO Controller
	unsigned long		flags;
/* flag symbols are bit numbers */
#define FLAG_REQUESTED	0
#define FLAG_IS_OUT	1
#define FLAG_EXPORT	2	/* protected by sysfs_lock */
#define FLAG_SYSFS	3	/* exported via /sys/class/gpio/control */
#define FLAG_ACTIVE_LOW	6	/* value has active low */
#define FLAG_OPEN_DRAIN	7	/* Gpio is open drain type */
#define FLAG_OPEN_SOURCE 8	/* Gpio is open source type */
#define FLAG_USED_AS_IRQ 9	/* GPIO is connected to an IRQ */
#define FLAG_IS_HOGGED	11	/* GPIO is hogged */
#define FLAG_SLEEP_MAY_LOOSE_VALUE 12	/* GPIO may loose value in sleep */

	/* Connection label */
	const char		*label;		//一般等于gpio_chip的label
	/* Name of the GPIO */
	const char		*name;			//引脚名
};
```


### 6.3 内核中gpio的使用

1 测试gpio端口是否合法 int gpio_is_valid(int number); 

2 申请某个gpio端口当然在申请之前需要显示的配置该gpio端口的pinmux
       int gpio_request(unsigned gpio, const char \*label)

3 标记gpio的使用方向包括输入还是输出
       *成功返回零失败返回负的错误值*
       int gpio_direction_input(unsigned gpio); 
       int gpio_direction_output(unsigned gpio, int value); 

4 获得gpio引脚的值和设置gpio引脚的值(对于输出)
        int gpio_get_value(unsigned gpio);
        void gpio_set_value(unsigned gpio, int value); 

5 gpio当作中断口使用
        int gpio_to_irq(unsigned gpio); 
        返回的值即中断编号可以传给request_irq()和free_irq()
        内核通过调用该函数将gpio端口转换为中断，在用户空间也有类似方法

6 导出gpio端口到用户空间
        int gpio_export(unsigned gpio, bool direction_may_change); 
        内核可以对已经被gpio_request()申请的gpio端口的导出进行明确的管理，参数direction_may_change表示用户程序是否允许修改gpio的方向，假如可以则参数direction_may_change为真
        /* 撤销GPIO的导出 \*/ 
        void gpio_unexport(); 

### 6.4 用户空间gpio的调用 

用户空间访问gpio，即通过sysfs接口访问gpio，下面是/sys/class/gpio目录下的三种文件： 
        --export/unexport文件
        --gpioN指代具体的gpio引脚
        --gpio_chipN指代gpio控制器
        必须知道以上接口没有标准device文件和它们的链接。 

#### export/unexport文件接口：
/sys/class/gpio/export，该接口只能写不能读

用户程序通过写入gpio的编号来向内核申请将某个gpio的控制权导出到用户空间当然前提是没有内核代码申请这个gpio端口

比如  `echo 19 > export`

上述操作会为19号gpio创建一个节点gpio19，此时 `/sys/class/gpio` 目录下边生成一个 gpio19的目录

/sys/class/gpio/unexport 和导出的效果相反。 

比如 `echo 19 > unexport`

上述操作将会移除gpio19这个节点。 

#### /sys/class/gpio/gpioN
指代某个具体的gpio端口,里边有如下属性文件

direction 表示gpio端口的方向，读取结果是in或out。该文件也可以写，写入out 时该gpio设为输出同时电平默认为低。写入low或high则不仅可以设置为输出 还可以设置输出的电平。当然如果内核不支持或者内核代码不愿意，将不会存在这个属性,比如内核调用了gpio_export(N,0)就表示内核不愿意修改gpio端口方向属性 

value         表示gpio引脚的电平,0(低电平)1（高电平）,如果gpio被配置为输出，这个值是可写 
                  的，任何非零的值都将输出高电平, 如果某个引脚能并且已经被配置为中断，则可以调用poll(2)函数监听该中断，中断触发后poll(2)函数就会返回。

edge         表示中断的触发方式，edge文件有如下四个值："none", "rising", "falling"，"both".
                none表示引脚为输入，不是中断引脚
                rising表示引脚为中断输入，上升沿触发
                falling表示引脚为中断输入，下降沿触发
                both表示引脚为中断输入，边沿触发


/sys/kernel/debug/pinctrl/pinctrl@0xfe004000