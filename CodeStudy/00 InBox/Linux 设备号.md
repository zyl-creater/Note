
# 基本信息

1.  设备号的数据类型：dev_t  
    `typedef unsigned int dev_t;`
    
2.  设备号是一个统称，分为主设备号和次设备号。  
    主设备号保存在**高12位**；  
    次设备号保存在**低20位**。
    
3.  设备号的功能  
    主设备号：**一个设备驱动对应一个主设备号**。应用程序通过设备文件中的主设备号在内核中找到对应的设备驱动。  
    次设备号：**一个设备对应一个次设备号**。当一个驱动程序管理多个设备时，驱动程序通过设备文件中的次设备号，找到需要操作的设备。
    

# 操作宏

```c
/* 通过已知的主次设备号，合成设备号 */
dev_t dev = MKDEV(major, minor);   
/* 通过已知的设备号，获取主设备号 */
major = MAJOR(dev);   
/* 通过已知的设备号，获取次设备号 */
minor = MINOR(dev);    
```

上述宏的实现：

```c
#define MINORBITS   20
#define MINORMASK   ((1U << MINORBITS) - 1)

#define MAJOR(dev)  ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev)  ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi)    (((ma) << MINORBITS) | (mi))
```

# 相关结构体

kernel用来保存设备号的结构体char_device_struct，来来看看这个结构体：

```c
static struct char_device_struct {
    struct char_device_struct *next;
    unsigned int major;
    unsigned int baseminor;
    int minorct;
    char name[64];
    struct cdev *cdev;      /* will die */
} *chrdevs[CHRDEV_MAJOR_HASH_SIZE];
```

内核为了管理设备号，维护了一个char_device_struct结构的散列表——chrdevs。  
![[Pasted image 20220815110146.png]]
设备号示意图

  
chrdevs的每一个成员，都是一个以char_device_struct结构体指针为元素的**单项链表**的**头指针**。  
每条链表中的所有元素拥有相同的主设备号，及该链表所在chrdevs数组的下标。

# 操作函数

内核提供的可以用来申请设备号的函数有三个。
```c
register_chrdev()
register_chrdev_region()
alloc_chrdev_region()
```

**register_chrdev( )**
```c
/* 字符设备注册函数 */
static int register_chrdev(unsigned int major, const char *name,
                                  const struct file_operations *fops)
```
值得注意的是，register_chrdev 函数也可以实现动态分配设备号的注册，只需要将主设备号 major 设为 0 即为动态注册。也就是说，major 为 非 0 值则表示静态分配 major 本身的设备号，major 为 0 则为动态分配未使用的设备号。而后下面所说的两种函数实际上是将 register_chrdev( ) 的静态、动态分配功能给分割了。**name是设备名称，可通过 cat /proc/devices 查看**
这个函数不只帮我们注册设备号，还帮我们做了cdev 的初始化以及 cdev 的注册

其中，register_chrdev_region 函数是静态注册，alloc_chrdev_region 函数是动态注册。

利用 **cat /proc/devices** 命令可以查看这个系统中所有已经被占用的设备号。

如果我们明确的知道所需要的设备号时，可以使用下面的函数：
```c
int register_chrdev_region(dev_t from, unsigned count, const char *name)
{
    struct char_device_struct *cd;
    dev_t to = from + count;
    dev_t n, next;

    for (n = from; n < to; n = next) {
        next = MKDEV(MAJOR(n)+1, 0);
        if (next > to)
            next = to;
        cd = __register_chrdev_region(MAJOR(n), MINOR(n),
                   next - n, name);
        if (IS_ERR(cd))
            goto fail;
    }
    return 0;
fail:
    to = n;
    for (n = from; n < to; n = next) {
        next = MKDEV(MAJOR(n)+1, 0);
        kfree(__unregister_chrdev_region(MAJOR(n), MINOR(n), next - n));
    }
    return PTR_ERR(cd);
}
```
from :         dev_t 类型，要分配的设备编号范围的初始值，明确了主设备号和起始次设备号；
count :     设备个数，即次设备号个数；
name :     相关联的设备名称（可在 /proc/devices 目录下查看到），也即本组设备的驱动名称。

