sudo fio --numjobs=64 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/data/ --bs=4k --iodepth=64 --size=1G --readwrite=randwrite --group_reporting=1
 


sudo fio --numjobs=16 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/data/ --bs=1k --iodepth=64 --size=64M --readwrite=randread --group_reporting=1


sudo fio --numjobs=16 --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --directory=/data/ --bs=1k --iodepth=64 --size=64M --readwrite=randwrite --group_reporting=1