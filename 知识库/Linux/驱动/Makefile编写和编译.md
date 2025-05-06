## Makefile编写和编译

在编译内核模块前，先准备一个 Makefile 文件：
`Makefile`
```makefile
MODULE_NAME = helloworld
obj-m := $(MODULE_NAME).o

ARCH ?= arm64
CROSS_COMPILE ?= aarch64-linux-gnu-
KSRC := /home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/out/target/product/ohm/obj/KERNEL_OBJ

modules:
    echo "====== $(MODULE_NAME) ======"
    $(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)  modules                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         

clean:
    @rm -fr Module.symvers ; rm -fr Module.markers ; rm -fr modules.order
    @rm -fr *.mod.c *.mod *.o .*.cmd *.ko .*.o.d *~
    @rm -fr .tmp_versions
```

**！！注意 `$(MAKE)` 前必须得是 `Tab` 缩进符。**

obj-m 表示把文件 helloworld.o 作为"模块"进行编译，不会编译到内核，但是会生成一个独立的 "helloworld.ko" 文件；

obj-y 表示把 helloworld.o 文件编译进内核;

`$(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)  modules`

### ARCH

^ea2762

即 architecture，就是选择编译哪一种 cpu architecture，也就是编译 arch/ 目录下的哪一个子目录。如指定 make ARCH=arm64 就是编译 arch/arm64下的代码。如果不指定，make 将使用本机（用什么机器编译就是什么）的cpu作为缺省 ARCH。注意：arch/arm64 下不但有 arm64 体系架构特有的代码，还有 arm64 特有的 kconfig，也就是配置选项，所以在 make menuconfig，make xxxx_defconfig 的时候也必须指定 ARCH＝arm64。

### CROSS_COMPILE

^9b068e

即交叉编译器的前缀（prefix），也就是选择将代码编译成目标 cpu 的指令的工具，如指定 make CROSS_COMPILE=aarch64-linux-gnu- 就是使用 aarch64-linux-gnu-gcc，aarch64-linux-gnu-ld等工具将代码编译成 arm64 的可执行指令。如果不指定 CROSS_COMPILE 参数，make 时将认为prefix 为空，即使用 gcc 来编译。这里 cross_compile 的设置，是假定所用的交叉工具链的 gcc 程序名称为 aarch64-linux-gnu-gcc。如果实际使用的 gcc 名称是 some-thing-else-gcc，则这里照葫芦画瓢填 some-thing-else- 即可。总之，要省去名称中最后的gcc那3个字母。

### -C $(KSRC)

^bfd954

内核源代码所在的目录。"make" 将实际更改到指定的目录执行时，并在完成时返回。

### M=$(shell pwd)

^fa1876

通知kbuild正在构建外部模块。给“M”的值是外部模块（kbuild文件）所在目录的绝对路径。最后生成的中间文件也在此目录下。

### MODULES

^f676ba

默认将构建位于当前目录中的模块，因此不需要指定目标。所有输出文件也将在这个目录中生成。没有尝试更新内核源代码，并且成功地为内核执行“make”是一个先决条件。

### makefile变量

^914004

在 Makefile 中的定义的变量，就像是 C/C++ 语言中的宏一样，他代表了一个文本字串，在Makefile 中执行的时候其会自动原模原样地展开在所使用的地方。其与 C/C++所不同的是，你可以在 Makefile中改变其值。在 Makefile 中，变量可以使用在“目标”，“依赖目标”， “命令”或是Makefile 的其它部分中。

变量的命名字可以包含字符、数字，下划线（可以是数字开头），但不应该含有 `:` 、 `#` 、 `=` 或是空字符（空格、回车等）。变量是大小写敏感的，“foo”、“Foo”和“FOO”是三个不同的变量名。传统的 Makefile 的变量名是全大写的命名方式，但推荐使用大小写搭配的变量名，如：MakeFlags。这样可以避免和系统的变量冲突，而发生意外的事情。