需要注意的一个细节是，使用next和to进行比较，如果next<to，说明count太大了，超过了此设备号从from开始，能够容纳的次设备号的数量。这个时候需要先将from能够使用的次设备号申请完毕（申请的数量：MKDEV(MAJOR(from)+1, 0) - from），再使用下一个主设备号进行申请（(from + count) - MKDEV(MAJOR(from)+1, 0)。

如果需要内核帮我们动态的分配设备号，使用下面的函数：

```c
int alloc_chrdev_region(dev_t *dev, unsigned baseminor, unsigned count,
            const char *name)
{
    struct char_device_struct *cd;
    cd = __register_chrdev_region(0, baseminor, count, name);
    if (IS_ERR(cd))
        return PTR_ERR(cd);
    *dev = MKDEV(cd->major, cd->baseminor);
    return 0;
}
```
dev :                保存申请到的主设备号，可以通过 *major=MMOR(dev)* 获取主设备号。
baseminor :     次设备号起始地址，alloc_chrdev_region 可以申请一段连续的多个设备号，这些设 
                        备号的主设备号一样，但是次设备号不同，次设备号以 baseminor 为起始地址地址开始递增，一般 baseminor 为 0，也就是说次设备号从 0 开始；
count :             要申请的设备号数量；
name :             设备名字；
返回值 :           小于 0，则分配设备号错误；成功则数据号被 \**dev* 带出。

上面个两个申请设备号的函数都调用了`__register_chrdev_region`函数，下面具体看一下这个真正分配设备号的函数：

```c
static struct char_device_struct *
__register_chrdev_region(unsigned int major, unsigned int baseminor,
               int minorct, const char *name)
{
    struct char_device_struct *cd, **cp;
    int ret = 0;
    int i;

    cd = kzalloc(sizeof(struct char_device_struct), GFP_KERNEL);
    if (cd == NULL)
        return ERR_PTR(-ENOMEM);

    mutex_lock(&chrdevs_lock);

    /* temporary */
    if (major == 0) {
        for (i = ARRAY_SIZE(chrdevs)-1; i > 0; i--) {
            if (chrdevs[i] == NULL)
                break;
        }

        if (i == 0) {
            ret = -EBUSY;
            goto out;
        }
        major = i;
    }

    cd->major = major;
    cd->baseminor = baseminor;
    cd->minorct = minorct;
    strlcpy(cd->name, name, sizeof(cd->name));

    i = major_to_index(major);

    for (cp = &chrdevs[i]; *cp; cp = &(*cp)->next)
        if ((*cp)->major > major ||
            ((*cp)->major == major &&
             (((*cp)->baseminor >= baseminor) ||
              ((*cp)->baseminor + (*cp)->minorct > baseminor))))
            break;

    /* Check for overlapping minor ranges.  */
    if (*cp && (*cp)->major == major) {
        int old_min = (*cp)->baseminor;
        int old_max = (*cp)->baseminor + (*cp)->minorct - 1;
        int new_min = baseminor;
        int new_max = baseminor + minorct - 1;

        /* New driver overlaps from the left.  */
        if (new_max >= old_min && new_max <= old_max) {
            ret = -EBUSY;
            goto out;
        }

        /* New driver overlaps from the right.  */
        if (new_min <= old_max && new_min >= old_min) {
            ret = -EBUSY;
            goto out;
        }
    }

    cd->next = *cp;
    *cp = cd;
    mutex_unlock(&chrdevs_lock);
    return cd;
out:
    mutex_unlock(&chrdevs_lock);
    kfree(cd);
    return ERR_PTR(ret);
}
```

\_\_register_chrdev_region第一个参数是主设备号，在alloc_chrdev_region中第一个参数给了0，也就是要求动态分配主设备号。

