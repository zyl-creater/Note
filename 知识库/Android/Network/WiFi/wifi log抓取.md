==========================================================================wifi&bt log抓取===========================================================================
如下只是基本的log抓取，如遇到特殊情况，请依据模组厂具体指令来获取信息


展锐 uwe5621
注：需要抓取的基本log为kernel，logcat，cp2的log，
	uboot下面：
		uboot下打开内核打印：setenv loglevel 8;saveenv;res
　　	selinux关闭：setenv EnableSelinux permissive;saveenv;res 
　　系统下：
　　　  echo 8   >  /proc/sys/kernel/printk  系统下打开内核打印
  		echo 0   >  /proc/sys/kernel/printk   系统下关闭内核打印
		setenforce 0 --->关闭selinux
	展锐的wifi log抓取提高log等级通过配置文件wifi_dbg.ini设置WiFi driver log等级，将配置wifi_dbg.ini放到“/data/misc/wifi/”或 “/vendor/etc/wifi” 或 “/etc“目录下，
	重新打开wifi后生效，该方式wifi重新开关或系统重启后仍然有效。
		wifi_dbg.ini  内容（注意为linux格式）
		[DEBUG]
		log_level=3
			
	kernel log：dmesg -c > /data/kernel.txt;while true; do dmesg -c >> /data/kernel.txt;sleep 0.1;done&
	logcat：logcat -c; logcat -v threadtime > /data/logcat.txt &
	
	wpa_supplicant 开debug level:
		wpa_cli log_level DEBUG
		wpa_cli log_level 查看是不是DEBUG
		PS：此命令当次开wifi有效。
	
	cp2 log的抓取
		串口输入echo "at+armlog=1\r" > /proc/mdbg/at_cmd--->表示打开cp2 log,
		串口输入echo "at+armlog=0\r" > /proc/mdbg/at_cmd--->表示关闭cp2 log,
		打开后会在data目录下生成unisoc_cp2log_0.txt，确认文件大小有变化即可
		如果怕cp2 log太大可以自定义路径，按照如下方式创建一个名为unisoc_cp2log_config.txt的配置文件
		具体内容和语句含义如下：
		wcn_cp2_log_limit_size=500M;---》每个cp2 log的文件大小
		wcn_cp2_file_max_num=2;---》可以写的cp2 log 个数
		wcn_cp2_file_over_cover_old=true;是否覆盖老的cp2 log
		wcn_cp2_log_path=“/data/unisoc_dbg";--》自定义路径
	


RTK系列wifi抓取
	前期准备：
　　　uboot下面：
		uboot下打开内核打印：setenv loglevel 8;saveenv;res
　　　	selinux关闭：setenv EnableSelinux permissive;saveenv;res 
　　　系统下：
　　　  echo 8   >  /proc/sys/kernel/printk  系统下打开内核打印
  		echo 0   >  /proc/sys/kernel/printk   系统下关闭内核打印  
		setenforce 0 --->关闭selinux
		进入系统后输入：echo 4 >/proc/net/$(ls /proc/net/|grep rtl)/log_level
		wifi吞吐量问题需要输入：echo dbg 30 1 > /proc/net/rtl88x2es/wlan0/odm/cmd
		
	log的抓取：
		内核的log：dmesg -c > /data/kernel.txt;while true; do dmesg -c >> /data/kernel.txt;sleep 0.1;done&
		logcat：logcat -c; logcat -v threadtime > /data/logcat.txt &
		蓝牙共存问题需要串口输入：while true; do cat /proc/net/$(ls /proc/net/|grep rtl)/wlan0/btcoex; sleep 2; done;(可以将结果保存为txt文件)
		wpa_supplicant 开debug level:
			wpa_cli log_level DEBUG
			wpa_cli log_level 查看是不是DEBUG
			PS：此命令当次开wifi有效。
			
	信号强度log抓取：
		while true; do cat /proc/net/$(ls /proc/net/|grep rtl)/wlan0/rx_signal; sleep 5; done;
　　连续的rssi打印：
　　	echo 1 > /proc/net/rtl88x2cs/wlan0/linked_info_dump
		

3、博通系列wifi log抓取
	前期准备：
　　　uboot下面：
		uboot下打开内核打印：setenv loglevel 8;saveenv;res
　　　	selinux关闭：setenv EnableSelinux permissive;saveenv;res 
　　　系统下：
　　　   echo 8   >  /proc/sys/kernel/printk  系统下打开内核打印
  		 echo 0   >  /proc/sys/kernel/printk   系统下关闭内核打印
		 setenforce 0 --->关闭selinux
 
	  对应的config.txt里面加：
		dhd_msg_level=0x801
		dhd_console_ms=20
		android_msg_level=0xf
		wl_dbg_level=0x9
		dump_msg_level=0xf
		
		wpa_supplicant 开debug level:
		wpa_cli log_level DEBUG
		wpa_cli log_level 查看是不是DEBUG 
	
	log的抓取：
	kernellog：dmesg -c > /data/kernel.txt;while true; do dmesg -c >> /data/kernel.txt;sleep 0.1;done&
	logcat：logcat -c; logcat -v threadtime > /data/logcat.txt &