有一些变量是很奇怪字串，如 `$<` 、 `$@` 等，这些是自动化变量。

在定义变量的值时，我们可以使用其它变量来构造变量的值，在Makefile中有两种方式来在用变量定义变量的值。

先看第一种方式，也就是简单的使用 `=` 号，在 `=` 左侧是变量，右侧是变量的值，右侧变量的值可以定义在文件的任何一处，也就是说，右侧中的变量不一定非要是已定义好的值，其也可以使用后面定义的值。如：
```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:
    echo $(foo)
```

我们执行“make all”将会打出变量 `$(foo)` 的值是 `Huh?` （ `$(foo)` 的值是 `$(bar)` ， `$(bar)` 的值是 `$(ugh)` ， `$(ugh)` 的值是 `Huh?` ）可见，变量是可以使用后面的变量来定义的。

这个功能有好的地方，也有不好的地方，好的地方是，我们可以把变量的真实值推到后面来定义，如：
```makefile
CFLAGS = $(include_dirs) -O
include_dirs = -Ifoo -Ibar
```

当 `CFLAGS` 在命令中被展开时，会是 `-Ifoo -Ibar -O` 。但这种形式也有不好的地方，那就是递归定义，如：
```makefile
CFLAGS = $(CFLAGS) -O

或：
A = $(B)
B = $(A)
```

这会让make陷入无限的变量展开过程中去，当然，我们的make是有能力检测这样的定义，并会报错。还有就是如果在变量中使用函数，那么，这种方式会让我们的make运行时非常慢，更糟糕的是，他会使用得两个make的函数“wildcard (通配符) ”和“shell”发生不可预知的错误。因为你不会知道这两个函数会被调用多少次。

为了避免上面的这种方法，我们可以使用 make 中的另一种用变量来定义变量的方法。这种方法使用的是 `:=` 操作符，如：
```makefile
x := foo
y := $(x) bar
x := later

其等价于：
y := foo bar
x := later
```

值得一提的是，这种方法，前面的变量不能使用后面的变量，只能使用前面已定义好了的变量。如果是这样：
```makefile
y := $(x) bar
x := foo
```

那么，y的值是“bar”，而不是“foo bar”。

上面都是一些比较简单的变量使用了，让我们来看一个复杂的例子，其中包括了make的函数、条件表达式和一个系统变量“MAKELEVEL”的使用：
```makefile
ifeq (0,${MAKELEVEL})
cur-dir   := $(shell pwd)
whoami    := $(shell whoami)
host-type := $(shell arch)
MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

关于条件表达式和函数，我们在后面再说，对于系统变量“MAKELEVEL”，其意思是，如果我们的make有一个嵌套执行的动作（参见前面的“嵌套使用make”），那么，这个变量会记录了我们的当前Makefile的调用层数。

下面再介绍两个定义变量时我们需要知道的，请先看一个例子，如果我们要定义一个变量，其值是一个空格，那么我们可以这样来：
```makefile
nullstring :=
space := $(nullstring) # end of the line
```

nullstring是一个Empty变量，其中什么也没有，而我们的space的值是一个空格。因为在操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止，这样，我们可以定义出其值是一个空格的变量。请注意这里关于“#”的使用，注释符“#”的这种特性值得我们注意，如果我们这样定义一个变量：
```makefile
dir := /foo/bar    # directory to put the frobs in
```

dir这个变量的值是“/foo/bar”，后面还跟了4个空格，如果我们这样使用这个变量来指定别的目录——“$(dir)/file”那么就完蛋了。

还有一个比较有用的操作符是 `?=` ，先看示例：
```makefile
FOO ?= bar
```

其含义是，如果FOO没有被定义过，那么变量FOO的值就是“bar”，如果FOO先前被定义过，那么这条语将什么也不做，其等价于：
```makefile
ifeq ($(origin FOO), undefined)
    FOO = bar
