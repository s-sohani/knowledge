

توی انسیبل، دوتا فایل با دقیقا اسامی که توی عکس میبینید ذیل فولدر deployment/ansible توی پروژه بسازید و پسوورد ssh و vault رو توی اونها قرار بدید.  
  

از بعدش دیگه دستورات انسیبل رو به شکلی شبیه زیر وارد کنید:  
ansible-playbook -i inventories/dev/ playbooks/clickhouse.yml --tags clickhouse  
  
و دیگه لازم نیست پسوورد هیچی رو بزنید.  
  
همچنان حالت قبلی که میزدیم  
--ask-pass --ask-become-pass --ask-vault-pass  
برقراره اگر که به هر دلیلی دوست نداشتید اون دوتا فایل رو توی لوکال خودتون بگذارید.