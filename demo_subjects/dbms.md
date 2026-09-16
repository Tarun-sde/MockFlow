### Q1. What are ACID properties in a relational database and how are they enforced?

**A.** 
ACID represents the core guarantees of relational database transactions:

1. **Atomicity**: All operations within a transaction succeed completely or all fail and are rolled back ("all-or-nothing"). Enforced via the database undo log / rollback segment and Write-Ahead Logging (WAL).
2. **Consistency**: A transaction transitions the database from one valid state to another, preserving all schema constraints, foreign keys, unique rules, and triggers.
3. **Isolation**: Concurrent transactions execute without interfering with one another, preventing dirty reads and conflicting writes. Enforced through multi-version concurrency control (MVCC) and lock managers (two-phase locking).
4. **Durability**: Once a transaction commits, its effects persist permanently even in the event of an immediate server crash or power failure. Enforced by flushing redo logs to non-volatile storage before confirming the commit to the client.

### Q2. What is the difference between Clustered and Non-Clustered Indexes?

**A.** 
- **Clustered Index**:
  Determines the physical on-disk storage order of the actual table rows. Because table rows can only be sorted physically in one way, there can be only **one** clustered index per table (typically the Primary Key). The leaf nodes of the clustered index B+ Tree contain the full data records themselves.
- **Non-Clustered Index**:
  A separate index structure with its own B+ Tree that contains the indexed key columns and a pointer (a row locator) to the actual physical data row. In InnoDB, this locator is the clustered index primary key value; in heap-based engines, it is a physical row ID (RID). Tables can have multiple non-clustered indexes. Non-clustered queries that retrieve unindexed columns require a secondary lookup ("bookmark lookup") unless satisfied entirely by a "covering index".

### Q3. Explain the different transaction isolation levels and the read phenomena they prevent.

**A.** 
ANSI SQL defines four isolation levels offering varying trade-offs between consistency and concurrency:

1. **Read Uncommitted**: Lowest isolation. Transactions can read data modified by uncommitted transactions. Subject to **Dirty Reads**, **Non-Repeatable Reads**, and **Phantom Reads**.
2. **Read Committed**: Reads only committed data. Prevents Dirty Reads, but a query re-read in the same transaction may see changed values if another transaction commits changes in between (**Non-Repeatable Read**).
3. **Repeatable Read**: Guarantees that any row read within a transaction produces the exact same values on subsequent reads. Prevents Dirty Reads and Non-Repeatable Reads. (In MySQL InnoDB, MVCC snapshot isolation also prevents **Phantom Reads** using gap locks).
4. **Serializable**: Highest isolation. Transactions execute with serial equivalence, preventing all anomalies including **Phantom Reads** and write skew, but with substantial locking and contention overhead.

### Q4. What is the difference between B-Trees and B+ Trees, and why do databases use B+ Trees for indexes?

**A.** 
- **B-Tree**: Stores both keys and record pointers/data in all internal nodes as well as leaf nodes.
- **B+ Tree**: Stores keys and child navigation pointers in internal nodes, while **all** actual data/row pointers are stored exclusively at the leaf level. Leaf nodes are linked sequentially as a doubly linked list.

**Why Databases Use B+ Trees**:
1. **Higher Branching Factor & Lower Height**: Because internal nodes store only keys and child pointers (no heavy data payloads), more keys fit into each fixed-size page (block). This produces a much wider branching factor, reducing tree height and requiring fewer disk I/O operations per lookup.
2. **Efficient Range Scans**: In a B-Tree, range queries require in-order tree traversals across multiple levels. In a B+ Tree, the engine traverses to the first leaf node and scans linearly across the sibling linked list, making pagination and `BETWEEN` queries significantly faster.
3. **Consistent Lookup Latency**: Every lookup travels from root to leaf, providing predictable I/O depth.

### Q5. What is Database Normalization and what are 1NF, 2NF, 3NF, and BCNF?

**A.** 
Database Normalization is the systematic technique of organizing relational database schemas to minimize data redundancy and eliminate insert, update, and delete anomalies.

1. **First Normal Form (1NF)**: Every column contains atomic (indivisible) values, no repeating groups or comma-separated lists, and each record has a unique identifier (primary key).
2. **Second Normal Form (2NF)**: Meets 1NF, and all non-key attributes are fully functionally dependent on the entire primary key (no partial dependencies on a subset of a composite key).
3. **Third Normal Form (3NF)**: Meets 2NF, and no non-key attribute depends transitively on another non-key attribute (no transitive dependencies; non-keys depend only on the candidate key).
4. **Boyce-Codd Normal Form (BCNF)**: A stricter version of 3NF where for every functional dependency `X -> Y`, `X` must be a superkey.

