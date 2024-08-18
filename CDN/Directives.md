# proxy_bind
This directive is particularly useful in scenarios where the server has multiple network interfaces, and you want to control which interface (and thus which IP address) is used for outbound connections to the upstream server.
It can also be used for controlling the source IP in a connection when dealing with firewalls or upstream servers that have IP-based access control.
``` nginx
location / { 
	proxy_pass http://backend_server; 
	proxy_bind $ip; 
}
```

# Caching
```nginx
proxy_cache_path /var/cache/site_cache levels=1:2 keys_zone=site_cache:10m max_size=100m inactive=60m use_temp_path=off;
```

**`levels=1:2`**:
- **Purpose**: This defines the directory structure for storing cached files.
- **Explanation**: The `levels` parameter controls the subdirectory levels under the cache path. In this example, `1:2` means:
    - The first level will have directories with 1 character (e.g., `/a/`).
    - The second level will have directories with 2 characters (e.g., `/a/ab/`).
    - This structure helps avoid having too many files in a single directory, which can be inefficient for file system performance.

**`keys_zone=site_cache:10m`**:
fines the name and size of the shared memory zone used to store cache keys and metadata.
- **Explanation**:
    - **`site_cache`**: The name of the cache zone.
    - **`10m`**: The size of the shared memory zone, in this case, 10 megabytes. This memory is used to store metadata about cached items, such as keys and usage statistics.

**`max_size=100m`**:
- **Purpose**: This sets the maximum size of the cache on disk.
- **Explanation**: The total size of the cache files stored in `/var/cache/site_cache` will not exceed 100 megabytes. When this limit is reached, older files will be removed according to the LRU (Least Recently Used) policy.

**`inactive=60m`**:
    - **Purpose**: This defines how long a cached item should remain in the cache if it is not accessed.
    - **Explanation**: If a cached item is not accessed for 60 minutes (1 hour), it will be removed from the cache, regardless of whether the cache is full.

**`use_temp_path=off`**:    
    - **Purpose**: This controls whether or not temporary files are written to a separate directory before being moved to the cache.
    - **Explanation**:
        - **`off`**: Cached files are written directly to their final location in the cache directory. This can improve performance by avoiding the overhead of moving files from a temporary directory.
        - **`on`** (the default): Cached files are first written to a temporary directory (usually `/var/lib/nginx/proxy`) and then moved to the final cache location.
