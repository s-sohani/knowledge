```lua
server {
        listen 80;
        resolver  ***.***.***.*** ipv6=off;
        error_page 301 302 307 = @handle_redirect;
        recursive_error_pages on;
        access_log /data/log/head_redirector.log main;
        location / {
                proxy_pass https:/$request_uri;
                proxy_bind $ip;
        }
        location @handle_redirect {
                proxy_pass $saved_redirect_location;
                proxy_bind $ip;
                proxy_method HEAD;
                set $saved_redirect_location '$upstream_http_location';
        }
}
```

# Disable redirect
```
proxy_hide_header Location;  
proxy_redirect off;
```



