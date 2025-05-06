### 一、pm help
```
# pm help

包管理器（package）命令：

help
    打印此帮助文本。

path [--user USER_ID] PACKAGE
    打印给定包的 .apk 的路径。

dump PACKAGE
    打印与给定 PACKAGE 关联的各种系统状态。

list features
    打印系统的所有功能。

has-feature FEATURE_NAME [version]
    当系统有 FEATURE_NAME 时打印 true 并返回退出状态 0，否则打印 false 并返回退出状态 1

list instrumentation [-f] [TARGET-PACKAGE]
    打印所有测试包；可选地只针对目标包
    选项：
        -f：转储包含测试包的.apk文件名

list libraries
    打印所有系统库。

list packages [-f] [-d] [-e] [-s] [-3] [-i] [-l] [-u] [-U] [--show-versioncode] [--apex-only] [--uid UID] [--user USER_ID] [FILTER]
打印所有包； 可选地仅那些其名称包含 FILTER 中的文本的包。 选项是：
    -f：查看它们的关联文件
    -a：所有已知的包（但不包括 APEXes）
    -d：过滤以仅显示禁用的包
    -e：过滤以仅显示启用的包
    -s: 过滤只显示系统包
    -3：过滤以仅显示第三方包
    -i：查看软件包的安装程序
    -l：忽略（用于与旧版本兼容）
    -U：同时显示包的 UID
    -u：还包括未安装的包
    --show-versioncode: 同时显示版本代码
    --apex-only: 只显示 APEX 包
    --uid UID：过滤以仅显示具有给定 UID 的包
    --user USER_ID：只列出属于给定用户的包

list permission-groups
    打印所有已知的权限组。

list permissions [-g] [-f] [-d] [-u] [GROUP]
    打印所有已知权限； 可选地只在 GROUP 中。 选项是：
        -g：按组组织
        -f：打印所有信息
        -s：简短摘要
        -d：只列出危险权限
        -u：仅列出用户将看到的权限

resolve-activity [--brief] [--components] [--query-flags FLAGS] [--user USER_ID] INTENT
    打印处理给定 INTENT 的 activity。

query-activities [--brief] [--components] [--query-flags FLAGS] [--user USER_ID] INTENT
    打印可以处理给定 INTENT 的所有活动。

query-services [--brief] [--components] [--query-flags FLAGS] [--user USER_ID] INTENT
    打印可以处理给定 INTENT 的所有服务。

query-receivers [--brief] [--components] [--query-flags FLAGS] [--user USER_ID] INTENT
    打印可以处理给定 INTENT 的所有广播接收器。

install [-lrtsfdgw] [-i PACKAGE] [--user USER_ID|all|current]
       [-p INHERIT_PACKAGE] [--install-location 0/1/2]
       [--install-reason 0/1/2/3/4] [--originating-uri URI]
       [--referrer URI] [--abi ABI_NAME] [--force-sdk]
       [--preload] [--instantapp] [--full] [--dont-kill]
       [--enable-rollback]
       [--force-uuid internal|UUID] [--pkg PACKAGE] [-S BYTES] [--apex]
       [PATH|-]
    安装应用程序。 必须提供要安装的 apk 数据，可以是文件路径，也可以是“-”以从标准输入读取。 选项是：
    -l：前向锁应用
    -R：不允许替换现有应用程序
    -t：允许测试包
    -i：指定拥有应用程序的安装程序的包名称
    -s：在sdcard上安装应用程序
    -f：在内部闪存上安装应用程序
    -d：允许版本代码降级（仅限可调试包）
    -p：部分应用程序安装（在现有 pkg 之上的新拆分）
    -g：授予所有运行时权限
    -S：包的大小（以字节为单位），标准输入需要
    --user：安装在给定用户下。
    --dont-kill: 安装一个新的 feature split，不要杀死正在运行的app
    --restrict-permissions：不要在安装时将限制权限列入白名单
    --originating-uri: 设置下载应用程序的 URI
    --referrer：设置启动应用程序安装的 URI
    --pkg：指定正在安装的应用程序的预期包名称
    --abi: 覆盖平台的默认 ABI
    --instantapp：使应用程序作为临时安装应用程序安装
    --full：使应用程序安装为非临时完整应用程序
    --install-location：强制安装位置：
       0=自动，1=仅限内部，2=首选外部
    --install-reason：指示安装应用程序的原因：
       0=未知，1=管理政策，2=设备恢复，
       3=设备设置，4=用户请求
    --force-uuid：强制安装到具有给定 UUID 的磁盘卷
    --apex：安装 .apex 文件，而不是 .apk

install-create [-lrtsfdg] [-i PACKAGE] [--user USER_ID|all|current]
       [-p INHERIT_PACKAGE] [--install-location 0/1/2]
       [--install-reason 0/1/2/3/4] [--originating-uri URI]
       [--referrer URI] [--abi ABI_NAME] [--force-sdk]
       [--preload] [--instantapp] [--full] [--dont-kill]
       [--force-uuid internal|UUID] [--pkg PACKAGE] [--apex] [-S BYTES]
       [--multi-package] [--staged]
    类似于“安装”，但启动安装会话。 使用“install-write”将数据推送到会话中，然后“install-commit”完成。

install-write [-S BYTES] SESSION_ID SPLIT_NAME [PATH|-]
    将 apk 写入给定的安装会话。 如果路径为“-”，则将从标准输入读取数据。 选项是：
       -S：包的大小（以字节为单位），标准输入需要

install-add-session MULTI_PACKAGE_SESSION_ID CHILD_SESSION_IDs
    向多包会话添加一个或多个会话 ID。

install-commit SESSION_ID
    提交给定的活动安装会话，安装应用程序。

install-abandon SESSION_ID
    删除给定的活动安装会话。

set-install-location LOCATION
    更改默认安装位置。 注意这仅用于调试； 使用它会导致应用程序中断和其他不良行为。 LOCATION 是以下之一：
        0 [auto]：让系统决定最佳位置
        1 [internal]：安装在内部设备存储上
        2 [external]：安装在外部媒体上

get-install-location
    返回当前安装位置：0、1 或 2，根据 set-install-location。

move-package PACKAGE [internal|UUID]

move-primary-storage [internal|UUID]

pm uninstall [-k] [--user USER_ID] [--versionCode VERSION_CODE] PACKAGE [SPLIT]
    从系统中删除给定的包名称。 如果未指定 SPLIT 名称，则可能会删除整个应用程序，否则将仅删除给定应用程序的拆分。 选项是：
       -k：删除包后保留数据和缓存目录。
       --user：从给定用户中删除应用程序。
       --versionCode：仅当应用程序具有给定的版本代码时才卸载。

clear [--user USER_ID] PACKAGE
    删除与包关联的所有数据。

enable [--user USER_ID] PACKAGE_OR_COMPONENT

disable [--user USER_ID] PACKAGE_OR_COMPONENT

disable-user [--user USER_ID] PACKAGE_OR_COMPONENT

disable-until-used [--user USER_ID] PACKAGE_OR_COMPONENT

default-state [--user USER_ID] PACKAGE_OR_COMPONENT

    这些命令更改给定包或组件（写为"package/class"）的启用状态。

hide [--user USER_ID] PACKAGE_OR_COMPONENT

unhide [--user USER_ID] PACKAGE_OR_COMPONENT

suspend [--user USER_ID] TARGET-PACKAGE
    暂停指定的包（作为用户）。

unsuspend [--user USER_ID] TARGET-PACKAGE
    取消挂起指定的包（作为用户）。

grant [--user USER_ID] PACKAGE PERMISSION
revoke [--user USER_ID] PACKAGE PERMISSION
    这些命令可以授予或撤销对应用程序的权限。 权限必须在应用程序的清单中声明为已使用，是运行时权限（保护级别危险），并且应用程序的目标 SDK 
    大于 Lollipop MR1。

reset-permissions
    将所有运行时权限恢复为默认状态。

set-permission-enforced PERMISSION [true|false]

get-privapp-permissions TARGET-PACKAGE
    打印包的所有特权权限。

get-privapp-deny-permissions TARGET-PACKAGE
    打印包被拒绝的所有特权权限。

get-oem-permissions TARGET-PACKAGE
    打印包的所有 OEM 权限。

set-app-link [--user USER_ID] PACKAGE {always|ask|never|undefined}
get-app-link [--user USER_ID] PACKAGE

trim-caches DESIRED_FREE_SPACE [internal|UUID]
    修剪缓存文件以达到给定的可用空间。

list users
    列出当前用户。

create-user [--profileOf USER_ID] [--managed] [--restricted] [--ephemeral] [--guest] [--pre-create-only] USER_NAME
    使用给定的 USER_NAME 创建一个新用户，打印该用户的新用户标识符。

remove-user USER_ID
    删除具有给定 USER_IDENTIFIER 的用户，删除与该用户关联的所有数据

set-user-restriction [--user USER_ID] RESTRICTION VALUE

get-max-users

get-max-running-users

compile [-m MODE | -r REASON] [-f] [-c] [--split SPLIT_NAME] [--reset] [--check-prof (true | false)] (-a | TARGET-PACKAGE)

如果为“-a”，则触发 TARGET-PACKAGE 或所有包的编译。 选项是：
    -a：编译所有包
    -c：编译前清空profile数据
    -f：即使不需要也强制编译
    -m：选择编译模式。MODE 是 dex2oat 编译器过滤器之一：
        assume-verified
        extract
        verify
        quicken
        space-profile
        space
        speed-profile
        speed
        everything
   -r：选择编译原因，原因是以下之一：
        first-boot
        boot
        install
        bg-dexopt
        ab-ota
        inactive
        shared
    --reset：将包恢复到安装后的状态
    --check-prof (true | false): 做 dexopt 时查看配置文件？
    --secondary-dex: 编译app二级dex文件
    --split SPLIT：只编译给定的拆分名称
    --compile-layouts：编译布局资源以加快通inflation

force-dex-opt PACKAGE
    强制立即执行给定包的 dex opt。

bg-dexopt-job
    立即执行后台优化。 请注意，该命令仅运行后台优化器逻辑。 它可能与实际作业重叠，但作业调度程序将无法取消它。 即使设备不处于空闲维护模式，它也会运行。

reconcile-secondary-dex-files TARGET-PACKAGE
    将包的辅助 dex 文件与生成的 oat 文件进行协调。

dump-profiles TARGET-PACKAGE
     将方法/类配置文件转储到 /data/misc/profman/TARGET-PACKAGE.txt

snapshot-profile TARGET-PACKAGE [--code-path path]
    快照软件包配置文件到 /data/misc/profman/TARGET-PACKAGE[-code-path].prof。如果 TARGET-PACKAGE=android 它将拍摄启动映像的快照

set-home-activity [--user USER_ID] TARGET-COMPONENT
    设置默认home activity（又名launcher）。TARGET-COMPONENT 可以是包名称 (com.package.my) 或完整组件 (com.package.my/component.name)。 然而，只有包名
    很重要：实际使用的组件将自动从包中确定。

set-installer PACKAGE INSTALLER
    设置installer包名称

get-instantapp-resolver
    返回作为当前即时应用程序安装程序的组件的名称。

set-harmful-app-warning [--user <USER_ID>] <PACKAGE> [<WARNING>]
    使用给定的警告消息将应用程序标记为有害。

get-harmful-app-warning [--user <USER_ID>] <PACKAGE>
    返回给定应用程序的有害应用程序警告消息（如果存在）

uninstall-system-updates
    删除对所有系统应用程序的更新并回退到它们的 /system 版本。

get-moduleinfo [--all | --installed] [module-name]
    显示模块信息。 如果指定了模块名称，则仅显示该信息。默认情况下，不带任何参数仅显示已安装的模块。
    --all: 显示所有模块信息
    --installed: 只显示已安装的模块

<INTENT> specifications include these flags and arguments:
    [-a <ACTION>] [-d <DATA_URI>] [-t <MIME_TYPE>] [-i <IDENTIFIER>]
    [-c <CATEGORY> [-c <CATEGORY>] ...]
    [-n <COMPONENT_NAME>]
    [-e|--es <EXTRA_KEY> <EXTRA_STRING_VALUE> ...]
    [--esn <EXTRA_KEY> ...]
    [--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE> ...]
    [--ei <EXTRA_KEY> <EXTRA_INT_VALUE> ...]
    [--el <EXTRA_KEY> <EXTRA_LONG_VALUE> ...]
    [--ef <EXTRA_KEY> <EXTRA_FLOAT_VALUE> ...]
    [--eu <EXTRA_KEY> <EXTRA_URI_VALUE> ...]
    [--ecn <EXTRA_KEY> <EXTRA_COMPONENT_NAME_VALUE>]
    [--eia <EXTRA_KEY> <EXTRA_INT_VALUE>[,<EXTRA_INT_VALUE...]]
        (mutiple extras passed as Integer[])
    [--eial <EXTRA_KEY> <EXTRA_INT_VALUE>[,<EXTRA_INT_VALUE...]]
        (mutiple extras passed as List<Integer>)
    [--ela <EXTRA_KEY> <EXTRA_LONG_VALUE>[,<EXTRA_LONG_VALUE...]]
        (mutiple extras passed as Long[])
    [--elal <EXTRA_KEY> <EXTRA_LONG_VALUE>[,<EXTRA_LONG_VALUE...]]
        (mutiple extras passed as List<Long>)
    [--efa <EXTRA_KEY> <EXTRA_FLOAT_VALUE>[,<EXTRA_FLOAT_VALUE...]]
        (mutiple extras passed as Float[])
    [--efal <EXTRA_KEY> <EXTRA_FLOAT_VALUE>[,<EXTRA_FLOAT_VALUE...]]
        (mutiple extras passed as List<Float>)
    [--esa <EXTRA_KEY> <EXTRA_STRING_VALUE>[,<EXTRA_STRING_VALUE...]]
        (mutiple extras passed as String[]; to embed a comma into a string,
         escape it using "\,")
    [--esal <EXTRA_KEY> <EXTRA_STRING_VALUE>[,<EXTRA_STRING_VALUE...]]
        (mutiple extras passed as List<String>; to embed a comma into a string,
         escape it using "\,")
    [-f <FLAG>]
    [--grant-read-uri-permission] [--grant-write-uri-permission]
    [--grant-persistable-uri-permission] [--grant-prefix-uri-permission]
    [--debug-log-resolution] [--exclude-stopped-packages]
    [--include-stopped-packages]
    [--activity-brought-to-front] [--activity-clear-top]
    [--activity-clear-when-task-reset] [--activity-exclude-from-recents]
    [--activity-launched-from-history] [--activity-multiple-task]
    [--activity-no-animation] [--activity-no-history]
    [--activity-no-user-action] [--activity-previous-is-top]
    [--activity-reorder-to-front] [--activity-reset-task-if-needed]
    [--activity-single-top] [--activity-clear-task]
    [--activity-task-on-home] [--activity-match-external]
    [--receiver-registered-only] [--receiver-replace-pending]
    [--receiver-foreground] [--receiver-no-abort]
    [--receiver-include-background]
    [--selector]
    [<URI> | <PACKAGE> | <COMPONENT>]
```