endif
```

### 变量用法

^c44792

#### 变量值的替换。

^938ddf

我们可以替换变量中的共有的部分，其格式是 `$(var:a=b)` 或是 `${var:a=b}` ，其意思是，把变量“var”中所有以“a”字串“结尾”的“a”替换成“b”字串。这里的“结尾”意思是“空格”或是“结束符”。
还是看一个示例吧：
```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```
这个示例中，我们先定义了一个 `$(foo)` 变量，而第二行的意思是把 `$(foo)` 中所有以 `.o` 字串“结尾”全部替换成 `.c` ，所以我们的 `$(bar)` 的值就是“a.c b.c c.c”。

另外一种变量替换的技术是以“静态模式”（参见前面章节）定义的，如：
```makefile
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```
这依赖于被替换字串中的有相同的模式，模式中必须包含一个 `%` 字符，这个例子同样让 `$(bar)` 变量的值为“a.c b.c c.c”。
    
#### 把变量的值再当成变量

^44b20f

先看一个例子：
```makefile
x = y
y = z
a := $($(x))
```
在这个例子中，`$(x)` 的值是 “y”，所以 `$($(x))` 就是 `$(y)` ，于是 `$(a)` 的值就是 “z”。（注意，是“x=y”，而不是“x=$(y)”）

我们还可以使用更多的层次：
```makefile
x = y
y = z
z = u
a := $($($(x)))
```
这里的 `$(a)` 的值是“u”，相关的推导留给读者自己去做吧。

让我们再复杂一点，使用上“在变量定义中使用变量”的第一个方式，来看一个例子：
```makefile
x = $(y)
y = z
z = Hello
a := $($(x))
```
这里的 `$($(x))` 被替换成了 `$($(y))` ，因为 `$(y)` 值是“z”，所以，最终结果是： `a:=$(z)` ，也就是“Hello”。

再复杂一点，我们再加上函数：
```makefile
x = variable1
variable2 := Hello
y = $(subst 1,2,$(x))
z = y
a := $($($(z)))
```
这个例子中， `$($($(z)))` 扩展为 `$($(y))` ，而其再次被扩展为 `$($(subst 1,2,$(x)))` 。 `$(x)` 的值是“variable1”，subst函数把“variable1”中的所有“1”字串替换成“2”字串，于是，“variable1”变成 “variable2”，再取其值，所以，最终， `$(a)` 的值就是 `$(variable2)` 的值——“Hello”。

在这种方式中，或要可以使用多个变量来组成一个变量的名字，然后再取其值：
```makefile
first_second = Hello
a = first
b = second
all = $($a_$b)
```
这里的 `$a_$b` 组成了“first_second”，于是， `$(all)` 的值就是“Hello”。

再来看看结合第一种技术的例子：
```makefile
a_objects := a.o b.o c.o
1_objects := 1.o 2.o 3.o

sources := $($(a1)_objects:.o=.c)
```
这个例子中，如果 `$(a1)` 的值是“a”的话，那么， `$(sources)` 的值就是“a.c b.c c.c”；如果 `$(a1)` 的值是“1”，那么 `$(sources)` 的值是“1.c 2.c 3.c”。

再来看一个这种技术和“函数”与“条件语句”一同使用的例子：
```makefile
ifdef do_sort
    func := sort
else
    func := strip
endif

bar := a d b g q c

