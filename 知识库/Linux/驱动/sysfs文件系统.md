sysfs文件系统

[[kobject]]
[[class]]
[[device]]
[[bus]]

**添加 sysfs 支持**

如果你正在[开发](https://link.zhihu.com/?target=http%3A//www.xxlinux.com/linux/article/development/) 的设备驱动程序中需要与用户层的接口，一般可选的方法有：

注册虚拟的字符设备文件， 以这个虚拟设备上的 read/write/ioctl 等接口与用户交互；但 read/write 一般只能做一件事情， ioctl 可以根据 cmd 参数做多个功能，但其缺点是很明显的： ioctl 接口无法直接在 Shell 脚本中使用，为了使用 ioctl 的功能，还必须编写配套的 C语言的虚拟设备操作程序， ioctl 的二进制数据接口也是造成大小端问题 (big endian与little endian)、32位/64位不可移植问题的根源；  
注册 proc 接口， 接受用户的 read/write/ioctl 操作；同样的，一个 proc 项通常使用其 read/write/ioctl 接口，它所存在的问题与上面的虚拟字符设备的的问题相似；  
注册 sysfs 属性 ；  
最重要的是，添加虚拟字符设备支持和注册 proc 接口支持这两者所需要增加的代码量都并不少，最好的方法还是使用 sysfs 属性支持，一切在用户层是可见的透明，且增加的代码量是最少的，可维护性也最好； 方法 就是使用 <include/linux/device.h> 头文件提供的这四个宏，分别应用于总线/类别/驱动/设备四种[内核](https://link.zhihu.com/?target=http%3A//www.xxlinux.com/linux/article/development/kernel/) 数据结构对象上：