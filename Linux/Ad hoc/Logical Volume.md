#### Step 1: Unmount the Disk if Already Mounted

As you have '/mnt/data1' already mounted to 'xvdc',you should unmount if first:

|   |
|---|
|`sudo umount /mnt/data1`<br><br>`sudo umount /mnt/data2`<br><br>`sudo umount /mnt/data3`|

  

#### Step 2: Install LVM if needed

|   |
|---|
|`sudo apt``-get` `update`<br><br>`sudo apt``-get` `install lvm2`|

#### Step 3: Create Physical Volumes

|   |
|---|
|`sudo pvcreate /dev/xvdc /dev/xvde /dev/xvdf`|

#### Step 4: Create a Volume Group

|   |
|---|
|`sudo vgcreate mongodb_vg /dev/xvdc /dev/xvde /dev/xvdf`|

#### Step 5: Create a Logical Volume 

|   |
|---|
|`sudo lvcreate` `-l` `100``%``FREE` `-n` `mongodb_lv mongodb_vg`|

#### Step 6: Create Filesystem

|   |
|---|
|`sudo mkfs.xfs /dev/mongodb_vg/mongodb_lv`|

#### Step 7: Mount the Logical Volume

|   |
|---|
|`sudo mkdir /data`<br><br>`sudo` `mount` `/dev/mongodb_vg/mongodb_lv /data`|

  

#### Step 8: Update /etc/fstab

|   |
|---|
|`echo` `"/dev/mongodb_vg/mongodb_lv /data xfs defaults 0 0"` `\| sudo` `tee` `-a` `/etc/fstab`|