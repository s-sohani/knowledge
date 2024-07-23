Understanding **internal storage** mechanisms helps developers **choose** and **tune** the appropriate storage **engine** for their applications. The chapter will explore storage engines optimized for transactional versus analytical workloads, starting with traditional relational and most NoSQL databases, specifically examining log-structured and page-oriented storage engines like B-trees.

# Data Structures That Power Your Database
A **simple database** can be implemented with two Bash functions: `db_set` to **append** key-value pairs to a file, and `db_get` to **retrieve** the latest value for a key **by scanning the file**. This approach is **efficient** for appending data but **inefficient** for lookups, as it requires scanning the entire file. Efficient data retrieval requires indexes, which provide metadata to locate data quickly. **Indexes** speed up read queries but slow down writes, necessitating careful selection based on query patterns to balance performance and overhead.

## Hash Indexes
Indexes for key-value data are essential for **efficient retrieval**. A simple approach involves appending key-value pairs to a file and maintaining an **in-memory hash map** to track byte offsets. This method is effective for high-performance reads and writes **if** all keys **fit** in memory. To manage disk space and performance, logs can be **segmented** and **compacted**, merging segments to keep lookups efficient. While this approach simplifies **concurrency** and crash **recovery**, it has **limitations**: the hash map must **fit** in memory, and **range** queries are inefficient. More advanced indexing structures can address these issues.

### Compaction
Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value. If that part of the data file is already in the filesystem cache, a read doesn’t require any disk I/O at all.
![[Pasted image 20240618065826.png|600]]

As described so far, we only ever **append** to a file—so how do we avoid eventually **running out of disk space**? A good solution is to break the log into segments of a certain size by closing a segment file when it reaches a certain size, and making subsequent writes to a new segment file. We can then perform compaction on these segments, as illustrated in Figure 3-2. Compaction means **throwing away duplicate keys** in the log, and keeping only the most recent update for each key.
![[Pasted image 20240618070214.png|600]]

Moreover, since compaction often makes segments much smaller (assuming that a key is overwritten several times on average within one segment), we can also **merge several segments** together at the same time.
![[Pasted image 20240618070635.png|600]]
- The merging and compaction of frozen segments can be done in a **background thread**
- Each segment now has its **own** in-memory **hash table**, mapping keys to file offsets.
- In order to **find** the value for a key, we first check the most recent segment’s hash map; if the key is not present we check the second-most-recent segment, and so on.
- If you want to **delete a key** and its associated value, you have to append a special deletion record to the data file (sometimes called a tombstone). When log segments are merged, the tombstone tells the merging process to discard any previous values for the deleted key.
- If a **database restarts**, in-memory hash maps are lost. Restoring them by reading the entire segment file is time-consuming for large files. Bitcask addresses this by **storing a snapshot** of each segment’s hash map on disk for quicker recovery.
- The database may **crash** at any time, including halfway through appending a record to the log. Bitcask files include checksums, allowing such corrupted parts of the log to be detected and ignored.
- Log files are immutable, so we can write with **multi thread** to the same file. 

Why don’t you update the file in place, overwriting the old value with the new value? An append-only design turns out to be good for several reasons:
 - **Concurrency** and crash recovery are much simpler. leaving you worry with a file containing part of the old and part of the new value spliced together.
 - Appending much **faster** than random writes especially on magnetic spinning-disk hard drives.
 - Merging old segments avoids the problem of data files getting **fragmented** over time.
Hash table index also has limitations:
- The hash table must **fit** in **memory**, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires **a lot of random access I/O**. 
- **Range** queries are not efficient. For example, you cannot easily scan over all keys between kitty00000 and kitty99999—you’d have to look up each key individually in the hash maps.

## SSTables and LSM-Trees
These pairs appear in the order that they were written, and values later in the log take precedence over values for the same key earlier in the log. We require that the sequence of key-value pairs is **sorted by key**. We call this format **Sorted String Table**, or **SSTable** for short. We also require that each key only appears once within each merged segment file. 
SSTables have several big advantages:
- Merging segments is simple and efficient, even if the files are bigger than the available memory. The approach is like the one used in the **mergesort algorithm**.
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
### Comparing B-Trees and LSM-Trees
B-trees and LSM-trees have different performance characteristics: LSM-trees generally offer faster write speeds, while B-trees are typically faster for read operations. This is because LSM-trees must check multiple data structures and SSTables at various compaction stages during reads. 

