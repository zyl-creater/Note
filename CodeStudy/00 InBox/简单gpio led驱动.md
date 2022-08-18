GPIO作为一个常用的信息输入输出手段经常被用在各种场合

GPIO像I2C一样，要使用某个引脚，需要先把引脚配置为GPIO功能，这要使用Pinctrl子系统，只需要在设备树里指定就可以了。
在驱动代码上不需要做任何事情。GPIO本身需要确定引脚，这也需要在设备树里指定。
设备树节点会被内核转换为platform_device。
对应的，驱动代码中要注册一个platform_driver，在probe函数中：获得引脚、注测file_operations。
在file_operation中：设置方向、读写值。
![[Pasted image 20220811182702.png]]

**GPIO子系统** 相对于 **pin control subsystem** 来说是更加上层的子系统，它将 **引脚** 配置为 **GPIO** 并且控制其输入输出，所以从这方面来看，**GPIO** 和 **引脚** 在系统软件层面并不是一个概念


## 内核中gpio的使用
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

## 三 用户空间gpio的调用 

用户空间访问gpio，即通过sysfs接口访问gpio，下面是/sys/class/gpio目录下的三种文件： 
        --export/unexport文件
        --gpioN指代具体的gpio引脚
        --gpio_chipN指代gpio控制器
        必须知道以上接口没有标准device文件和它们的链接。 

###  (1) export/unexport文件接口：
/sys/class/gpio/export，该接口只能写不能读

用户程序通过写入gpio的编号来向内核申请将某个gpio的控制权导出到用户空间当然前提是没有内核代码申请这个gpio端口

比如  echo 19 > export 

上述操作会为19号gpio创建一个节点gpio19，此时/sys/class/gpio目录下边生成一个gpio19的目录

/sys/class/gpio/unexport和导出的效果相反。 

比如 echo 19 > unexport

上述操作将会移除gpio19这个节点。 

###  (2) /sys/class/gpio/gpioN

   指代某个具体的gpio端口,里边有如下属性文件

 direction 表示gpio端口的方向，读取结果是in或out。该文件也可以写，写入out 时该gpio设为输出同时电平默认为低。写入low或high则不仅可以设置为输出 还可以设置输出的电平。当然如果内核不支持或者内核代码不愿意，将不会存在这个属性,比如内核调用了gpio_export(N,0)就表示内核不愿意修改gpio端口方向属性 

value      表示gpio引脚的电平,0(低电平)1（高电平）,如果gpio被配置为输出，这个值是可写的，任何非零的值都将输出高电平, 如果某个引脚能并且已经被配置为中断，则可以调用poll(2)函数监听该中断，中断触发后poll(2)函数就会返回。

edge      表示中断的触发方式，edge文件有如下四个值："none", "rising", "falling"，"both".
                none表示引脚为输入，不是中断引脚
                rising表示引脚为中断输入，上升沿触发
                falling表示引脚为中断输入，下降沿触发
                both表示引脚为中断输入，边沿触发

```c
struct zyl_led_info
{
    char num;  //led编号
    int gpio;
};

struct zyl_led_platform_data
{
    struct zyl_led_info *leds;
    int nleds;
};

struct zyl_led_info zyl_leds[] = {
    [0] = {
        .num = 1,
        .gpio = 465,
    },
    [1] = {
        .num = 2,
        .gpio = 458,
    },
    [2] = {
        .num = 3,
        .gpio = 453,
    },
};

  

struct zyl_led_platform_data zyl_led_date = {
    .leds = zyl_leds,
    .nleds = ARRAY_SIZE(zyl_leds),
};
```

long led_ioctl中
```c
    switch (ret)
    {
    case 0:
        gpio_set_value(pdata->leds[0].gpio,0);
        break;
    case 1:
        gpio_set_value(pdata->leds[0].gpio,1);
        break;
    case 2:
        gpio_set_value(pdata->leds[1].gpio,0);
        break;
    case 3:
        gpio_set_value(pdata->leds[1].gpio,1);
        break;
    case 4:
        gpio_set_value(pdata->leds[2].gpio,0);
        break;
    case 5:
        gpio_set_value(pdata->leds[2].gpio,1);
        break;
    default:
        break;
    }
    return 0;
}
```

```c
static struct platform_device zyl_led_device = {
    .name = "zyl_led",
    .id = 1,
    .dev =
    {
        .platform_data = &zyl_led_data,
        .release = platform_led_release,
    },
};

static int zyl_led_probe(struct platform_device *dev)
{
    struct zyl_led_platform_data *pdata = dev->dev.platform_data;
    int ret = 0;
    int i;
    dev_t devno;
    printk("%s:Success Cin zyl_led_probe \n",__func__);
    for ( i = 0; i < pdata->nleds; i++)
    {
        gpio_direction_output(pdata->leds[i].gpio,0);
        udelay(30000);
        gpio_set_value(pdata->leds[i].gpio,1);
        udelay(30000);
    }
    ...
}
```

