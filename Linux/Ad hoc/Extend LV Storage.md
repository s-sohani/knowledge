change partition size  
  
growpart /dev/sda 3  
  
lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv