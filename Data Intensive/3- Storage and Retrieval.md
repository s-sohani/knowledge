Understanding internal storage mechanisms helps developers choose and tune the appropriate storage engine for their applications. The chapter will explore storage engines optimized for transactional versus analytical workloads, starting with traditional relational and most NoSQL databases, specifically examining log-structured and page-oriented storage engines like B-trees.

# Data Structures That Power Your Database
A simple database can be implemented with two Bash functions: `db_set` to **append** key-value pairs to a file, and `db_get` to **retrieve** the latest value for a key by scanning the file. This approach is efficient for appending data but inefficient for lookups, as it requires scanning the entire file. Efficient data retrieval requires indexes, which provide metadata to locate data quickly. Indexes speed up read queries but slow down writes, necessitating careful selection based on query patterns to balance performance and overhead.

## Hash Indexes
Indexes for key-value data are essential for efficient retrieval. A simple approach involves appending key-value pairs to a file and maintaining an in-memory hash map to track byte offsets. This method is effective for high-performance reads and writes if all keys fit in memory. To manage disk space and performance, logs can be segmented and compacted, merging segments to keep lookups efficient. While this approach simplifies concurrency and crash recovery, it has limitations: the hash map must fit in memory, and range queries are inefficient. More advanced indexing structures can address these issues.

### Compaction
Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value. If that part of the data file is already in the filesystem cache, a read doesn’t require any disk I/O at all.
![[Pasted image 20240618065826.png|600]]

As described so far, we only ever append to a file—so how do we avoid eventually running out of disk space? A good solution is to break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform compaction on these segments, as illustrated in Figure 3-2. Compaction means throwing away duplicate keys in the log, and keeping only the most recent update for each key.
![[Pasted image 20240618070214.png|600]]

Moreover, since compaction often makes segments much smaller (assuming that a key is overwritten several times on average within one segment), we can also merge several segments together at the same time.
![[Pasted image 20240618070635.png|600]]
- The merging and compaction of frozen segments can be done in a background thread
- Each segment now has its own in-memory hash table, mapping keys to file offsets.
- In order to find the value for a key, we first check the most recent segment’s hash map; if the key is not present we check the second-most-recent segment, and so on.
- If you want to delete a key and its associated value, you have to append a special deletion record to the data file (sometimes called a tombstone). When log segments are merged, the tombstone tells the merging process to discard any previ‐ ous values for the deleted key.
- If a database restarts, in-memory hash maps are lost. Restoring them by reading the entire segment file is time-consuming for large files. Bitcask addresses this by storing a snapshot of each segment’s hash map on disk for quicker recovery.
- The database may crash at any time, including halfway through appending a record to the log. Bitcask files include checksums, allowing such corrupted parts of the log to be detected and ignored.
- Log files are immutable, so we can write with multithread to the same file. 

Why don’t you update the file in place, overwriting the old value with the new value? An append-only design turns out to be good for several reasons:
 - Concurrency and crash recovery are much simpler. leaving you worry with a file containing part of the old and part of the new value spliced together.
 - Appending much faster than random writes especially on magnetic spinning-disk hard drives.
 - Merging old segments avoids the problem of data files getting fragmented over time.
Hash table index also has limitations:
- The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O. 
- Range queries are not efficient. For example, you cannot easily scan over all keys between kitty00000 and kitty99999—you’d have to look up each key individually in the hash maps.

## SSTables and LSM-Trees

