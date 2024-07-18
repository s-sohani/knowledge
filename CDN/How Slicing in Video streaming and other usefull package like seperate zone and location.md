## Convert image to proper format with nginx
https://github.com/nginx/nginx/blob/master/src/http/modules/ngx_http_image_filter_module.c

## Nginx virtual host traffic status
https://github.com/vozlt/nginx-module-vts

## HLS/RTML live Streaming
https://github.com/arut/nginx-rtmp-module


## Video slicing
```bash 
./configure -j2 --with-cc-opt='-DNGX_LUA_ABORT_AT_PANIC -I/usr/local/openresty/zlib/include -I/usr/local/openresty/pcre/include -I/usr/local/openresty/openssl111/include' --with-http_slice_module --with-http_v2_module  --with-http_stub_status_module --with-http_realip_module --with-http_random_index_module --with-http_gzip_static_module --with-http_sub_module --with-http_gunzip_module  --with-threads  --with-pcre-jit
```
