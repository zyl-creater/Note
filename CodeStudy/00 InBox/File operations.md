
# File_operations结构体

*file_operation就是把系统调用和驱动程序关联起来的关键数据结构*。这个结构的每一个成员都对应着一个系统调用。读取 file_operation 中相应的函数指针，接着把控制权转交给函数，从而完成了 Linux 设备驱动程序的工作。 ^6b810d

在系统内部，I/O设备的存取操作通过特定的入口点来进行，而这组特定的入口点恰恰是由设备驱动程序提供的。通常这组设备驱动程序接口是由结构 file_operations 结构体向系统说明的，它定义在 include/linux/fs.h中。

传统上, 一个 file_operation 结构或者其一个指针称为 *fops* ( 或者它的一些变体). 结构中的每个成员必须指向驱动中的函数, 这些函数实现一些特定的操作, 或者对于不支持的操作留置为 NULL.，当指定为 NULL 指针时内核的确切的行为，每个函数是不同的。

在通读 file_operations 方法的列表时, 会注意到不少参数包含字串 \_user. 这种注解是一种文档形式, 注意, 一个指针是一个不能被直接引用的用户空间地址. 对于正常的编译, \_user 没有效果, 但是它可被外部检查软件使用来找出对用户空间地址的错误使用。

注册设备编号仅仅是驱动代码必须进行的诸多任务中的第一个。大部分的基础性的驱动操作包括 3个重要的内核数据结构，称为：file_operations、file、和 inode。
    **struct file_operations** 是一个字符设备把驱动的操作和设备号联系在一起的纽带，是一系列指针的集合，每个被打开的文件都对应于一系列的操作，这就是 file_operation，用来执行一系列的系统调用。  
    **struct file** 代表一个打开的文件，在执行 file_operation 中的 open 操作时被创建，这里需要注意的是与用户空间 inode 指针的区别，一个在内核，而 file 指针在用户空间，由 c 库来定义。  
    **struct inode** 被内核用来代表一个文件，注意和 struct file 的区别，struct inode 一个是代表文件，struct file 一个是代表打开的文件。

struct inode包括二个重要的成员：
    **dev_t            i_rdev   设备文件的设备号**
    **struct cdev   \*i_cdev 代表字符设备的数据结构**
    struct inode 结构是用来在内核内部表示文件的。同一个文件可以被打开好多次,所以可以对应很多struct file，但是只对应一个struct inode。


# File_operations的数据结构

```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
    ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
    int (*iterate) (struct file *, struct dir_context *);
    int (*iterate_shared) (struct file *, struct dir_context *);
    unsigned int (*poll) (struct file *, struct poll_table_struct *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    ssize_t (*sendpage) (struct file *, struct page *, int, size_t, loff_t *, int);
    unsigned long (*get_unmapped_area)(struct file *, unsigned long, unsigned long, unsigned long, unsigned long);
    int (*check_flags)(int);
    int (*flock) (struct file *, int, struct file_lock *);
    ssize_t (*splice_write)(struct pipe_inode_info *, struct file *, loff_t *, size_t, unsigned int);
    ssize_t (*splice_read)(struct file *, loff_t *, struct pipe_inode_info *, size_t, unsigned int);
    int (*setlease)(struct file *, long, struct file_lock **, void **);
    long (*fallocate)(struct file *file, int mode, loff_t offset,
              loff_t len);
    void (*show_fdinfo)(struct seq_file *m, struct file *f);
#ifndef CONFIG_MMU
    unsigned (*mmap_capabilities)(struct file *);
#endif
    ssize_t (*copy_file_range)(struct file *, loff_t, struct file *,
            loff_t, size_t, unsigned int);
    int (*clone_file_range)(struct file *, loff_t, struct file *, loff_t,
            u64);
    ssize_t (*dedupe_file_range)(struct file *, u64, u64, struct file *,
            u64);
};
```

**struct module \*owner;**
第一个 file_operations 成员不是一个操作, 它是一个指向拥有该结构的模块的指针，避免正在操作时被卸载，一般为初始化为THIS_MODULES (一个在 <linux/module.h> 中定义的宏)。

**loff_t (\*llseek) (struct file \*, loff_t, int);**
函数指针用来修改文件当前的读写位置，返回新位置.
loff_t 参数是一个"long offset", 并且就算在 32位平台上也至少 64 位宽. 错误由一个负返回值指示. 如果这个函数指针是 NULL, seek 调用会以潜在地无法预知的方式修改 file 结构中的位置计数器( 在"file 结构" 一节中描述).

**ssize_t (\*read) (struct file \*, char __user \*, size_t, loff_t \*);**
函数指针用来从设备中同步读取数据。读取成功返回读取的字节数。设置为NULL，调用时返回-EINVAL.  

