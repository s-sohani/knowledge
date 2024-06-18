Understanding internal storage mechanisms helps developers choose and tune the appropriate storage engine for their applications. The chapter will explore storage engines optimized for transactional versus analytical workloads, starting with traditional relational and most NoSQL databases, specifically examining log-structured and page-oriented storage engines like B-trees.

# Data Structures That Power Your Database
A simple database can be implemented with two Bash functions: `db_set` to append key-value pairs to a file, and `db_get` to retrieve the latest value for a key by scanning the file. This approach is efficient for appending data but inefficient for lookups, as it requires scanning the entire file. Efficient data retrieval requires indexes, which provide metadata to locate data quickly. Indexes speed up read queries but slow down writes, necessitating careful selection based on query patterns to balance performance and overhead.

## Hash Indexes
Indexes for key-value data are essential for efficient retrieval. A simple approach involves appending key-value pairs to a file and maintaining an in-memory hash map to track byte offsets. This method is effective for high-performance reads and writes if all keys fit in memory. To manage disk space and performance, logs can be segmented and compacted, merging segments to keep lookups efficient. While this approach simplifies concurrency and crash recovery, it has limitations: the hash map must fit in memory, and range queries are inefficient. More advanced indexing structures can address these issues.

Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value. If that part of the data file is already in the filesystem cache, a read doesn’t require any disk I/O at all.
![[Pasted image 20240618065826.png]]

As described so far, we only ever append to a file—so how do we avoid eventually running out of disk space? A good solution is to break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform compaction on these segments, as illustrated in Figure 3-2. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key.
![[Pasted image 20240618070214.png]]

