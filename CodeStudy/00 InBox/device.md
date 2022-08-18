# 一、class_create

开始写Linux设备驱动程序的时候，很多时候都是利用mknod命令手动创建设备节点(包括ldd3中不少例子也是这样)，实际上现在Linux内核为我们提供了一组函数，可以用来在模块加载的时候自动在/dev目录下创建相应设备节点，并在卸载模块时删除该节点。

内核中定义了struct class结构体，顾名思义，一个struct class结构体类型变量对应一个类，内核同时提供了class_create(…)函数，可以用它来创建一个类，这个类存放于sysfs下面，一旦创建 好了这个类，再调用device_create(…)函数来在/dev目录下创建相应的设备节点。这样，加载模块的时候，用户空间中的udev会自动响应 device_create(…)函数，去/sysfs下寻找对应的类从而创建设备节点。

此外，利用device_create_file函数可以在/sys/class/下创建对应的属性文件，从而通过对该文件的读写实现特定的数据操作。

```c
/* This is a #define to keep the compiler from merging different
 * instances of the __key variable */
#define class_create(owner, name)		\
({						\
  static struct lock_class_key __key;	\
  __class_create(owner, name, &__key);	\
})

/**
 * class_create - create a struct class structure
 * @owner: pointer to the module that is to "own" this struct class
 * @name: pointer to a string for the name of this class.
 * @key: the lock_class_key for this class; used by mutex lock debugging
 *
 * This is used to create a struct class pointer that can then be used
 * in calls to device_create().
 *
 * Returns &struct class pointer on success, or ERR_PTR() on error.
 *
 * Note, the pointer created here is to be destroyed when finished by
 * making a call to class_destroy().
 */
struct class *__class_create(struct module *owner, const char *name,
           struct lock_class_key *key)
```

关键的一句是：

* This is used to create a struct class pointer that can then be used in calls to device_create().
 -->这个函数用来创建一个struct class的结构体指针，这个指针可用作device_create()函数的参数。

也就是说，这个函数主要是在调用device_create()前使用，创建一个struct class类型的变量，并返回其指针。  

# 二、device_create

官方说明：
```c
/**
 * device_create - creates a device and registers it with sysfs
 * @class: pointer to the struct class that this device should be registered to
 * @parent: pointer to the parent struct device of this new device, if any
 * @devt: the dev_t for the char device to be added
 * @drvdata: the data to be added to the device for callbacks
 * @fmt: string for the device's name
 *
 * This function can be used by char device classes.  A struct device
 * will be created in sysfs, registered to the specified class.
 *
 * A "dev" file will be created, showing the dev_t for the device, if
 * the dev_t is not 0,0.
 * If a pointer to a parent struct device is passed in, the newly created
 * struct device will be a child of that device in sysfs.
 * The pointer to the struct device will be returned from the call.
 * Any further sysfs files that might be required can be created using this
 * pointer.
 *
 * Returns &struct device pointer on success, or ERR_PTR() on error.
 *
 * Note: the struct class passed to this function must have previously
 * been created with a call to class_create().
 */
struct device *device_create(struct class *class, struct device *parent,
           dev_t devt, void *drvdata, const char *fmt, ...)
```

首先解释一下"sysfs"：sysfs是linux2.6所提供的一种虚拟档案系统；在设备模型中，sysfs文件系统用来表示设备的结构，将设备的层次结构形象的反应到用户空间中，从而可以通过修改sysfs中的文件属性来修改设备的属性值；sysfs被挂载到根目录下的"/sys"文件夹下。

官方说明：
```c
/**
 * device_create_file - create sysfs attribute file for device.
 * @dev: device.
 * @attr: device attribute descriptor.
 */
int device_create_file(struct device *dev,
           const struct device_attribute *attr)
```

使用这个函数时要引用 device_create 所返回的 device\* 指针，作用是在 /sys/class/ 下创建一个属性文件，从而通过对这个属性文件进行读写就能完成对应的数据操作。  
如：  

## a.在驱动程序中使用 device_create_file创建属性文件

```c
static DEVICE_ATTR(val, S_IRUGO | S_IWUSR, hello_val_show, hello_val_store);  

/*读取寄存器val的值到缓冲区buf中，内部使用*/  
static ssize_t __hello_get_val(struct xxx_dev* dev, char* buf) {  
  int val = 0;		  
  
  /*同步访问*/  
  if(down_interruptible(&(dev->sem))) {				  
    return -ERESTARTSYS;		  
  }		  
  
  val = dev->val;		  
  up(&(dev->sem));		  
  
  return snprintf(buf, PAGE_SIZE, "%d/n", val);  
}  
  
/*把缓冲区buf的值写到设备寄存器val中去，内部使用*/  
static ssize_t __hello_set_val(struct xxx_dev* dev, const char* buf, size_t count) {  
  int val = 0;		  
  
  /*将字符串转换成数字*/		  
  val = simple_strtol(buf, NULL, 10);		  
  
  /*同步访问*/		  
  if(down_interruptible(&(dev->sem))) {				  
    return -ERESTARTSYS;		  
  }		  
  
  dev->val = val;		  
  up(&(dev->sem));  
  
  return count;  
}  
  
/*读取设备属性val*/  
static ssize_t hello_val_show(struct device* dev, struct device_attribute* attr, char* buf) {  
  struct xxx_dev* hdev = (struct xxx_dev*)dev_get_drvdata(dev);		  
  
  return __hello_get_val(hdev, buf);  
}  
  
/*写设备属性val*/  
static ssize_t hello_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count) {   
  struct xxx_dev* hdev = (struct xxx_dev*)dev_get_drvdata(dev);	
    
  return __hello_set_val(hdev, buf, count);  
} 

/*模块加载方法*/  
static int __init xxx_init(void){   
  ... 
  
  /*在/sys/class/xxx/xxx目录下创建属性文件val*/  
  err = device_create_file(temp, &dev_attr_val);  
  if(err < 0) {  
    printk(KERN_ALERT"Failed to create attribute val.");				  
    goto destroy_device;  
  }
  
  ...
}
```

## b.在用户空间读取属性

```c
...
read(dev->fd, val, sizeof(*val));
...
write(dev->fd, &val, sizeof(val));
...
```

# 三、使用示例

```c
/*在/sys/class/目录下创建设备类别目录xxx*/ 
  g_vircdev_class = class_create(THIS_MODULE, VIRCDEV_CLASS_NAME);
  if(IS_ERR(g_vircdev_class)) {  
    err = PTR_ERR(g_vircdev_class);  
    printk(KERN_ALERT "Failed to create class.\n");  
    goto CLASS_CREATE_ERR;  
  }
  
  /*在/dev/目录和/sys/class/xxx目录下分别创建设备文件xxx*/
  dev = device_create(g_vircdev_class, NULL, devt, NULL, VIRCDEV_DEVICE_NAME);
  if(IS_ERR(dev)) {  
    err = PTR_ERR(dev);  
    printk(KERN_ALERT "Failed to create device.\n");  
    goto DEVICE_CREATE_ERR;  
  }
  
  /*在/sys/class/xxx/xxx目录下创建属性文件val*/ 
  err = device_create_file(dev, attr);  
  if(err < 0) {  
    printk(KERN_ALERT"Failed to create attribute file.");				  
    goto DEVICE_CREATE_FILE_ERR;  
  }
```
