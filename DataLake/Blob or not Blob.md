# File size challenges in open-source ObjectStores
# **Introduction**

ObjectStore is a great place to keep various types of data. It writes data once and reads many times, and it may be hosted securely. I experimented some ObjectStores and in this article, will share my experience in working with them. I hope it helps you to make the best decision in selecting an ObjectStore.

![](https://miro.medium.com/v2/resize:fit:421/1*QdlB2HN2wLs2-EeAPLP47A.png)

# ObjectStore

**ObjectStore** is a commercial object database, a specialized type of NoSQL database designed to handle data created by applications or files. In other words, ObjectStore is an object-based storage. This includes unstructured data like email, videos, photos, docker images, web pages, audio files, sensor data, and other types of media and web content (textual or non-textual).

Each object is a simple, self-contained repository that includes the data, metadata (descriptive information associated with an object, useful for important functions such as policies for retention, deletion and routing and additional context that can be later extracted and leveraged to perform business insights), and a unique identifying ID number (instead of a file name and file path). Objects are discrete units of data that are stored in a structurally flat data environment. There are no folders, directories, or complex hierarchies as in a file-based system.

![ObjectStore System, File size challenges](https://miro.medium.com/v2/resize:fit:356/1*Aa6r3m_L5nKcLf7CuaL3Jw.png)

ObjectStore System

With object-based storage, you can store and manage data volumes on the order of terabytes (TBs), petabytes (PBs), and even larger. You can replicate objects to other storage and aggregate object storage devices into larger storage pools and distribute these storage pools across locations. This allows for unlimited scale, as well as improved data resiliency and disaster recovery.

With regard to object-based storage systems there are several open source solutions available such as Ceph, MinIO, Openio.io, and Apache Ozone. While these ObjectStores have different features, policy options, and methodologies, each has the same goal — to enable large-scale storage of unstructured, digital data.

# **File Size Challenges**

## IOPS

IOPS (input/output operations per second) is the standard unit of measurement for the maximum number of reads and writes to storage. An IOPS is made up of seek time, read time and data transmission time. For HDD storage, maximum number of IOPS is 200 per sec and it’s about 40000 for SSD. If stored files in ObjectStore database are small, IOPS may be the bottleneck. These databases usually store metadata separately, so a part of IOPS will be spent for storing metadata. For example if the average size of files is about 50Kb then maximum throughput you can get from HDD storage is about 10Mb/sec (ignoring metadata overload) while by considering metadata overload, it will be less. So it will be recommended to store large files in theses databases to avoid the IOPS problem.

What is large or small file? I will explain it in another article in the future.

To monitor IOPS in ubuntu, use this command:

> _iostat -dx 1_

## INode

By definition, an inode is an index node. It serves as a unique identifier for a specific piece of metadata on a given filesystem. The more number files you have, the lesser number of free inodes will be available. Therefor if the size of files is small, you can store more files and inode free space will decrease rapidly.

To monitor INode space in ubuntu, use this command:

> _df -i_

## Metadata Database

Some ObjectStore systems index file’s metadata in some nodes. If the average size of files is small then the size of index database and the number of indexes increase rapidly. On the other hand, when the size of object is large metadata database is the bottleneck. Therefor it’s better to use ObjectStores to store large files in order to make metadata database smaller.

## Network Throughput

ObjectSotore systems make replicates of an file to other nodes. It’s usually creates three replicates per file. Often, clients send data to one node, then the node makes a copy of that file to other nodes. This process makes load on network, So it is recommended to use suitable network. To test your network and become confident of your network throughput, you can use **iperf** command.

## Disk Throughput

**dd** is a command utility for Unix operating systems whose primary purpose is to convert and copy files. dd reads from stdin and writes to stdout, but this can be changed by using the if (input file) and of (output file) options. By dd you can test your disk throughput to compute how many bytes can be served by disk per second. To use dd for this goal, use this commands:

> _dd if=/dev/zero of=/mnt/drive/test bs=16M count=64 oflag=direct_
> 
> _dd of=/dev/null if=/mnt/drive/test bs=16M count=64 iflag=direct_

## Conclusion

Querying many small files incurs overhead (due to reading metadata), conducts a non-contiguous disk seek, open the file, close the file and repeat. The overhead is only a few milliseconds per file, but when you’re querying thousands, millions or even billions of files, those milliseconds add up. **So it’s better to store small files to other NoSql databases (e.g. as blobs) and store large files to ObjectStore databases.**