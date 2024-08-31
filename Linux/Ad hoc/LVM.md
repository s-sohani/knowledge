```
sudo pvcreate /dev/sdX
```

```
sudo vgcreate my_vg_name /dev/sdX /dev/sdY
```

```
sudo lvcreate -l 100%FREE -n my_lv_name my_vg_name
```

```
sudo mkfs.ext4 /dev/my_vg_name/my_lv_name
```

```
/dev/my_vg_name/my_lv_name /mnt/my_mount_point ext4 defaults 0 0
```