### Q6. What is the difference between Optimistic Locking and Pessimistic Locking?

**A.** 
- **Pessimistic Locking**:
  Assumes collisions and concurrent conflicts will happen frequently. Records are locked at the database level when read (e.g., using `SELECT ... FOR UPDATE` or exclusive write locks). Other transactions attempting to access or modify the locked rows must wait until the lock is released. Best suited for high-contention environments with short-duration transactions (such as financial account transfers).
- **Optimistic Locking**:
  Assumes conflicts are rare. No database locks are held while reading. Each row maintains a `version` number or `updated_at` timestamp. When committing an update, the query checks if the version has changed: `UPDATE table SET val = :val, version = version + 1 WHERE id = :id AND version = :current_version`. If zero rows are updated, a concurrent modification occurred and the transaction rolls back or retries. Best suited for web applications with high read-to-write ratios and long user think times.

### Q7. What is Write-Ahead Logging (WAL) and how does it guarantee durability and crash recovery?

**A.** 
Write-Ahead Logging (WAL) is a core technique in database storage engines (such as PostgreSQL and SQLite) to ensure Atomicity and Durability without needing to synchronously flush dirty data pages to disk on every transaction.

**Protocol**:
No data page in the database buffer pool can be written to disk until the corresponding log records describing the modification have been written and flushed (`fsync`) to non-volatile append-only log storage.

**Crash Recovery (ARIES Protocol)**:
1. **Analysis Phase**: Identifies active transactions and dirty pages in the buffer pool at the time of the crash using checkpoint markers.
2. **Redo Phase**: Replays all logged modifications forward to restore the database state to the exact moment of the crash.
3. **Undo Phase**: Rolls back all active transactions that had not committed when the crash occurred, restoring table state to a consistent baseline.

### Q8. Explain the differences between Nested Loop Join, Hash Join, and Merge Join.

**A.** 
Relational query optimizers evaluate three primary physical join algorithms:

1. **Nested Loop Join**: For every row in the outer table, it scans the inner table for matching keys. Extremely efficient when the outer table is small and the inner table has an index on the join column (Index Nested Loop Join, O(N * log M)). Poor performance without indexes (O(N * M)).
2. **Hash Join**: In-memory hash table created on the smaller table's join keys (the build phase), followed by scanning the larger table and probing the hash table for matches (the probe phase). Optimal for large tables with equality join predicates (`=`) where no indexes exist.
3. **Sort-Merge Join**: Both inputs are sorted on the join key (or read from an ordered index), and then scanned concurrently in lockstep to find matches. Optimal when inputs are already sorted by the index or for non-equi joins.

### Q9. What is the difference between Database Sharding and Read Replication?

**A.** 
- **Read Replication (Primary-Replica)**:
  All write traffic goes to a primary database node, which asynchronously or semi-synchronously streams replication logs (binlog/WAL) to one or more read-only replicas. Read queries can be distributed across replicas. Limits: Does not scale write throughput; total storage is limited by the capacity of a single node; subject to replication lag (eventual consistency).
- **Database Sharding (Horizontal Partitioning)**:
  Splits a large dataset across multiple independent database nodes (shards) based on a shard key (e.g., `user_id` hash or geographic region). Each shard holds a unique subset of rows. Both read and write throughput scale linearly with the number of shards. Trade-offs: Cross-shard joins and distributed transactions (2PC) are complex, slow, and often require application-level coordination.

### Q10. What is the CAP theorem and how does PACELC expand upon it for distributed databases?

**A.** 
- **CAP Theorem**:
  In a distributed data store, under a network partition (**P**), you must choose between Consistency (**C**, every read receives the most recent write or an error) or Availability (**A**, every non-failing node returns a non-error response without guarantee of latest write). You cannot have both during a network split.
- **PACELC Theorem**:
  Extends CAP by recognizing that network partitions are rare, and tradeoffs occur during normal operations:
  - If there is a **P**artition: trade off **A**vailability vs **C**onsistency.
  - **E**lse (normal operation): trade off **L**atency vs **C**onsistency.

For example, MongoDB and Cassandra in default modes choose PA/EL (prioritize availability during partitions, and low latency over strong consistency during normal operations), whereas Spanner and relational databases choose PC/EC (prioritize consistency in both cases).