foo := $($(func) $(bar))
```
这个示例中，如果定义了“do_sort”，那么： `foo := $(sort a d b g q c)` ，于是 `$(foo)` 的值就是 “a b c d g q”，而如果没有定义“do_sort”，那么： `foo := $(strip a d b g q c)` ，调用的就是strip函数。

当然，“把变量的值再当成变量”这种技术，同样可以用在操作符的左边:
```makefile
dir = foo
$(dir)_sources := $(wildcard $(dir)/*.c)
define $(dir)_print
lpr $($(dir)_sources)
endef
```
这个例子中定义了三个变量：“dir”，“foo_sources”和“foo_print”。

#### 追加变量

^0df296

我们可以使用 `+=` 操作符给变量追加值，如：
```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```
于是，我们的 `$(objects)` 值变成：“main.o foo.o bar.o utils.o another.o”（another.o被追加进去了）

使用 `+=` 操作符，可以模拟为下面的这种例子：
```makefile
objects = main.o foo.o bar.o utils.o
objects := $(objects) another.o
```

所不同的是，用 `+=` 更为简洁。

如果变量之前没有定义过，那么， `+=` 会自动变成 `=` ，如果前面有变量定义，那么 `+=` 会继承于前次操作的赋值符。如果前一次的是 `:=` ，那么 `+=` 会以 `:=` 作为其赋值符，如：
```makefile
variable := value
variable += more

等价于：
variable := value
variable := $(variable) more
```

但如果是这种情况：
```makefile
variable = value
variable += more
```
由于前次的赋值符是 `=` ，所以 `+=` 也会以 `=` 来做为赋值，那么岂不会发生变量的递补归定义，这是很不好的，所以make会自动为我们解决这个问题，我们不必担心这个问题。

#### 目标变量

^ba7201

前面我们所讲的在Makefile中定义的变量都是“全局变量”，在整个文件，我们都可以访问这些变量。当然，“自动化变量”除外，如 `$<` 等这种类量的自动化变量就属于“规则型变量”，这种变量的值依赖于规则的目标和依赖目标的定义。

当然，我也同样可以为某个目标设置局部变量，这种变量被称为“Target-specific Variable”，它可以和“全局变量”同名，因为它的作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值。

其语法是：
```makefile
<target ...> : <variable-assignment>;

<target ...> : overide <variable-assignment>
```
`<variable-assignment>`；可以是前面讲过的各种赋值表达式，如 `=` 、 `:=` 、 `+=` 或是 `?=` 。第二个语法是针对于make命令行带入的变量，或是系统环境变量。

这个特性非常的有用，当我们设置了这样一个变量，这个变量会作用到由这个目标所引发的所有的规则中去。如：
```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
    $(CC) $(CFLAGS) prog.o foo.o bar.o

prog.o : prog.c
    $(CC) $(CFLAGS) prog.c

foo.o : foo.c
    $(CC) $(CFLAGS) foo.c

bar.o : bar.c
    $(CC) $(CFLAGS) bar.c
```
在这个示例中，不管全局的 `$(CFLAGS)` 的值是什么，在prog目标，以及其所引发的所有规则中（prog.o foo.o bar.o的规则）， `$(CFLAGS)` 的值都是 `-g`

当完成 Makefile 编写时，在 Makefile 目录下执行 **make**，产生可执行文件和中间文件
```shell
zhuyuelin@ubuntu:~/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test$ make
echo "====== helloworld ======"
====== helloworld ======
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -C /home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/out/target/product/ohm/obj/KERNEL_OBJ M=/home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test modules 
make[1]: Entering directory '/home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/out/target/product/ohm/obj/KERNEL_OBJ'
arch/arm64/Makefile:27: ld does not support --fix-cortex-a53-843419; kernel may be susceptible to erratum
  CC [M]  /home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test/helloworld.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test/helloworld.mod.o
  LD [M]  /home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test/helloworld.ko
make[1]: Leaving directory '/home/zhuyuelin/amlogic/q/q-amlogic-20210208-gtvs.xml/out/target/product/ohm/obj/KERNEL_OBJ'

#### build completed successfully (3 seconds) ####

zhuyuelin@ubuntu:~/amlogic/q/q-amlogic-20210208-gtvs.xml/vendor/sdmc/drivers/test$ ls
helloworld.c  helloworld.ko  helloworld.mod.c  helloworld.mod.o  helloworld.o  Makefile  modules.order  Module.symvers
```
