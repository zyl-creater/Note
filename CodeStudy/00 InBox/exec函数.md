# 1. exec函数说明

fork 子进程是为了执行新程序（fork创建了子进程后，子进程和父进程同时被OS调度执行，因此子进程可以单独的执行一个程序，这个程序宏观上将会和父进程程序同时进行）

可以直接在子进程的 if 中写入新程序的代码。这样可以，但是不够灵活，因为我们只能把子进程程序的源代码贴过来执行（必须知道源代码，而且源代码太长了也不好控制），譬如说我们希望子进程来执行 **ls -la** 命令就不行了（没有源代码，只有编译好的可执行程序）

使用exec族运行新的可执行程序（exec族函数可以直接把一个编译好的可执行程序直接加载运行）

有了exec族函数后，我们典型的父子进程程序是这样的：子进程需要运行的程序被单独编写、单独编译连接成一个可执行程序（叫 hello），（项目是一个多进程项目）主程序为父进程，fork 创建了子进程后在子进程中 exec 来执行 hello，达到父子进程分别做不同程序同时（宏观上）运行的效果。

在 **[[frok函数#^3f3cb6|fork]]** 后的子进程中使用 exec 函数族，可以装入和运行其它程序（子进程替换原有进程，和父进程做不同的事）。exec 函数族可以根据指定的文件名或目录名找到可执行文件，并用它来取代原调用进程的数据段、代码段和堆栈段。在执行完后，原调用进程的内容除了进程号外，其它全部被新程序的内容替换了。另外，这里的可执行文件既可以是二进制文件，也可以是Linux下任何可执行脚本文件。

执行exec系统调用，一般都是这样，用 fork 函数新建立一个进程，然后让进程去执行 exec 调用。我们知道，在 fork 建立新进程之后，父进各与子进程共享代码段，但数据空间是分开的，但父进程会把自己数据空间的内容 copy 到子进程中去，还有上下文也会 copy 到子进程中去。而为了提高效率，采用一种 **[[frok函数#3 fork的执行过程：|写时拷贝技术]]** ，而对于 fork 之后执行 exec 后，这种策略能够很好的提高效率，如果一开始就 copy，那么 exec 之后，子进程的数据会被放弃，被新的进程所代替。

# 2. Linux中使用exec函数族的两种情况

当进程认为自己不能再为系统和用户做出任何贡献时，就可以调用任何 exec 函数族让自己重生；

如果一个进程想执行另外一个程序，那么它就可以调用 **fork** 函数新建一个进程，然后调用任何一个 exec 函数使子进程重生；

# 3. exec函数族语法

实际上，在 Linux 中并没有 exec 函数，而是有6个以 exec 开头的函数族，下表列举了exec 函数族的6个成员函数的语法。
```c
所需头文件 #include <unistd.h>

函数类型   执行文件

函数原型   int execl(const char *path, const char *arg, ...)
          int execv(const char *path, char *const argv[])
          int execle(const char *path, const char *arg, ..., char *const envp[])
          int execve(const char *path, char *const argv[], char *const envp[])
          int execlp(const char *file, const char *arg, ...)
          int execvp(const char *file, char *const argv[])

函数返回值成功：   函数：不会返回
                 出错：返回 -1，失败原因记录在 error 中
```

 1. execl 和 execv 这两个函数是最基本的 exec，都可以用来执行一个程序，区别是传参的格式不同。execl 是把参数列表（本质上是多个字符串，必须以NULL结尾）依次排列而成（l其实就是list的缩写），execv 是把参数列表事先放入一个字符串数组中，再把这个字符串数组传给 execv 函数。
2. execlp 和 execvp 这两个函数在上面2个基础上加了p，较上面2个来说，区别是：上面2个执行程序时必须指定可执行程序的全路径（如果 exec 没有找到 path 这个文件则直接报错），而加了 p 的传递的可以是 file（也可以是 path，只不过兼容了 file。加了 p 的这两个函数会首先去找file，如果找到则执行执行，如果没找到则会去环境变量 PATH 所指定的目录下去找，如果找到则执行如果没找到则报错）
3. execle 和 execvpe 这两个函数较基本 exec 来说加了 e，函数的参数列表中也多了一个字符串数组 envp 形参，e 就是 environment 环境变量的意思，和基本版本的 exec 的区别就是：执行可执行程序时会多传一个环境变量的字符串数组给待执行的程序。

这6个函数在**函数名和使用语法的规则**上都有细微的区别，下面就可执行文件查找方式、参数传递方式及环境变量这几个方面进行比较说明。
1. 查找方式：上表中前4个函数的查找方式都是完整的文件目录路径(即绝对路径)，而最后两个函数(也就是以p结尾的两个函数)可以只给出文件名，系统就会自动从环境变量“$PATH”所指出的路径中进行查找。
2. 参数传递方式：有两种方式，一种是逐个列举的方式，另一种是将所有参数整体构造成一个指针数组进行传递。（在这里，字母“l”表示逐个列举的方式，字母“v”表示将所有参数整体构造成指针数组进行传递，然后将该数组的首地址当做参数传递给它，数组中的最后一个指针要求时NULL）
3. 环境变量：exec函数族使用了系统默认的环境变量，也可以传入指定的环境变量。这里以“e”结尾的两个函数就可以在envp[]中指定当前进程所使用的环境变量替换掉该进程继承的所有环境变量。

# 4. PATH环境变量说明

PATH环境变量包含了一张目录表，系统通过PATH环境变量定义的路径搜索执行码，PATH环境变量定义时目录之间需用 **“;”** 分隔，以 **“.”** 表示结束 。
PATH 环境变量定义在用户的 **.profile** 或 **.bash_profile** 中，下面是 PATH 环境变量定义的样例，此 PATH 环境变量指定 **/bin** 、**/usr/bin** 和 **当前目录** 三个目录进行搜索执行码。

PATH=/bin:/usr/bin..

export $PATH

# 5. 进程中的环境变量说明

在 Linux 中，Shell 进程是所有执行码的父进程。当一个执行码执行时，Shell 进程会 fork 子进程然后调用 exec 函数去执行执行码。
Sehll 进程堆栈中存放着该用户下的所有环境变量，使用不带 “e” 的4个函数使执行码重生时，Shell 进程会将所有环境变量复制给生成的新进程；
而使用带 “e” 的两个函数时新进程不继承任何 Shell 进程的环境变量，而由 **envp[ ]** 数组自行设置环境变量。

# 6. exec函数族关系

事实上，这6个函数中真正的系统调用只有 **execve**，其他5个都是库函数，它们最终都会调用execve 这个系统调用，调用关系如下图所示：
![[Pasted image 20220802205108.png]]

# 7. exec调用举例如下

```c
char *const ps_argv[] ={"ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL};

char *const ps_envp[] ={"PATH=/bin:/usr/bin", "TERM=console", NULL};

execl("/bin/ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL);

execv("/bin/ps", ps_argv);

execle("/bin/ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL, ps_envp);

execve("/bin/ps", ps_argv, ps_envp);

execlp("ps", "ps", "-o", "pid,ppid,pgrp,session,tpgid,comm", NULL);

execvp("ps", ps_argv);
```

请注意 exec 函数族形参展开时的前两个参数，第一个参数是带路径的执行码（execlp、execvp函数第一个参数是无路径的，系统会根据 PATH 自动查找然后合成带路径的执行码），第二个是不带路径的执行码，执行码可以是二进制执行码和 Shell 脚本。

# 8. exec函数族使用注意点

在使用exec函数族时，一定要加上错误判断语句。因为exec很容易执行失败，其中最常见的原因有：
找不到文件或路径，此时errno被设置为ENOENT。
数组argv和envp忘记用NULL结束，此时errno被设置为EFAULT。
没有对用可执行文件的运行权限，此时errno被设置为EACCES。

# 9. exec后新进程保持原进程以下特征

环境变量（使用了execle、execve函数则不继承环境变量）；

进程ID和父进程ID；

实际用户ID和实际组ID；

附加组ID；

进程组ID；

会话ID；

控制终端；

当前工作目录；

根目录；

文件权限屏蔽字；

文件锁；

进程信号屏蔽；

未决信号；

资源限制；

tms_utime、tms_stime、tms_cutime以及tms_ustime值。

对打开文件的处理与每个描述符的 exec 关闭标志值有关，进程中每个文件描述符有一个 exec 关闭标志 (FD_CLOEXEC)，若此标志设置，则在执行 exec 时关闭该描述符，否则该描述符仍然打开。除非特地用了 fcntl 设置了该标志，否则系统的默认操作是在 exec 后仍保持这种描述符打开，利用这一点可以实现 I/O 重定向。

# 10. 代码用例测试

exec.c
```c
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <string.h>  
#include <errno.h>  
#include <config.h>  
  
int main(int argc, char *argv[])  
{  
//以NULL结尾的字符串数组的指针，适合包含v的exec函数参数  
char *arg[] = {"ls", "-a", NULL};  
  
/**  
* 创建子进程并调用函数execl  
* execl 中希望接收以逗号分隔的参数列表，并以NULL指针为结束标志  
*/  
if( fork() == 0 )  
{  
// in clild  
printf( "1------------execl------------\n" );  
if( execl( "/bin/ls", "ls","-a", NULL ) == -1 )  
{  
perror( "execl error " );  
exit(1);  
}  
}  
  
/**  
*创建子进程并调用函数execv  
*execv中希望接收一个以NULL结尾的字符串数组的指针  
*/  
if( fork() == 0 )  
{  
// in child  
printf("2------------execv------------\n");  
if( execv( "/bin/ls",arg) < 0)  
{  
perror("execv error ");  
exit(1);  
}  
}  
  
/**  
*创建子进程并调用 execlp  
*execlp中  
*l希望接收以逗号分隔的参数列表，列表以NULL指针作为结束标志  
*p是一个以NULL结尾的字符串数组指针，函数可以DOS的PATH变量查找子程序文件  
*/  
if( fork() == 0 )  
{  
// in clhild  
printf("3------------execlp------------\n");  
if( execlp( "ls", "ls", "-a", NULL ) < 0 )  
{  
perror( "execlp error " );  
exit(1);  
}  
}  
  
/**  
*创建子里程并调用execvp  
*v 望接收到一个以NULL结尾的字符串数组的指针  
*p 是一个以NULL结尾的字符串数组指针，函数可以DOS的PATH变量查找子程序文件  
*/  
if( fork() == 0 )  
{  
printf("4------------execvp------------\n");  
if( execvp( "ls", arg ) < 0 )  
{  
perror( "execvp error " );  
exit( 1 );  
}  
}  
  
/**  
*创建子进程并调用execle  
*l 希望接收以逗号分隔的参数列表，列表以NULL指针作为结束标志  
*e 函数传递指定参数envp，允许改变子进程的环境，无后缀e时，子进程使用当前程序的环境  
*/  
if( fork() == 0 )  
{  
printf("5------------execle------------\n");  
if( execle("/bin/ls", "ls", "-a", NULL, NULL) == -1 )  
{  
perror("execle error ");  
exit(1);  
}  
}  
  
/**  
*创建子进程并调用execve  
* v 希望接收到一个以NULL结尾的字符串数组的指针  
* e 函数传递指定参数envp，允许改变子进程的环境，无后缀e时，子进程使用当前程序的环境  
*/  
if( fork() == 0 )  
{  
printf("6------------execve-----------\n");  
if( execve( "/bin/ls", arg, NULL ) == 0)  
{  
perror("execve error ");  
exit(1);  
}  
}  
return EXIT_SUCCESS;  
}
```

运行结果
```shell
1------------execl------------
2------------execv------------
3------------execlp------------
4------------execvp------------
5------------execle------------
6------------execve-----------
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c
.  ..  exec  exec.c  fork_test  fork_test.c  main  main.c  note.sh  prt  prt.c  son  son.c  test  test.c

```

