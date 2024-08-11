# Understand the problem
- which feature include in design?
- what is the type of data size?
- How much data do we need to store in one year?
- How long data durability and system availability

Object storage usually bottlenecks in disk IOPS. 
Design of object store like UNIX file system. file name stores in data structure named inode and file data stores in another location. 