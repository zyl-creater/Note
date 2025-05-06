S905Y4/X4 DDR展频：  
查询寄存器  
echo 0xfe036c00 6 > /sys/kernel/debug/aml_reg/dump
cat /sys/kernel/debug/aml_reg/dump
  
开展频1000ppm  
echo 0xfe036c08 0x0100120 >/sys/kernel/debug/aml_reg/paddr  
  
开展频2000ppm  
echo 0xfe036c08 0x0100140 >/sys/kernel/debug/aml_reg/paddr  
  
开展频3000ppm  
echo 0xfe036c08 0x0100160 >/sys/kernel/debug/aml_reg/paddr