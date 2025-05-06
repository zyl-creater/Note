系统执行ls -al /dev/block/by-name 获得分区路径与分区别名映射:






1.把打印放开，重新抓下log，看看是否有更详细的打印：

setenv loglevel 8;setenv initargs $initargs printk.devkmsg=on;saveenv; reset

2.目前dm-verity校验失败，可能需要dump下异常板子里面的镜像

需要dump出如下分区：

metadata/vbmeta/super

3.同时需要dump下同版本软件，正常启动的板子的metadata/vbmeta/super分区镜像对比



cat /proc/partitions | grep mmcblk0