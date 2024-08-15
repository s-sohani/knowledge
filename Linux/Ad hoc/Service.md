[Unit]  
Description=movadian  
[After=network-online.target](http://after=network-online.target/)  
[Wants=network-online.target](http://wants=network-online.target/)  
[Service]  
WorkingDirectory=/home/user  
User=user  
ExecStart=/home/user/exec.sh  
Restart=always  
RestartSec=5s  
Type=forking  
LimitNOFILE=infinity  
LimitNPROC=infinity  
[Install]  

[WantedBy=multi-user.target](http://wantedby=multi-user.target/)