#### Advantages of LSM-trees
- A B-tree index must write every piece of data at least twice: once to the write-ahead log, and once to the tree page itself (and perhaps again as pages are split). There is also overhead from having to write an entire page at a time, even if only a few bytes in that page changed.
- Moreover, LSM-trees are typically able to sustain higher write throughput than B- trees, partly because they sometimes have lower write amplification.
- LSM-trees can be compressed better, and thus often produce smaller files on disk than B-trees. B-tree storage engines leave some disk space unused due to fragmenta‐ tion: when a page is split or when a row cannot fit into an existing page, some space in a page remains unused.
#### Downsides of LSM-trees
Log-structured storage, like LSM-trees, can experience performance interference during compaction, affecting read and write operations. While typically minor, high percentile response times can be significantly impacted, making B-trees more predictable. High write throughput can overwhelm compaction processes, causing unmerged segments to accumulate and slow down reads, necessitating careful monitoring.

B-trees, with each key existing in one place, are advantageous for strong transactional semantics, often used in relational databases for efficient range key locking. Despite the rise of log-structured indexes in new datastores, B-trees remain integral to many databases due to their reliable performance across various workloads. 

## Other Indexing Structures
So far we have only discussed key-value indexes, which are like a primary key index in the relational model. A primary key uniquely identifies one row in a relational table, or one document in a document database, or one vertex in a graph database. It is also very common to have secondary indexes. In relational databases, you can create several secondary indexes on the same table using the CREATE INDEX. A secondary index can easily be constructed from a key-value index. The main difference is that keys are not unique; i.e., there might be many rows (documents, vertices) with the same key. This can be solved in two ways: either by making each value in the index a list of matching row identifiers (like a postings list in a full-text index) or by making each key unique by appending a row identifier to it. Both B-trees and log-structured indexes can be used as secondary indexes.

### Storing values within the index
The key in an index is the thing that queries search for, but the value can be one of two things: it could be the actual row (document, vertex) in question, or it could be a reference to the row stored elsewhere.
A compromise between a clustered index (storing all row data within the index) and a nonclustered index (storing only references to the data within the index) is known as a covering index or index with included columns, which stores some of a table’s columns within the index. 
As with any kind of duplication of data, clustered and covering indexes can speed up reads, but they require additional storage and can add overhead on writes. Databases also need to go to additional effort to enforce transactional guarantees, because appli‐ cations should not see inconsistencies due to the duplication.

### Multi-column indexes
The indexes discussed so far only map a single key to a value. That is not sufficient if we need to query multiple columns of a table (or multiple fields in a document) simultaneously.
The most common type of multi-column index is called a concatenated index, which simply combines several fields into one key by appending one column to another (the index definition specifies in which order the fields are concatenated). This is like an old-fashioned paper phone book, which provides an index from (lastname, first‐ name) to phone number. Due to the sort order, the index can be used to find all the people with a particular last name, or all the people with a particular lastname- firstname combination. However, the index is useless if you want to find all the peo‐ ple with a particular first name.

### Full-text search and fuzzy indexes
All the indexes discussed so far assume that you have exact data and allow you to query for exact values of a key, or a range of values of a key with a sort order. What they don’t allow you to do is search for similar keys, such as misspelled words. Such fuzzy querying requires different techniques. For example Lucene is able to search text for words within a certain edit distance.
Lucene uses a SSTable-like structure for its term dictionary. This structure requires a small in- memory index that tells queries at which offset in the sorted file they need to look for a key.

### Keeping everything in memory
Disks have two significant advantages: they are durable (their contents are not lost if the power is turned off), and they have a lower cost per gigabyte than RAM.
Many datasets are simply not that big, so it’s quite feasible to keep them entirely in memory.
Some in-memory key-value stores, such as Memcached, are intended for caching use only, where it’s acceptable for data to be lost if a machine is restarted. But other in- memory databases aim for durability, which can be achieved with special hardware, by writing a log of changes to disk, by writing periodic snapshots to disk, or by replicating the in-memory state to other machines.
Products such as VoltDB, MemSQL, and Oracle TimesTen are in-memory databases with a relational model, and the vendors claim that they can offer big performance improvements by removing all the overheads associated with managing on-disk data structures.
Redis and Couchbase provide weak durability by writing to disk asyn‐ chronously.

# Transaction Processing or Analytics?
In the early days of business data processing, a write to the database typically corresponded to a commercial transaction taking place like paying an employee’s salary.

> A transaction needn’t necessarily have ACID properties. Transaction processing just means allowing clients to make low-latency reads and writes as opposed to batch processing jobs, which only run periodically (for example, once per day).

Databases were initially used for various types of data, such as blog comments, game actions, and address book contacts, but the core access pattern remained similar to business transactions. Applications typically retrieve a few records using a key and update records based on user input, a pattern known as online transaction processing (OLTP). 

Over time, databases began to be used more for data analytics, which involves different access patterns. Analytical queries usually scan large numbers of records, read only a few columns per record, and calculate aggregate statistics (like counts, sums, or averages) rather than returning raw data. For example, analytic queries might include calculating total store revenue in a month, comparing sales during a promotion, or identifying purchase patterns of certain products. It has been called online analytic processing (OLAP)