```c
    /* temporary */
    if (major == 0) {
        for (i = ARRAY_SIZE(chrdevs)-1; i > 0; i--) {
            if (chrdevs[i] == NULL)
                break;
        }

        if (i == 0) {
            ret = -EBUSY;
            goto out;
        }
        major = i;
    }
```

\_\_register_chrdev_region第一个参数是主设备号，如果传递0，进入上述if语句，从最大值（`ARRAY_SIZE(chrdevs)`的大小为`#define CHRDEV_MAJOR_HASH_SIZE 255`）开始遍历chrdevs数组，直到chrdevs[1]或者遇到`chrdevspi[ == NULL`为止。（也就是说，**主设备号0和255不会被动态分配**）  
如果1到244没有空闲的主设备号，则分配失败。

```c
i = major_to_index(major);

for (cp = &chrdevs[i]; *cp; cp = &(*cp)->next)
    if ((*cp)->major > major ||
        ((*cp)->major == major &&
         (((*cp)->baseminor >= baseminor) ||
          ((*cp)->baseminor + (*cp)->minorct > baseminor))))
        break;
```

major_to_index()是个取模操作：

```c
/* fs/char_dev.c */
#define CHRDEV_MAJOR_HASH_SIZE  255
/* index in the above */
static inline int major_to_index(unsigned major)
{
    return major % CHRDEV_MAJOR_HASH_SIZE;
}
```

由这个函数也可以了解到，**主设备号可以大于255**。当然了如果是动态分配，是不会出现这种情况的。

上述的for循环的目的，是为即将被分配的设备号寻找一个合适的位置（\*cp指向的位置）。  
上述for循环break的条件：

-   \*cp == NULL。（cd节点放到\*cp的位置就可以了）
-   \*cp指向的节点的major > 即将分配的major。（cd节点放到\*cp节点前面一个位置）
-   \*cp指向的节点的major == 即将分配的major；\*cp指向的节点次设备号的起始值 >= 即将分配的设备号的次设备号的起始值。（需要进一步check）
-   \*cp指向的节点的major == 即将分配的major；\*cp指向节点的次设备号的结束值 > 即将分配的设备号的次设备号的起始值。（需要进一步check）

上面的for循环有两个遗留问题需要进一步check，下面的就是检查minior是否重叠的代码：

```c
/* Check for overlapping minor ranges.  */
if (*cp && (*cp)->major == major) {
    int old_min = (*cp)->baseminor;
    int old_max = (*cp)->baseminor + (*cp)->minorct - 1;
    int new_min = baseminor;
    int new_max = baseminor + minorct - 1;

    /* New driver overlaps from the left.  */
    if (new_max >= old_min && new_max <= old_max) {
        ret = -EBUSY;
        goto out;
    }

    /* New driver overlaps from the right.  */
    if (new_min <= old_max && new_min >= old_min) {
        ret = -EBUSY;
        goto out;
    }
}
```

这就是两个集合是否重叠的问题。画图更容易理解。

```c
cd->next = *cp;
*cp = cd;
mutex_unlock(&chrdevs_lock);
return cd;
```

将cd插入到 \*cp指向的节点前面。

最后是设备号释放的方法：

用 register_chrdev( ) 来注册，用 unregister_chrdev( ) 来注销；
用 register_chrdev_region( ) / alloc_chrdev_region( ) 来注册，用 unregister_chrdev_region( ) 来注销。

*unregister_chrdev( )* 字符设注销函数
```c
/* 字符设备注销函数 */
static void unregister_chrdev(unsigned int major, const char *name)
```
major :         要注销设备对应的主设备号；
name :         要注销设备对应的设备名，指向一串字符串。


*unregister_chrdev_region( )* 字符设注销函数
```c
/* 注销 register_chrdev_region / alloc_chrdev_region 函数所注册设备的函数 */
static void unregister_chrdev_region(dev_t from, unsigned count)
```
from :        要释放的设备号；
count :      表示从 from 开始，要释放的设备号数量。