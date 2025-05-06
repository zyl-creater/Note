echo performance > /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor;  
echo 1000000 > /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq; 
/*设置CPU频率，如1200000=1200MHz*/  

cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq;
/*查看设置CPU频率*/  
注意：定频的时候可以直接关闭温控  

压测：  
./data/stressapptest -s 86400 -i 2 -m 2 -M 300 -W --pause_delay 180 --pause_duration 10 -l /sdcard/stressapptest_memory.log &  
  
关闭温控  
echo disabled > /sys/class/thermal/thermal_zone0/mode  
打开温控  
echo enabled > /sys/class/thermal/thermal_zone0/mode  
查看温度门限(第一个值为降频的温度)：  
cat /sys/devices/virtual/thermal/thermal_zone*/trip_point_*_temp