### **二、pm命令梳理**

#### 1. 命令列表

用法：$ pm  < command >

| 命令                               | 功能          | 实现方法                                  |
| -------------------------------- | ----------- | ------------------------------------- |
| list packages                    | 列举app包信息    | PMS.getInstalledPackages              |
| install `[options`] `<PATH`>     | 安装应用        | PMS.installPackageAsUser              |
| uninstall `[options`]`<package`> | 卸载应用        | IPackageInstaller.uninstall           |
| enable `<包名或组件名`>                | enable      | PMS.setEnabledSetting                 |
| disable `<包名或组件名`>               | disable     | PMS.setEnabledSetting                 |
| hide `<package`>                 | 隐藏应用        | PMS.setApplicationHiddenSettingAsUser |
| unhide `<package`>               | 显示应用        | PMS.setApplicationHiddenSettingAsUser |
| get-install-location             | 获取安装位置      | PMS.getInstallLocation                |
| set-install-location             | 设置安装位置      | PMS.setInstallLocation                |
| path `<package`>                 | 查看App路径     | PMS.getPackageInfo                    |
| clear `<package`>                | 清空App数据     | AMS.clearApplicationUserData          |
| get-max-users                    | 最大用户数       | UserManager.getMaxSupportedUsers      |
| force-dex-opt `<package`>        | dex优化       | PMS.forceDexOpt                       |
| dump `<package`>                 | dump信息      | AM.dumpPackageStateStatic             |
| trim-caches `<目标size`>           | 紧缩cache目标大小 | PMS.freeStorageAndNotify              |

