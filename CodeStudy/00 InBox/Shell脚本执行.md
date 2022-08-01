# 1. 如何运行/执行脚本？

```shell
note.sh文件

echo "Hello World !"
```

在保存文件后，我们可以使用bash命令来运行，把我们的文件作为它的参数：

```shell
zhuyuelin@ubuntu:~/zyltest$ bash note.sh
Hello World !
```

实际上，这样来执行脚本是很不方便的。如果不使用**bash**命令作为前缀来执行，会更舒服一些。要让脚本可执行，我们可以使用**chmod**命令：
```shell
zhuyuelin@ubuntu:~/zyltest$ ls -la note.sh 
-rw-rw-r-- 1 zhuyuelin zhuyuelin 22 Aug  1 14:09 note.sh
zhuyuelin@ubuntu:~/zyltest$ chmod +x note.sh 
zhuyuelin@ubuntu:~/zyltest$ ls -la note.sh 
-rwxrwxr-x 1 zhuyuelin zhuyuelin 22 Aug  1 14:09 note.sh
```

**注意** ：ls命令显示了当前文件夹内的文件。通过添加-la键，它会显示更多文件信息。

如我们所见，在**chmod**命令执行前，脚本只有读（r）和写（w）权限。在执行**chmod +x**后，它就获得了执行（x）权限。
现在，我们只需这么来运行：
```shell
zhuyuelin@ubuntu:~/zyltest$ ./note.sh 
Hello World !
```

在脚本名前，我添加了 ./ 组合。.(点）在unix世界中意味着当前位置（当前文件夹），/（斜线）是文件夹分隔符。（在Win系统中，我们使用反斜线 / 表示同样功能）所以，这整个组合的意思是说："从当前文件夹执行note.sh脚本"。如果我用完整路径来运行这个脚本的话，会更加清楚一些：

```shell
zhuyuelin@ubuntu:~/zyltest$ /home/zhuyuelin/zyltest/note.sh 
Hello World !
```

如果所有linux用户都有相同的默认shell，那就万事OK。如果我们只是执行该脚本，默认的用户shell就会用于解析脚本内容并运行命令。不同的shell的语法、内部命令等等有着一丁点不同，所以，为了保证我们的脚本会使用**bash**，我们应该添加 **#!/bin/bash** 到文件首行。这样默认的用户shell将调用 **/bin/bash** ，而只有在那时候，脚本中的命令才会被执行：
```shell
zhuyuelin@ubuntu:~/zyltest$ cat note.sh 
#!/bin/bash
echo "Hello World !"

```

直到现在，我们才100%确信**bash**会用来解析我们的脚本内容。

# 2. 脚本的调用形式

打开终端时系统自动调用：/etc/profile 或 ~/.bashrc
## /etc/profile
此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行，系统的公共环境变量在这里设置
开始自启动的程序，一般也在这里设置
## ~/.bashrc
用户自己的家目录中的.bashrc
登录时会自动调用，打开任意终端时也会自动调用
这个文件一般设置与个人用户有关的环境变量，如交叉编译器的路径等等
用户手动调用：用户实现的脚本
```shell
zhuyuelin@ubuntu:~$ ls -a
.                      .cache                  .repopickle_.gitconfig
..                     .git                    scripts
amlogic                .gitconfig              sdmc_patch
amlogic_product_q_1.0  git_modified_files.txt  .ssh
.bash_history          .profile                .viminfo
.bash_logout           .repoconfig             workspace
.bashrc                .repo_.gitconfig.json   zyltest
```

# 3. Shell脚本存在三种不同的执行方式

## ./xxx.sh
会先按照文件中 **#!** 指定的解析器解析
如果 **#!** 指定的解析器不存在，才会使用系统默认的解析器
## bash xxx.sh
指明先用 bash 解析器解析
如果 bash 不存在，才会使用默认解析器
## . xxx.sh
直接使用默认解析器，并不会执行第一行的 #! 指定的解析器，但第一行还是要写的

# 4. 三种执行情况

打开终端后就会有一个解析器，我们称之为**当前解释器**
在我们指定解释器的时候 ( 使用 ./xxx.sh 或 bash xxx.sh ) 时会创建一个子 shell 解析脚本

![[Pasted image 20220801110923.png]]

# 注意：在 win 下写脚本，在 linux 下执行时

CR（ 回车，ASCII 13，\\r ）LF（ 换行，ASCII 10，\\n ）
由于在 Win 中广泛使用 CRLF 来标识一行的结束，而在 Linux / UNIX 系统中只有 LF 换行符。
需要将 Win 文件转换成 UNIX 文件

![[Pasted image 20220801112411.png]]

![[Pasted image 20220801112434.png]]

