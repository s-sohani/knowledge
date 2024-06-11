
## HDFS Overview
- designed with hardware failure in mind
- built for large datasets, with a default block size of 128 MB
- optimized for sequential operations
- rack-aware
- cross-platform and supports heterogeneous clusters

Data in a Hadoop cluster is broken down into smaller units (called blocks) and distributed throughout the cluster. Each block is duplicated twice (for a total of three copies), with the two replicas stored on two nodes in a rack somewhere else in the cluster. Since the data has a default replication factor of three, it is highly available and fault-tolerant.

HDFS is based on a **leader/follower** architecture. Each cluster is typically composed of a single NameNode, an optional SecondaryNameNode (for data recovery in the event of failure), and an arbitrary number of DataNodes.

### NameNode
In addition to managing the file system namespace and associated metadata (file-to-block maps), the NameNode acts as the leader and brokers access to files by clients.
Though once brokered, clients communicate directly with DataNodes.
The NameNode operates entirely **in memory**, persisting its state to disk.
It represents a single point of failure for a Hadoop cluster that is not running in high-availability mode.  In high-availability mode, Hadoop maintains a standby NameNode to guard against failures.

The NameNode stores file system metadata in two different files: the **fsimage** and the **edit log** (same as aof and rdb in redis).
The fsimage stores a complete snapshot of the file system’s metadata at a specific moment in time. Incremental changes are then stored in the edit log for durability, rather than creating a new fsimage snapshot each time the namespace is modified.

The NameNode can restore its state by loading the fsimage and performing all the transforms from the edit log, restoring the file system to its most recent state.

if the NameNode goes down in the presence of a SecondaryNameNode, the NameNode doesn’t need to replay the edit log on top of the fsimage; cluster administrators can retrieve an updated copy of the fsimage from the SecondaryNameNode.

### HA NameNode service
Early versions of Hadoop introduced several concepts (like SecondaryNameNodes, among others) to make the NameNode more resilient. With Hadoop 2.0 and Standby NameNodes, a mechanism for true high availability was realized.

Standby NameNodes, which are incompatible with SecondaryNameNodes, provide automatic failover in the event of primary NameNode failure. Achieving high availability with Standby NameNodes requires shared storage between the primary and standbys

![[Pasted image 20240611170319.png]]


## MapReduce overview
![[Pasted image 20240611170547.png]]

YARN (Yet Another Resource Negotiator) is the framework responsible for assigning computational resources for application execution.
![[Pasted image 20240611170646.png]]

### YARN
YARN consists of three core components:
- ResourceManager (one per cluster)
- ApplicationMaster (one per application)
- NodeManagers (one per node)

#### ResourceManager
The ResourceManager is the rack-aware leader node in YARN. It is responsible for taking inventory of available resources and runs several critical services, the most important of which is the Scheduler.
The Scheduler component of the YARN ResourceManager allocates resources to running applications.

#### ApplicationMaster
Each application running on Hadoop has its own dedicated ApplicationMaster instance.
- sends heartbeat messages to the ResourceManager
- requests for additional resources
- execution of an application
- submitting container release requests to the NodeManager

#### NodeManagers
The NodeManager is a per-node agent tasked with overseeing containers.

### ZooKeeper
Apache ZooKeeper is a popular tool used for coordination and synchronization of distributed systems.
Enabling high-availability of former single points of failure, specifically the **HDFS NameNode** and **YARN ResourceManager**.
![[Pasted image 20240611171645.png]]