**ssize_t (\*aio_read)(struct kiocb \*, char __user \*, size_t, loff_t);**
初始化一个异步读 -- 可能在函数返回前不结束的读操作. 如果这个方法是 NULL, 所有的操作会由 read 代替进行(同步地).

**ssize_t (\*write) (struct file \*, const char __user \*, size_t, loff_t \*);**
函数指针用来发送数据给设备. 如果 NULL, -EINVAL 返回给调用 write 系统调用的程序. 如果非负, 返回值代表成功写的字节数.

**ssize_t (\*aio_write)(struct kiocb \*, const char __user \*, size_t, loff_t \*);**
函数指针用来初始化一个异步的写入操作

**int (\*readdir) (struct file \*, void \*, filldir_t);**
函数指针仅用于读取目录，对于设备文件，该字段为 NULL  

**unsigned int (\*poll) (struct file \*, struct poll_table_struct \*);**
函数指针返回一个位掩码，用来指出非阻塞的读取或写入是否可能。并且, 提供给内核信息用来使调用进程睡眠直到 I/O 变为可能. 如果一个驱动把pool设置为 NULL，设备会被认为即可读也可写。

**int (\*ioctl) (struct inode \*, struct file \*, unsigned int, unsigned long);**
函数指针提供一种执行设备特殊命令的方法(例如格式化软盘的一个磁道, 这不是读也不是写)。不设置入口点，返回-ENOTTY.  

**int (\*mmap) (struct file \*, struct vm_area_struct \*);**
mmap 用来请求将设备内存映射到进程的地址空间. 如果这个方法是 NULL, mmap 系统调用返回 -ENODEV.

**int (\*open) (struct inode \*, struct file \*);**
指针函数用于打开设备,对设备文件进行的第一个操作, 不要求驱动声明一个对应的方法. 如果这个项是 NULL, 设备打开一直成功, 但系统不会通知驱动程序.

**int (\*flush) (struct file \*);**
指针函数用于在进程关闭设备文件描述符副本时，执行并等待，若设置为NULL，内核将忽略用户应用程序的请求。

**int (\*release) (struct inode \*, struct file \*);**
函数指针在file结构释放时，将被调用

**int (\*fsync) (struct file \*, struct dentry \*, int);**
函数指针用于刷新待处理的数据，如果驱动程序没有实现，fsync调用将返回-EINVAL.

**int (\*aio_fsync)(struct kiocb \*, int);**
函数对应异步fsync.

**int (\*fasync) (int, struct file \*, int);**
函数指针用于通知设备FASYNC标志发生变化，如果设备不支持异步通知，该字段可以为NULL.

**int (\*lock) (struct file \*, int, struct file_lock \*);**
lock 方法用来实现文件加锁; 加锁对常规文件是必不可少的特性, 但是设备驱动几乎从不实现它.

**ssize_t (\*readv) (struct file \*, const struct iovec \*, unsigned long, loff_t \*);**
**ssize_t (\*writev) (struct file \*, const struct iovec \*, unsigned long, loff_t \*);**
这些方法实现发散/汇聚读和写操作. 应用程序偶尔需要做一个包含多个内存区的单个读或写操作; 这些系统调用允许它们这样做而不必对数据进行额外拷贝. 如果这些函数指针为 NULL, read 和 write 方法被调用( 可能多于一次 ).

**ssize_t (\*sendfile)(struct file \*, loff_t \*, size_t, read_actor_t, void \*);**
这个方法实现 sendfile 系统调用的读, 使用最少的拷贝从一个文件描述符搬移数据到另一个. 例如, 它被一个需要发送文件内容到一个网络连接的 web 服务器使用. 设备驱动常常使 sendfile 为 NULL.

**ssize_t (\*sendpage) (struct file \*, struct page \*, int, size_t, loff_t \*, int);**
sendpage 是 sendfile 的另一半; 它由内核调用来发送数据, 一次一页, 到对应的文件. 设备驱动实际上不实现 sendpage.

**unsigned long (\*get_unmapped_area)(struct file \*, unsigned long, unsigned long, unsigned long, unsigned long);**
这个方法的目的是在进程的地址空间找一个合适的位置来映射在底层设备上的内存段中. 这个任务通常由内存管理代码进行; 这个方法存在为了使驱动能强制特殊设备可能有的任何的对齐请求. 大部分驱动可以置这个方法为 NULL.

**int (\*check_flags)(int)**
这个方法允许模块检查传递给 fnctl(F_SETFL...) 调用的标志.

**int (\*dir_notify)(struct file \*, unsigned long);**
这个方法在应用程序使用 fcntl 来请求目录改变通知时调用. 只对文件系统有用; 驱动不需要实现 dir_notify.

file_operations 结构是如下初始化的:

```c
struct file_operations fops = {
    .owner = THIS_MODULE,
    .llseek = scull_llseek,
    .read = scull_read,
    .write = scull_write,
    .ioctl = scull_ioctl,
    .open = scull_open,
    .release = scull_release,
};
```
