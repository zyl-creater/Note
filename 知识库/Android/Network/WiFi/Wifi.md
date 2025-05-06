抓包
tcpdump -p -vv -s 0 -w /data/1.pcap -i wlan0 &  
logcat -c;logcat -s wpa_supplicant > /data/wpa_supplicant.log &



打开wifi更多log
cmd wifi set-verbose-logging enabled