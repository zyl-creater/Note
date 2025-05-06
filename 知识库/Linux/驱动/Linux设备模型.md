除了驱动模型还有
[[字符驱动模型]]
[[平台驱动模型]]
[[简单gpio led驱动]]

## 3、设备驱动模型
### 3.1 linux设备驱动模型简介
#### 什么是设备驱动模型
(1)类[[class]]、总线[[bus]]、设备[[device]]、驱动[[driver]]（内核代码中有4个结构体）
(2)[[kobject]]和对象生命周期（自动管理生命周期）
(3)**[[sysfs文件系统]]**（虚拟文件系统，在内核空间和用户空间间建立关系，把内核的结构体变量的一些值以文件的形式展现出来，通过echo、cat操作属性节点）
(4)[[udev]]（实现内核空间和用户空间的信息的同步和转换）

#### 为什么需要设备驱动模型
(1)早期内核（2.4之前）没有统一的设备驱动模型，但照样可以用
(2)2.6版本中正式引入设备驱动模型，目的是在设备越来越多，功耗要求等新特性要求的情况下让驱动体系更易用、更优秀。
(3)设备驱动模型负责统一实现和维护一些特性，诸如：电源管理、热插拔、对象生命周期、用户空间和驱动空间的交互等基础设施
(4)设备驱动模型目的是简化驱动程序编写，但是客观上设备驱动模型本身设计和实现很复杂。

#### 驱动开发的2个点
(1)驱动源码本身编写、调试。重点在于对硬件的了解。
(2)驱动什么时候被安装、驱动中的函数什么时候被调用。跟硬件无关，完全和设备驱动模型有关。

### 3.2 设备驱动模型的底层架构
Linux设备模型的核心是使用Bus、Class、Device、Driver四个核心数据结构，将大量的、不同功能的硬件设备（以及驱动该硬件设备的方法），以树状结构的形式，进行归纳、抽象，从而方便Kernel的统一管理。

linux将这些数据结构的共同功能抽象出来，同一实现， 表示为 kobject。
- 通过parent指针，将所有kobject以层次结构的形式组合起来。
- 使用引用计数(reference count)来表示kobject被引用的次数，并且在ref变为0时，释放掉。
- 它与sysfs文件系统紧密相连，每个注册的kobject都对应sysfs文件系统中的一个目录。即"/sys/"下的某个目录。
- kobject：是总线、驱动、设备的三种对象的一个基类，实现公共接口。
- ktype：记录了 kobject 对象的一些属性。
- kset：是同类型 kobject 对象的集合，即一个容器； 是特殊的kobject，故也会在 `/sys` 下的某个目录。
![[Pasted image 20220829203405.png]]

### 3.3 kobject简介
Kobject实现了基本的面向对象管理机制，是构成Linux设备模型的核心结构。它与sysfs文件系统紧密相连，在内核中注册的每个kobject对象对应sysfs文件系统中的一个目录。
类似于C++中的基类，Kobject常被嵌入于其他类型（即：容器 kset）中。如bus,devices, drivers 都是典型的容器。这些容器通过kobject连接起来，形成了一个树状结构。

kset与kobject的关系：
![[Pasted image 20220829203539.png]]

kset, kobject,sysfs的关系：
![[Pasted image 20220829203614.png]]

#### kobject
```c
struct kobject {
	const char *name;
	struct list_head entry;  /*连接到kset建立层次结构*/
	struct kobject *parent;/*指向父节点，面向对象的层次架构。即反应到sysfs中为父目录*/
	struct kset	*kset; /*指向所属的kset*/
	struct kobj_type *ktype; /*属性文件*/
	struct sysfs_dirent *sd;  /*sysfs directory entry,对接虚拟文件系统*/
	struct kref	kref;    
#ifdef CONFIG_DEBUG_KOBJECT_RELEASE 
    struct delayed_work release; 
#endif
    /*初始化状态*/
	unsigned int state_initialized:1;
	/*是否处在sysfs下了*/
	unsigned int state_in_sysfs:1;
	unsigned int state_add_uevent_sent:1;
	unsigned int state_remove_uevent_sent:1;
	unsigned int uevent_suppress:1;
};
```
(1)定义在linux/kobject.h中
(2)各种对象最基本单元，提供一些公用型服务如：对象引用计数（kref，其实是帮助维护对象的生命周期，当引用计数归零，就可以释放了）、维护对象链表、对象上锁(竞争状态避免)、对用户空间的表示（kobj_type）
(3)设备驱动模型中的各种对象其内部都会包含一个kobject
(4)地位相当于面向对象体系架构中的总基类

