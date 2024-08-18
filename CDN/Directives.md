# proxy_bind
This directive is particularly useful in scenarios where the server has multiple network interfaces, and you want to control which interface (and thus which IP address) is used for outbound connections to the upstream server.
It can also be used for controlling the source IP in a connection when dealing with firewalls or upstream servers that have IP-based access control.
``` nginx
location / { 
	proxy_pass http://backend_server; 
	proxy_bind $ip; 
}
```