## Data Warehousing
DBAs are usually reluctant to let business analysts run ad hoc analytic queries on an OLTP data‐ base, since those queries are often expensive, scanning large parts of the dataset, which can harm the performance of concurrently executing transactions. A data warehouse, is a separate database that analysts can query to their hearts’ content, without affecting OLTP operations.
The data warehouse con‐ tains a read-only copy of the data in all the various OLTP systems in the company. Data is extracted from OLTP databases. Process of getting data into the warehouse is known as Extract–Transform–Load (ETL).
![[Pasted image 20240622073221.png|600]]

The data model of a data warehouse is most commonly relational, because **SQL is generally a good fit for analytic** queries.
Some opensource project for data warehouses include Apache Hive, Spark SQL, Cloudera Impala, Facebook Presto, Apache Tajo, and Apache Drill.

## Stars and Snowflakes: Schemas for Analytics
Many data warehouses are used in a fairly formulaic style, known as a star schema (also known as dimensional modeling).
At the center of the star there is a table schema is a so-called fact table. Each row of the fact table represents an event that occurred at a particular time.
Some of the columns in the fact table are attributes, such as the price at which the product was sold and the cost of buying it from the supplier (allowing the profit mar‐ gin to be calculated). Other columns in the fact table are foreign key references to other tables, called dimension tables. As each row in the fact table represents an event, the dimensions represent the who, what, where, when, how, and why of the event.

![[Pasted image 20240622075338.png|600]]

A variation of this template is known as the snowflake schema, where dimensions are further broken down into subdimensions. For example, there could be separate tables for brands and product categories, and each row in the dim_product table could ref‐ erence the brand and category as foreign keys, rather than storing them as strings in the dim_product table. Snowflake schemas are more normalized than star schemas, but star schemas are often preferred because they are simpler for analysts to work with.

# Column-Oriented Storage
Although fact tables are often over 100 columns wide, a typical data warehouse query only accesses 4 or 5 of them at one time.
![[Pasted image 20240622084112.png|600]]
A row-oriented storage engine still needs to load all of those rows (each consisting of over 100 attributes) from disk into memory, parse them, and filter out those that don’t meet the required conditions. That can take a long time.
The idea behind column-oriented storage is simple: don’t store all the values from one row together, but store all the values from each column together instead. If each col‐ umn is stored in a separate file, a query only needs to read and parse those columns that are used in that query, which can save a lot of work.

## Column Compression
It's often look quite repetitive, which is a good sign for compression. Depending on the data in the column, different compression techniques can be used. One technique that is particu‐ larly effective in data warehouses is bitmap encoding.
![[Pasted image 20240623063118.png|600]]

## Sort Order in Column Storage
In a column store, it doesn’t necessarily matter in which order the rows are stored. It’s easiest to store them in the order in which they were inserted, since then inserting a new row just means appending to each of the column files. However, we can choose to impose an order, like we did with SSTables previously, and use that as an indexing mechanism.
A second column can determine the sort order of any rows that have the same value in the first column.

## Writing to Column-Oriented Storage
An update-in-place approach, like B-trees use, is not possible with compressed columns. If you wanted to insert a row in the middle of a sorted table, you would most likely have to rewrite all the column files. As rows are identified by their position within a column, the insertion has to update all columns consistently. Fortunately, we have already seen a good solution earlier in this chapter: LSM-trees. All writes first go to an in-memory store, where they are added to a sorted structure and prepared for writing to disk. It doesn’t matter whether the in-memory store is row-oriented or column-oriented. When enough writes have accumulated, they are merged with the column files on disk and written to new files in bulk.

## Aggregation: Data Cubes and Materialized Views
Another aspect of data warehouses that is worth mentioning briefly is materialized aggregates. As discussed earlier, data warehouse queries often involve an aggregate function, such as COUNT, SUM, AVG, MIN, or MAX. 
One way of creating such a cache is a materialized view. In a relational data model, it is often defined like a standard (virtual) view: a table-like object whose contents are the results of some query. The difference is that a materialized view is an actual copy of the query results, written to disk, whereas a virtual view is just a shortcut for writ‐ ing queries. When you read from a virtual view, the SQL engine expands it into the view’s underlying query on the fly and then processes the expanded query.
A common special case of a materialized view is known as a data cube or OLAP cube.
![[Pasted image 20240623083500.png|600]]
The advantage of a materialized data cube is that certain queries become very fast because they have effectively been precomputed. For example, if you want to know the total sales per store yesterday, you just need to look at the totals along the appro‐ priate dimension—no need to scan millions of rows. The disadvantage is that a data cube doesn’t have the same flexibility as querying the raw data. For example, there is no way of calculating which proportion of sales comes from items that cost more than $100, because the price isn’t one of the dimensions.

