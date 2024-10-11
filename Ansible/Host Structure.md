
```yaml
all:  
  hosts:  
    app2:  
      ansible_host: 172.16.16.164  
    app3:  
      ansible_host: 172.16.16.152
```

```yaml
apps:  
  hosts:  
    app2:    
  vars:  
    runner_token:  
      app2: "glrt-f6Mzp_pTXWxy6FDT79-D"  
      app3: "glrt-6DUkB4xSM7eFhKBATq7z"
```

