# 一、Jenkins当前状态

## 1.1 ATV项目情况

Jenkins当前一共有14个项目正在运行，14个都项目运行基本正常

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/dbb930b6-f602-49f3-9517-e73a71eed9d1.png)

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|项目|机型|负责人|备注|创建状态|运行状态|
|PT|DV8985|博林|新项目|已创建|运行正常|
|A1_Bulgaria|DV9161|博林|新项目|已创建|运行正常|
|Entel|DV8235|凯能|Q2S|已创建|运行正常|
|MNC|DV9187|海城|新项目|已创建|运行正常|
|Telia-SE|DV8919|海城|Q2S|已创建|运行正常|
|Telered|DV9061|龚靖|新项目|已创建|运行正常|
|Vectra|DV8519|龚靖|Q2S|已创建|运行正常|
|PT|DV8555|博林|Q2S|已创建|运行正常|
|Altice|DV8555|龚靖|Q2S|已创建|运行正常|
|NOS|DV9161|博林|新项目|已创建|运行正常|
|DT|DV6067Y|海城|P2Q2S|已创建|运行正常|
|UNITEL|DV9187|文燕|新项目|已创建|运行正常|
|MT|DV8519|海城|Q2S|已创建|运行正常|
|Telia-LT|DV8919|海城|R2S|已创建|运行正常|

各个项目成功运行次数

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/60bfe1b2-716e-4e18-8d71-3e7c7ab9c9ee.png)

## 1.2 HyBrid项目情况

Android S HyBrid版本项目也在适配上线中

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|HyBrid项目|   |   |   |   |   |
|项目|机型|负责人|备注|创建状态|运行状态|
|ECONET|DV9157-S2|钟映浩|HyBrid项目|已创建|运行正常|
|PLAY|DV8945-T2C|刘席|HyBrid项目|未创建||
|masmovil||李连捷|HyBrid项目|未创建||

# 二、遇到的问题

jenkins项目运行一个半月内，一共遇到问题15个，大致是四类问题：构建参数因素，服务器环境因素，打包工程因素，基线因素

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/bd398ec4-bed9-4d0a-b2af-87d3af9678b7.png)

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/0188ef9d-acdb-4799-b24a-89b3a5dbbb5b.png)

## 2.1 构建参数因素：

1. Telia-SE
    
    1. 上传打包材料到SVN失败，分析原因是svn的路径填写有问题
        
    2. ![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/6be3c3a5-fca8-440f-933b-53604caf3684.png)
        
2. PT
    
    1. 打包时，填入的材料没有生效，jenkins使用了以前的旧材料，经排查是在打包的时候，将材料填入了custom，但是选择的参数是user，导致jenkins使用user的材料进行了打包
        

## 2.2 服务器环境因素：

1. Telered
    
    1. 在第一次sdk_build时，出现了服务器环境导致的有文件被服务器锁死，无法改动，这个现象在25，24服务器上也出现过
        
    2. ![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/2b956ae0-67cb-4257-8cbe-42b6ed33bb7a.png)
        

2. Telered，vectra
    
    1. 在编译bootloader的时候，相同的代码在25服务器上可以编译通过，在27服务器上编译不通过，后续排查原因，是一个打印输出中unsigned long 使用 %x参数导致的（补充解决方法）
        
    2. ![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/53f6b42c-52c3-4478-b031-0b2714f92657.png)![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/186c1b80-923f-4647-a50c-860aa96be052.png)
        
3. 邮件发送失败问题
    
    1. post信息发邮件失败。正常编译结束，post信息导致结果失败
        
4. 服务器空间不足问题
    
    2. vector，entel，Telered都存在编译空间和打包空间不足的问题，经排查是27服务器的tmp缓存用完的原因，问李阿勇只有重启服务器才能解决，重启后问题得到解决
        
    3. ![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/f9b07b81-5f09-4fc0-a5bf-9afdd3494c6d.png)
        

## 2.3 打包工程因素：

1. Entel
    
    1. 打包存在问题，原因是打包工程在本地修改后没有上传到svn
        
2. 状态集参数获取失败问题
    
    2. entel，Telered项目post参数问题，原因是打包工程中安全补丁和指纹存在用#号注释的同名属性，导致获取失败
        
3. MNC
    
    1. 打包失败问题，打包工程错误配置，服务器上工程和本地工程不一致
        

## 2.4 基线因素：

1. vectra 项目存在autobuild失败问题，排查是蓝天新推的版本回退在jenkins上没有适配好，已由蓝天修复
    
2. 状态集信息中获取amlogic_s_stb 分支有问题，排查原因是蓝天的版本回退会多切一个0106的分支在本地，获取信息时会产生错误，已经进行优化
    
3. A1 Bul编译失败问题，排查原因是伟明那边有推基线上改动，修复后已无问题
    
4. PT 项目生成打包材料问题，排查原因是找不到diff文件报错，修复后已无问题
    
    1. ![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/d846ad33-7e1a-4483-b49d-023a7c33e967.png)
        

# 三、建议改进实现情况

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|Jenkins问题建议反馈表|   |   |   |   |   |
|序号|建议或问题|反馈者|处理进展|状态|处理人|
|1|常用打包材料的配置，其他人没有权限可以配置，只能每次打包的时候去填写，操作不方便|杨海城|已释放权限给编译人员|close|袁帅鹏|
|2|打包时间有点长，最近打包有16分钟左右|龚婧|主要是从svn下载以及上传svn耗时，现已优化到11min|close|袁帅鹏|
|3|希望有删除软件包的权限，如果有多个包名一样，比较难区分哪个需要用|龚婧|已释放删除某次构建权限给编译人员，可以删除本次构建来删除软件包|close|袁帅鹏|
|4|Jenkins服务器偶尔会出现无法访问的问题|杨海城|已处理，由于最近服务器不稳定，导致服务器重启后jenkins服务没有开机自启导致|close|袁帅鹏|
|5|状态集可以增加user/userdebug/eng版本的显示，然后将部分常用信息和不常用的分开显示|杨海城|编译类型的显示已处理，常用信息和不常用信息分类处理中|close|袁帅鹏|
|6|使用custom模式填入打包材料路径，但是实际打包的却取的是其他材料|杨海城|使用错误的构建参数|close|朱岳霖|
|7|外网安全性问题讨论|叶飞||open|蒋瑞锋|
|8|创建开机启动jenkins服务|袁帅鹏||open|蒋瑞锋|
|9|状态集无显示问题|龙贵平||close|蒋瑞锋|
|10|邮件虚拟发送人|袁帅鹏||close|蒋瑞锋|
|11|编译标记增加eng|杨海城||open|蒋瑞锋|
|12|时间显示不正确问题|蒋瑞锋||close|蒋瑞锋|
|13|在Jenkins页面上显示重复apk报错信息|杨海城||||
|14||||||

在线查看代码 web 形式 opengrok

![](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/3BMqYZXEL8vNOwZL/img/38e2c607-f7ff-4d48-ab32-4b14c5ee8f38.png)