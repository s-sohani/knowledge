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
These pairs appear in the order that they were written, and values later in the log take precedence over values for the same key earlier in the log. We require that the sequence of key-value pairs is sorted by key. We call this format **Sorted String Table**, or **SSTable** for short. We also require that each key only appears once within each merged segment file. 
SSTables have several big advantages:
- Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the mergesort algorithm.
![[Pasted image 20240619070219.png|600]]

- In order to find a particular key in the file, you no longer need to keep an index of all the keys in memory. You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly. For example if you now `hadbag` and `hadsome` offset, you can scan keys between them to find `hadiwork` offset.
![[Pasted image 20240619070818.png|600]]

- Since read requests need to scan over several key-value pairs in the requested range anyway, it is possible to group those records into a block and compress it before writing it to disk.

### Constructing and maintaining SSTables
To maintain data sorted by key, an in-memory balanced tree structure like a red-black or AVL tree (a memtable) is used. Incoming writes are added to the memtable, which maintains sorted key-value pairs. When the memtable reaches a certain size, it's written to disk as an SSTable file, maintaining the sorted order. Read requests check the memtable first, then recent SSTable segments. A background process periodically merges and compacts these segments. To prevent data loss during a crash, writes are also appended to an unsorted log on disk, used to restore the memtable after a crash. The log is discarded once the memtable is saved as an SSTable.

### Making an LSM-tree out of SSTables
The algorithm described is used in key-value storage engines like LevelDB and RocksDB, which can be embedded in other applications. It's also used in Riak, Cassandra, and HBase, inspired by Google’s Bigtable. This indexing structure, known as the Log-Structured Merge-Tree (LSM-Tree), was first described by Patrick O’Neil and is based on merging and compacting sorted files. Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, employs a similar method to store its term dictionary, mapping words to document IDs in SSTable-like files, which are merged in the background.

### Performance optimizations
Making a storage engine perform well involves various optimizations. The LSM-tree algorithm can be slow for non-existent key lookups, requiring checks from the memtable to the oldest segment, potentially reading from disk each time. To address this, **Bloom filters** are used to efficiently determine if a key does not exist, reducing unnecessary disk reads. Different compaction strategies, such as size-tiered and leveled compaction, manage how SSTables are merged. Size-tiered compacts newer, smaller SSTables into older (larger ones), while leveled compaction splits the key range into smaller SSTables and organizes older data into levels. Despite complexities, the LSM-tree's core idea of merging SSTables in the background is simple and effective, supporting high write throughput and efficient range queries.

## B-Trees
While log-structured indexes are gaining traction, B-trees remain the most common indexing structure. Introduced in 1970, B-trees are standard in almost all relational databases and many non-relational ones. Like SSTables, B-trees maintain sorted key-value pairs but differ significantly in design. B-trees use fixed-size blocks or pages (e.g., 4 KB), reading and writing one page at a time, aligning closely with hardware storage.

B-trees are organized with a root page leading to child pages through references, forming a hierarchical tree. Each page covers a range of keys, and searching involves traversing from the root to the relevant leaf page. Updates and insertions are handled within pages, splitting them if necessary to maintain balance. The tree depth remains O(log n), typically just three or four levels deep, enabling efficient lookups and updates. The number of references to child pages in one page of the B-tree is called the branching factor. A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB.
![[Pasted image 20240619074206.png|600]]

In the example, we are looking for the key 251, so we know that we need to follow the page reference between the boundaries 200 and 300. That takes us to a similar-looking page that further breaks down the 200–300 range into subranges.

### Making B-trees reliable
In order to make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (WAL, also known as a redo log). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself.
An additional complication of updating pages in place is that careful concurrency control is required if multiple threads are going to access the B-tree at the same time —otherwise a thread may see the tree in an inconsistent state. This is typically done by protecting the tree’s data structures with latches (lightweight locks).

