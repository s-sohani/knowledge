```
[defaults]  
forks = 12  
  
transport = ssh  
host_key_checking = False  
  
gathering = smart  
gather_subset = network,hardware  
fact_caching = jsonfile  
fact_caching_connection = fact_files  
retry_files_save_path = retry_files  
  
stdout_callback = yaml  
callback_whitelist = profile_tasks, timer  
  
roles_path = roles  
  
deprecation_warnings = False  
log_path = ansible.log  
  
[ssh_connection]  
pipelining=True  
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ConnectionAttempts=100 -o UserKnownHostsFile=/dev/null
```