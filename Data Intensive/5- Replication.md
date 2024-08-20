There are various reasons why you might want to distribute a database across multiple machines:
- Scalability
- Fault tolerance/high availability
- Latency

# Scaling to Higher Load
If you constantly have a high load, it's better to do vertical scaling and have a machine with sufficient resources.
Next, it mentions two methods for vertical scaling: shared-memory architecture and shared-disk architecture.
The shared-memory architecture suggests that you can connect several CPUs, RAM, or disks together and place them under a single operating system.
The shared-disk architecture, on the other hand, suggests that you can have several independent machines, each with its own CPU and RAM, but all working on a shared disk.

The problem with shared-memory architecture is that when you have a machine with twice the resources, first, the costs more than double, and secondly, it can't handle exactly twice the load due to various reasons, including bottlenecks.
The problem with shared-disk architecture is the overhead of locking, which disrupts the system's ability to scale.

# Shared-Nothing Architectures
The opposite of this is the shared-nothing architecture, also known as horizontal scaling. In this architecture, you distribute the data across independent machines or nodes.
It seems that this approach can reduce latency and help address the issue of data loss.

# Replication Versus Partitioning
In the shared-nothing architecture, data is distributed across different nodes.

There are two different approaches for data distribution:
- **Replication:** Keeping multiple identical copies of data on different machines.
- **Partitioning:** Dividing data into smaller parts and storing each part on an independent machine. This is also known as sharding.

# Replication
Replication means having multiple copies of the same data on different machines that are accessible through a network.

### Why Replication is Necessary:
1. **Reducing Latency**: By placing data geographically closer to users, we can reduce latency.
2. **Fault Tolerance**: To ensure that the system remains available even if one copy of the data fails.
3. **Increasing Throughput**: Distributing read requests across multiple machines can increase the system's throughput.

### Challenges in Replication:
The main difficulty in replication is that data is constantly changing, so the copies need to be kept in sync.

### Common Methods for Data Synchronization:
1. **Single-Leader**
2. **Multi-Leader**
3. **Leaderless**

Challenges also arise in choosing between synchronous and asynchronous replication, as well as handling failed replicas.

### Common Solution: Master/Slave or Active/Passive Model
In this model, writes are only performed on the master, while reads can be done from any replica. The master, acting as the leader, must propagate data changes to the other nodes.
This Master/Slave model is a built-in feature of most relational and non-relational databases.

![[Pasted image 20240813180742.png|600]]

### Synchronous vs Asynchronous Replication:
- **Synchronous Replication**: A successful write response is only returned to the client when all slaves have synchronized with the master. This model ensures that no written data is lost if the leader fails. However, it blocks write requests until the data is reflected on all followers. If a follower encounters an issue during this process, the entire write operation may fail.

- **Asynchronous Replication**: In this model, a successful response is returned to the client as soon as the data is written to the master, and data is then propagated to the slaves in parallel. While this ensures that the system remains operational even if all followers fail, there’s a risk that if the leader fails before syncing with the followers, some written data may be lost, making the write operation non-durable despite the client receiving a confirmation.

Typically, a leader-based model is configured to be fully asynchronous. Although the loss of durability is not ideal, this replication model is widely used, especially when there are many followers or they are geographically dispersed.

![[Pasted image 20240813181004.png|600]]

### Adding a New Follower:
When adding a new follower, which node’s data should be copied to the new follower? To avoid downtime, locking the data is not an option.

Steps for copying the latest version of data to the new follower:
1. Take snapshots of the leader’s data at specific time points.
2. Copy the snapshot to the new follower.
3. The new follower requests any remaining data changes that occurred after the snapshot from the leader.
4. Once the new follower has processed the backlog, it announces that it is ready.

# Handling Node Outages
For various reasons, such as node failure, rebooting nodes during maintenance, or installing kernel security patches, one or more nodes might drop out of the replication set. Availability dictates that in such situations, the system should not completely fail or experience downtime. How is availability managed in leader-based architectures?

If a follower experiences a failure or, for any reason, loses connection with the leader for a while, it can use the data-changes log it previously obtained from the leader and stored on its disk to recover itself. It can then request new changes from the leader that occurred after the failure to update its data.

Leader failure is much more complex. In this situation, one of the followers needs to be selected as the new leader. Additionally, all clients need to be reconfigured to send write operations to the new leader, and all followers must also be reconfigured to receive data-logs from the new leader. This process is done automatically in several stages.

While it cannot be determined with absolute certainty, the nodes continuously exchange messages among themselves, and if a node does not respond for a certain period, it is considered dead.

The new leader is selected using one of two methods:

1. Through an election process where a node is chosen by majority vote.
2. The new leader is selected by a controller node.

The best candidate for becoming the leader is the node with the most up-to-date data to minimize data loss. All nodes must update their routing to point to the new leader. If the previous leader resolves its issue and returns, the system must inform it that it is no longer the leader and should be added back as a follower.

Data loss during the failover process can also cause issues elsewhere. For example, there was an incident on GitHub where a follower from MySQL with outdated data was selected as the new leader. This resulted in data loss for primary key counters, causing duplicate primary keys to be used from a certain point onward. This issue also affected data in Redis, leading to personal information of some users being displayed to other individuals.
A problematic scenario is when, during failover, two nodes simultaneously think they are the leader. This condition is known as "split brain."
This can lead to data loss or corruption. Some systems handle this by shutting down one of the leaders. However, if this solution is not implemented carefully, it might result in both leaders being shut down.
How long should the timeout be to determine if a node is dead?
A long timeout increases the recovery time, while a short timeout can result in unnecessary failovers. Additionally, temporary load on the system or temporary network issues can increase response times.


# Implementation of Replication Logs
As mentioned in Chapter 3, all write and update operations are recorded in the file using the write-ahead method. This can be applied to mechanisms such as SSTables, LSM-Trees, or B-trees. However, the core concept remains the Write-ahead log.

Each update operation is recorded in several copies, each of which only writes and changes one record. This log is called a logical log.

Mechanisms similar to triggers and stored procedures available in some databases can be used to propagate data changes to other nodes.

For example, you can create a trigger that generates a log upon data changes and writes it somewhere else. Another process can then distribute those logs to the other nodes.

However, this method has higher costs and a greater chance of bugs compared to other methods, but it offers high flexibility.

# Problems with Replication Lag