#### kobj_type
```c
/*成员变量结构*/
/*1. 名称：kobj_type
     作用：Kobject的ktype成员是一个指向kobj_type结构的指针，该结构中记录了kobject对象的一些属性。
     注：
     release：用于释放kobject占用的资源，当kobject的引用计数为0时被调用。
*/
struct kobj_type {
	void (*release)(struct kobject *kobj);	/*用于释放kobject占用的资源,不同于close，我们可能要反复打开，在release时要判断他还有没有被别人打开，检测对象的引用计数*/
	const struct sysfs_ops *sysfs_ops;	/*属性文件的读写回调函数*/
	struct attribute **default_attrs;	/*默认属性文件列表*/
	const struct kobj_ns_type_operations *(*child_ns_type)(struct kobject *kobj);
	const void *(*namespace)(struct kobject *kobj);
};
```
(1)很多书中简称为ktype，每一个kobject都需要绑定一个ktype来提供相应功能（注意包含是包含了一个变量，而绑定是包含了一个指针）
(2)关键点1：sysfs_ops，提供该对象在sysfs中的操作方法（show和store）
(2)关键点2：attribute，提供在sysfs中以文件形式存在的属性，其实就是应用接口

#### kset
```c
struct kset {
	struct list_head list;
	spinlock_t list_lock;
	struct kobject kobj;// kset包含kobj，kobj绑定kset，所以kset是比kobj更大的结构体
	const struct kset_uevent_ops *uevent_ops;
};
```
(1)kset的主要作用是做顶层kobject的容器类
(2)kset的主要目的是将各个kobject（代表着各个对象）组织出目录层次架构
(3)可以认为kset就是为了在sysfs中弄出目录，从而让设备驱动模型中的多个对象能够有层次有逻辑性的组织在一起

#### kobject操作内核接口
```c
/*1.初始化kobject结构*/
void kobject_init(struct kobject * kobj)
 
/*2.将kobject对象注册到Linux系统*/
int kobject_add(struct kobject * kobj)
 
/*3.初始化kobject，并将其注册到linux系统*/
int kobject_init_and_add(struct kobject *kobj, struct kobj_type *ktype,
    struct kobject *parent, const char *fmt, ...)
 
/*4.从Linux系统中删除kobject对象*/
void kobject_del(struct kobject * kobj)
 
/*5.将kobject对象的引用计数加1，同时返回该对象指针。*/
struct kobject *kobject_get(struct kobject *kobj)
 
/*6.将kobject对象的引用计数减1，如果引用计数降为0，则调用release方法释放该kobject对象。*/
void kobject_put(struct kobject * kobj)
```

### 3.4 总线式设备驱动组织方式

#### bus
(1)物理上的真实总线及其作用（英文bus）
(2)驱动框架中的总线式设计
cpu总线式管理驱动，首先创建一些总线，如USB总线，pci总线，然后操作系统管理好总线就可以，由总线来管理驱动，总线还分两个分支，驱动和设备，设备之间、驱动之间由链表连起来，那么所有的设备来注册，比如插了一个热插拔usb设备，系统就将设备添加到usb总线设备里面去，然后到驱动的链表下面去找，安装相应的驱动。总线也是一堆代码，插入，删除，遍历，查找……
(3)bus_type结构体(总线的模板)，关键是match函数（做总线下面的设备和驱动的匹配）和uevent函数

#### device
(1)struct device是硬件设备在内核驱动框架中的抽象
(2)device_register用于向内核驱动框架注册一个设备
(3)通常device不会单独使用，而是被包含在一个具体设备结构体中，如struct usb_device

#### driver
(1)struct device_driver是驱动程序在内核驱动框架中的抽象
(2)关键元素1：name，驱动程序的名字，很重要，经常被用来作为驱动和设备的匹配依据
(3)关键元素2：probe，驱动程序的探测函数，用来检测一个设备是否可以被该驱动所管理

#### class
(1)相关结构体：struct class 和 struct class_device
(2)udev（热插拔的实现）的使用离不开class
(3)class的真正意义在于作为同属于一个class的多个设备的容器。类发明来就是来管理设备的，bus也管理设备，class也管理设备，设备是要进行多重管理的，这是不同的思路和管理方法。比如摄像头和U盘在bus角度看都是USB设备，但是在class角度看一个是大容量存储设备，一个是摄像头。（其实class和bus目录下都是devices的链接文件）也就是说，class是一种人造概念，目的就是为了对各种设备进行分类管理。当然，class在分类的同时还对每个类贴上了一些“标签”，这也是设备驱动模型为我们写驱动提供的基础设施。
一个设备需要有多种方法来管理，class和bus就是不同的管理方式。sys/devices才是真正的设备，从class或bus进去，最终都会指向devices目录下。

#### 总结
(1)模型思想很重要，其实就是面向对象的思想
(2)全是结构体套结构体，对基础知识要求很高