pm命令实的实现方式在 Pm.java，最后大多数都是调用 `PackageManagerService` 相应的方法来完成的。disbale 之后，在桌面和应用程序列表里边都看到不该app。

#### 2. pm list packages

(1) 查看所有的 package:
$ list packages [ options ] < FILTER >

[ options ]参数：
-f: 显示包名所关联的文件；  
-d: 只显示disabled包名；  
-e: 只显示enabled包名；  
-s: 只显示系统包名；  
-3: 只显示第3方应用的包名；  
-i: 包名所相应的installer;  
-u: 包含uninstalled包名.

规律： disabled + enabled = 总应用个数； 系统 + 第三方 = 总应用个数。

(2) 比如：查看第3方应用：
$ pm list packages -3

(3) 又比如，查看已经被禁用的包名。（国内的厂商一般把google的服务禁用了）
$ pm list packages -d

(4) < FILTER >参数：当 FILTER 为不为空时，则只会输出包名带有 FILTER 字段的应用；当 FILTER 为空时，则默认显示所有满足条件的应用。
查看包名带google字段的包名:
$ pm list packages google

#### 3. pm install

安装应用:

$ pm install [options] < PATH >

其中[options]参数：

-r: 覆盖安装已存在Apk，并保持原有数据；  
-d: 运行安装低版本Apk;  
-t: 运行安装测试Apk  
-i : 指定Apk的安装器；  
-s: 安装apk到共享快存储，比如sdcard;  
-f: 安装apk到内部系统内存；  
-l: 安装过程，持有转发锁  
-g: 准许Apk manifest中的所有权限；

PATH参数：

该参数是必须的，是指需要安装的apk所在的路径。

#### 4. 其他参数

pm list users //查看当前手机用户

pm list libraries //查看当前设备所支持的库

pm list features //查看系统所有的features

pm list instrumentation //所有测试包的信息

pm list permission-groups //查看所有的权限组

pm list permissions [options] < group > 查看权限

-g: 以组形式组织；  
-f: 打印所有信息；  
-s: 简要信息；  
-d: 只列举危险权限；  
-u: 只列举用户可见的权限。

4. 例子1：杀掉前台App进程

dumpsys window | grep mCurrentFocus //可以先找到包名
pm disable-user <包名> //杀死进程，pm path <包名> 查看包的存放路径，然后重命名包名也可以达到不启动的目的

disable后必须要 pm enable < package > 后此package才能正常运行，否则即使是重启机器也无法正常运行。