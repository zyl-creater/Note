通用蓝牙hci log的抓取(确保data/misc/bluedroid/btsnoop_hci.cfa文件大小 )  
　　　确保时间更新到网络时间  
　　　串口输入setprop persist.bluetooth.btsnoopenable true  
　　　并且setting 的开发者模式中打开 Enable Bluetooth HCI snoop log 选项  
　　　修改如下配置文件：  
　　　vi /system/etc/bluetooth/bt_stack.conf 所有的 2 为 6  
　　　vi /vendor/etc/bluetooth/rtkbt.conf 修改 RtkBtsnoopDump=true  
　　　设置btsnoop的路径  
　　　setprop persist.bluetooth.btsnooppath /data/misc/bluedroid/btsnoop_hci.cfa  
　　　开关蓝牙一次(或者重启盒子也可以)  
　　　service call bluetooth_manager 8 关掉BT  
　　　service call bluetooth_manager 6 打开BT