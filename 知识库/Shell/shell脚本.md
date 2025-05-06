shell脚本就是一种专门使用shell编写的脚本程序，它虽然没有C++、Java、Python等一系列高级语言功能强大，但是在服务器运维领域以及嵌入式开发领域，shell脚本具有举足轻重的地位。

shell脚本编程如同其他编程语言的一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以运行了，而linux下的shell种类众多，常用的用：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）
- … …

在诸多linux发行版系统中，最常用的就是Bash，就是Bourne Again Shell，因为其能工提供环境变量以配置用户shell环境，支持历史记录、内置算数功能、支持通配符表达式等高效性能，将linux常用命令进行的简化，被广泛应用于Debian系列的linux发行版中。

## shell注释
### 单行注释
和python注释相同，以`#`号开头作为单行注释
```shell
# 这是一个注释
# author：ohuohuoo
# date：`date`
```
### 多行注释
如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，可以将其定义为一个花括号的注释函数，也可以用多行注释
```shell
:<<EOF
注释内容...
注释内容...
注释内容...
EOF
 
# EOF可以换成其他符号
:<<E！
注释内容...
注释内容...
注释内容...
！
```

