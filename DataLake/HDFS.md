
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

The NameNode stores file system metadata in two different files: the **fsimage** and the **edit log**.
The fsimage stores a complete snapshot of the file system’s metadata at a specific moment in time. Incremental changes are then stored in the edit log for durability, rather than creating a new fsimage snapshot each time the namespace is modified.

The NameNode can restore its state by loading the fsimage and performing all the transforms from the edit log, restoring the file system to its most recent state.
