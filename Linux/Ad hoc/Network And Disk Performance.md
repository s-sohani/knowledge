```
dd if=/dev/zero of=test.tmp bs=4k count=10000000  
fallocate -l 1G test
```

```
  "fio” for test disk"
  
sudo fio --numjobs=64 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/mnt/disk1/ --bs=4k --iodepth=64 --size=1G --readwrite=randrw --rwmixread=75 --group_reporting=1  
  
  
sudo fio --numjobs=64 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/mnt/disk1/ --bs=4k --iodepth=64 --size=1G --readwrite=randwrite --group_reporting=1  
  
  
sudo fio --numjobs=64 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/mnt/disk1/ --bs=4k --iodepth=64 --size=1G --readwrite=randread --group_reporting=1
```


```
iperf → to evaluate network throughput from client and server
```


```
number of connection to specific port  
ss -ntp | grep 9858 | wc -l  
  
number of open file (socket)  
lsof | wc -l  
  
limit of number of open file (socket)  
ulimit -n [number]
```
