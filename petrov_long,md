Here is a quick-revision summary of Chapter 1 using short, easy-to-remember phrases:

**Database Architecture & Query Lifecycle**
*   **Client/Server Model**: Database nodes act as servers, applications act as clients.
*   **4 Core Components**: 
    *   **Transport subsystem**: Handles communication.
    *   **Query processor**: Parses, validates, and optimizes.
    *   **Execution engine**: Executes the query locally or remotely.
    *   **Storage engine**: Reads/writes data to storage.
*   **Query Lifecycle**: Client -> Transport -> Query Processor -> Execution Engine -> Storage Engine -> Execution Engine -> Transport -> Client.
*   **Non-linear execution**: The execution engine and storage engine constantly interact back-and-forth during complex queries (e.g., sorting, grouping).

**Storage Engine Components**
*   **Transaction Manager**: Schedules transactions and prevents inconsistencies.
*   **Lock Manager**: Controls concurrency and data integrity.
*   **Access Methods**: Organizes data on disk (e.g., B-Trees, LSM trees).
*   **Buffer Manager**: Caches data pages in memory.
*   **Recovery Manager**: Recovers database state after failures.

**Database Categories (Workloads)**
*   **OLTP**: Optimized for transactions and user-facing requests.
*   **OLAP**: Optimized for analytics and aggregations.
*   **HTAP**: Hybrid of both, but essentially a compromise.

**Storage Mediums & Durability**
*   **In-Memory**: Stores data in RAM for speed; uses disk only for recovery/logging.
*   **Disk-Based**: Stores data on disk; uses RAM for caching/temporary storage.
*   **Durability in Memory**: Maintained via **Write-Ahead Logging** (sequential logs) and a **Backup Copy** on disk.
*   **Checkpointing**: Asynchronously applying log batches to the backup copy to create a snapshot, then discarding the applied logs so the file doesn't grow too large.

**Storage Layouts**
*   **Row-Oriented**: Stores complete rows together. Best for OLTP (e.g., fetching a specific user record). Improves spatial locality for rows.
*   **Column-Oriented**: Stores entire columns together. Best for OLAP (e.g., fetching all ages to find an average). Improves cache utilization, compression ratios, and vectorization.

**Data Files vs. Index Files**
*   **Data Files**: Hold the actual data. Three organization types:
    *   **Heap files**: Unordered/write-order.
    *   **Hash files**: Bucketed using a hash of the key.
    *   **Index Organized Tables (IOT)**: Data stored directly inside the index.
*   **Index Files**: Hold metadata to locate records. Depending on the data file type, indexes contain file offsets (heap), bucket IDs (hash), or primary keys (IOT).

**Index Types & Access**
*   **Primary Index**: Built on the data file/primary key.
*   **Secondary Index**: Any other index.
*   **Clustered**: Data matches the index order.
*   **Non-clustered**: Data order differs from the index order.
*   **Direct Access (Offsets)**: Faster reads, but pointers break/need updates when data moves.
*   **Indirect Access (Primary Key)**: Slower reads (more disk seeks), but immune to data movement overhead.

**Storage Structure Variables**
1.  **Buffering**: Is data held in memory before hitting disk?
2.  **Immutability**: Is data updated in-place or append-only?
3.  **Ordering**: Is data stored sorted by key or unordered?

**Binary Search Trees (BST)**
*   **Purpose**: Sorted, in-memory data structures used for efficient key-value lookups.
*   **BST Invariant**: A node's left subtree contains smaller keys, and its right subtree contains greater or equal keys.
*   **Balancing**: Unbalanced trees risk slow O(n) lookups; balanced trees maintain an O(log n) height by rotating nodes.

**Why BSTs Fail on Disk**
*   **Low Fanout**: Nodes have few children, leading to taller trees and too many expensive disk seeks.
*   **Poor Locality**: Parent and child nodes might not be written to the same disk page, increasing seek times.

**Storage Hardware Mediums**
*   **HDDs (Hard Drives)**: Mechanical and slow at positioning the read/write head; data is read/written in minimal units called "sectors".
*   **SSDs (Solid State Drives)**: No moving parts, organized into dies, planes, blocks, and pages.
*   **SSD Quirk**: Data is read/written at the "page" level but must be erased at the "block" level due to the physical properties of quantum tunneling and voltage needs.

**B-Tree Architecture**
*   **Disk-Optimized**: Based on BSTs but feature a much higher fanout, resulting in shorter trees and fewer disk seeks.
*   **Node Layout**: Keys and child pointers are packed into a single block or consecutive blocks.
*   **Internal Sorting**: All keys inside a B-Tree node are kept sorted, allowing quick binary searches within the node itself.
*   **Lookup Complexity**: Search takes log M time, where M is the total number of nodes.

**Key B-Tree Components**
*   **Separator Keys**: Also called divider cells, these keys route lookups to the correct child pointers based on value ranges.
*   **Sibling Pointers**: Links connecting leaf nodes together like a linked list, which dramatically speeds up range queries by avoiding parent-node backtracking.

**B-Tree Maintenance**
*   **Node Splitting (Overflow)**: When a node gets entirely full, it splits into two separate nodes, and the middle key is pushed upward into the parent node.
*   **Node Merging (Underflow)**: When a deletion leaves a node nearly empty, elements are shifted from a sibling node, or the two nodes are merged completely while updating the parent.

**Terminology Quirk: Pages vs. Nodes vs. Blocks**
*   **Interchangeable Terms**: Databases use B-Trees to organize fixed-size chunks of data called "pages", so the terms "node" and "page" are often used interchangeably.
*   **The Discrepancy**: While an SSD defines a "page" as a smaller unit inside a "block", databases define a "page" as the minimum bytes they read/write, which is usually a multiple of the underlying hardware or filesystem block.

**Binary Search Trees (BST)**
*   **Purpose**: Sorted, in-memory data structures used for efficient key-value lookups.
*   **BST Invariant**: A node's left subtree contains smaller keys, and its right subtree contains greater or equal keys.
*   **Balancing**: Unbalanced trees risk slow O(n) lookups; balanced trees maintain an O(log n) height by rotating nodes.

**Why BSTs Fail on Disk**
*   **Low Fanout**: Nodes have few children, leading to taller trees and too many expensive disk seeks.
*   **Poor Locality**: Parent and child nodes might not be written to the same disk page, increasing seek times.

**Storage Hardware Mediums**
*   **HDDs (Hard Drives)**: Mechanical and slow at positioning the read/write head; data is read/written in minimal units called "sectors".
*   **SSDs (Solid State Drives)**: No moving parts, organized into dies, planes, blocks, and pages.
*   **SSD Quirk**: Data is read/written at the "page" level but must be erased at the "block" level due to the physical properties of quantum tunneling and voltage needs.

**B-Tree Architecture**
*   **Disk-Optimized**: Based on BSTs but feature a much higher fanout, resulting in shorter trees and fewer disk seeks.
*   **Node Layout**: Keys and child pointers are packed into a single block or consecutive blocks.
*   **Internal Sorting**: All keys inside a B-Tree node are kept sorted, allowing quick binary searches within the node itself.
*   **Lookup Complexity**: Search takes log M time, where M is the total number of nodes.

**Key B-Tree Components**
*   **Separator Keys**: Also called divider cells, these keys route lookups to the correct child pointers based on value ranges.
*   **Sibling Pointers**: Links connecting leaf nodes together like a linked list, which dramatically speeds up range queries by avoiding parent-node backtracking.

**B-Tree Maintenance**
*   **Node Splitting (Overflow)**: When a node gets entirely full, it splits into two separate nodes, and the middle key is pushed upward into the parent node.
*   **Node Merging (Underflow)**: When a deletion leaves a node nearly empty, elements are shifted from a sibling node, or the two nodes are merged completely while updating the parent.

**Terminology Quirk: Pages vs. Nodes vs. Blocks**
*   **Interchangeable Terms**: Databases use B-Trees to organize fixed-size chunks of data called "pages", so the terms "node" and "page" are often used interchangeably.
*   **The Discrepancy**: While an SSD defines a "page" as a smaller unit inside a "block", databases define a "page" as the minimum bytes they read/write, which is usually a multiple of the underlying hardware or filesystem block.

**Page Headers & Magic Numbers**
*   **Page Header**: Stores crucial metadata like the number of cells, page size, and layout version.
*   **Magic Numbers**: Multi-byte blocks placed in the header to easily identify the file type and version.

**Navigating Nodes (Pointers & Links)**
*   **Sibling Links**: Connect child nodes directly horizontally (e.g., child1 -> child2), which speeds up navigation by avoiding trips back to the parent,. The downside is the extra overhead to update them during node splits and merges.
*   **Rightmost Pointers**: Stored separately in the header to successfully pair the final separator key with its corresponding rightmost child pointer,.
*   **Node High Keys**: An alternative to rightmost pointers that explicitly stores the highest key in the node to simplify edge case handling.

**Overflow Pages**
*   **Purpose**: Handles payloads that are too large to fit within a single standard page.
*   **Structure**: Functions as a linked list of pages. The primary page stores a pointer to the first overflow page, which stores the page ID of the next overflow page, and so on.

**Searching & Traversing**
*   **Binary Search**: Used to find the "insertion point", which is the index of the first element greater than the searched key. It does this by checking the page's sorted cell offsets,.
*   **Breadcrumbs**: A common technique that uses a stack to remember the exact traversal path (node pointers and insertion points) from the root down to the leaf,. This makes it easy to propagate splits and merges back up the tree without paying the overhead of storing permanent parent pointers in every node,.

**Optimizing Writes & Rebalancing**
*   **Rebalancing**: Delays expensive splits and merges by shifting elements between neighboring nodes to maintain high occupancy. For example, B* Trees balance elements until siblings are full, and then split 2 nodes into 3 (instead of 1 into 2).
*   **Right-Only Appends**: An optimization for inserting sequentially increasing, auto-incremented IDs (often called "fastpath" in Postgres or "quickbalance" in SQLite),.
*   **Bulk Loading**: Used for pre-sorted data to compose the tree from the bottom-up,. This allows pages to be completely filled to maximum occupancy without the overhead of individual inserts,.

**Compression**
*   **The Trade-off**: Higher compression saves disk space and disk read times, but demands more CPU and RAM. Lower compression uses less CPU but requires more disk space and I/O.
*   **Where to apply**: Compressing at the page level or data level is effective, but compressing at the whole-file level is highly impractical because reading a single page would require decompressing the entire file.

**Maintenance & Defragmentation**
*   **Addressability**: When an element is deleted, it is not immediately erased or zero-filled. Its cell offset is simply removed from the page header, turning it into "non-addressable" garbage data that will eventually be overwritten.
*   **Page Splits**: When a page splits, only the cell offsets are moved; the actual old data cells are left behind as non-addressable garbage to save work.
*   **Compaction / Vacuum**: A background maintenance process that reclaims dead space by completely rewriting pages, defragmenting them, and adding newly freed on-disk pages to a persistent "freelist".

Here is a quick-revision summary of Chapter 5 using short, easy-to-remember phrases:

**Transactions & ACID**
*   **Transaction**: An indivisible unit of work that either succeeds completely or fails completely.
*   **ACID Properties**: **A**tomicity (all or nothing), **C**onsistency (database remains in a valid state), **I**solation (concurrent transactions do not interfere), and **D**urability (committed changes are permanent).
*   **4 Core Components**: Transaction Manager (schedules), Lock Manager (prevents concurrent access), Page Cache (caches disk pages in RAM), and Log Manager (writes changes to the log).

**Buffer Management (Page Cache)**
*   **Page States**: 
    *   **Paged in**: Read from disk to memory.
    *   **Dirty**: Modified in memory, but not yet saved to disk.
    *   **Flushed**: Written safely to disk.
    *   **Evicted**: Removed from the cache.
*   **Pinning**: Locking specific pages (like upper-level B-Tree nodes) in the cache so they cannot be evicted.
*   **Eviction Algorithms**: FIFO (oldest first), LRU (least recently used), Clock (circular buffer using access bits), and LFU (least frequently used).
*   **Belady’s Anomaly**: The counterintuitive situation where increasing the cache size actually increases the number of evictions (due to a poor eviction policy).

**Recovery & Write-Ahead Logging (WAL)**
*   **WAL Purpose**: An append-only log that buffers updates and allows lost in-memory changes to be reconstructed after a crash.
*   **LSN**: Log Sequence Number, a unique monotonic ID for every log record.
*   **Checkpoints**: Reduce recovery time. *Sync* checkpoints flush all dirty pages, while *Fuzzy* checkpoints do it asynchronously and just update a pointer.
*   **Flush Policies**:
    *   **Steal**: Dirty pages *can* be flushed before commit, requiring "undo" logs to roll back if needed.
    *   **No-Force**: Transactions *can* commit before dirty pages are flushed, requiring "redo" logs to prevent data loss in a crash.
*   **ARIES**: A standard Steal/No-Force recovery algorithm featuring an Analysis phase, a Redo phase (physical), and an Undo phase (logical).

**Concurrency Control**
*   **Optimistic (OCC)**: Assumes conflicts are rare; executes transactions in Read, Validation, and Write phases, aborting if a conflict is found.
*   **Pessimistic (PCC)**: Assumes conflicts are frequent; locks data upfront using schemes like Two-Phase Locking (2PL) or Timestamp Ordering.
*   **MVCC (Multi-version)**: Keeps multiple versions of data so reads don't block writes; ideal for Snapshot Isolation.

**Anomalies & Isolation Levels**
*   **Read Anomalies**: 
    *   **Dirty Read**: Reading uncommitted data.
    *   **Non-repeatable Read**: Reading the same row twice and getting different results.
    *   **Phantom Read**: Repeating a range query and getting a different set of rows.
*   **Write Anomalies**: 
    *   **Lost Update**: Overwriting someone else's changes.
    *   **Dirty Write**: Modifying uncommitted data.
    *   **Write Skew**: Individual operations are valid, but together they violate an invariant.
*   **Snapshot Isolation**: Transactions read from a snapshot taken at their start; prevents lost updates but remains vulnerable to write skew.

**Locks vs. Latches**
*   **Locks**: Guard *logical* database objects (like keys), held for the entire transaction, and are heavyweight.
*   **Latches**: Guard *physical* storage structures (like pages), held only briefly, and are lightweight.
*   **Deadlocks**: Transactions waiting on each other forever. Handled by wait-for graphs, timeouts, or priority rules (Wait-Die / Wound-Wait).

**Tree Navigation Optimizations**
*   **Latch Crabbing (Coupling)**: Acquiring a child latch and immediately releasing the parent latch to maximize concurrency.
*   **Blink-Trees**: Use sibling pointers for a "half-split" state, which completely avoids the need to lock parent nodes during splits.
Here is a quick-revision summary of Chapter 6 using short, easy-to-remember phrases:

**Copy-on-Write (CoW) B-Trees**
*   **The Mechanism**: Page updates create a parallel tree hierarchy by copying the page, modifying it, and then updating the root pointer to the new tree. 
*   **Pros**: Readers and writers do not block each other, readers need no synchronization, and the tree remains safe from crash corruption.
*   **Cons**: Requires higher space and CPU overhead.
*   **Implementation**: LMDB (used by OpenLDAP) is a CoW engine that holds exactly 2 versions of the tree at a time.

**Lazy B-Trees & WiredTiger**
*   **The Mechanism**: Buffers updates in memory to delay disk propagation.
*   **WiredTiger (MongoDB default)**: Maintains clean/dirty in-memory pages with an original on-disk image. Dirty pages get an "update buffer" implemented as a skiplist.
*   **Reconciliation**: During a flush, the update buffer is merged with the on-disk page contents before writing.

**Lazy-Adaptive (LA) Trees**
*   **The Mechanism**: Groups nodes into subtrees and attaches an update buffer to the top node of each subtree to batch operations.
*   **Propagation**: When a buffer fills, changes are recursively pushed down to lower-level buffers until they hit the leaves, heavily reducing random disk accesses.

**FD-Trees (Flash-Drive Trees)**
*   **The Mechanism**: Built to minimize random write I/O. Uses a small mutable "head tree" that acts as an in-place buffer, which flushes into immutable, logarithmically sized sorted runs.
*   **Fractional Cascading**: A search optimization that bridges gaps between levels by pulling alternate elements to an upper level, making searches through cascaded arrays much faster.

**Bw-Trees (Buzzword Trees)**
*   **The Goal**: Solves write amplification, space amplification, and latching complexity.
*   **Lock-Free**: Uses a mapping table and single Compare-and-Swap (CAS) operations to install pointers without locks.
*   **Update Chains**: Instead of rewriting nodes, updates are appended as "delta nodes" linked to a "base node".
*   **Garbage Collection**: Uses Epoch-Based Reclamation to safely clean up unreferenced nodes and deltas only after all older readers have finished.

**Bw-Tree Structural Modification Operations (SMOs)**
*   **Splits**: A 2-step process that appends a "split delta node" to the splitting node, then updates the parent to point to the new sibling.
*   **Merges**: A 3-step process that marks the right sibling for deletion with a "remove delta node", creates a "merge delta node" on the left sibling, and updates the parent.

**Cache-Oblivious B-Trees**
*   **The Mechanism**: Adapts to diverse memory hierarchies for optimal performance without needing manual tuning parameters.
*   **Layouts**: Uses a **van Emde Boas layout** to ensure contiguous memory storage for static trees, and a packed array with density-based gaps to efficiently handle dynamic inserts/updates.

**LSM Trees (Log-Structured Merge-Trees)**
*   **The Mechanism**: Uses in-memory buffering and append-only disk writes to achieve fast, sequential write operations. 
*   **Immutability**: Files are never modified in-place, which avoids expensive disk seeks and simplifies concurrent access since readers and writers don't intersect.

**3 Core Components**
*   **Memtable**: An in-memory, mutable buffer that receives incoming writes and serves recent reads.
*   **WAL (Write-Ahead Log)**: An append-only log on disk where records are saved before the memtable to guarantee durability.
*   **Disk-Resident Tables (SSTables)**: Immutable, sorted files written to disk once the memtable fills up; used strictly for reads.

**Flushing Process**
*   **Memtable Switch**: When full, a new memtable is allocated for incoming writes, and the old one enters a "flushing" state (still available for reads).
*   **Disk Write**: The old memtable is flushed to disk as a new SSTable.
*   **WAL Truncation**: Once the SSTable is safely on disk, the old memtable is discarded and the corresponding section of the WAL is deleted.

**Handling Updates & Deletes**
*   **Updates**: Simply appended as new records; the redundant older records are reconciled later during reads.
*   **Deletes (Tombstones)**: Deletions are also handled as appends using special "tombstone" markers, which prevent older versions of the record from being resurrected during a read.

**Lookups & Read Mechanics**
*   **Merge-Iteration**: Because tables are internally sorted, databases use a multiway merge-sort algorithm (like a min-heap) to read from multiple tables simultaneously.
*   **Reconciliation**: If multiple versions of a key exist, the database checks metadata (like timestamps) to ensure the newest version takes precedence.

**Compaction Strategies**
*   **Purpose**: A background process that merges disk-resident tables to reduce file count and purge dead/overwritten data.
*   **Leveled Compaction**: Tables are separated into levels with target sizes that grow exponentially; merges happen between overlapping key ranges.
*   **Size-Tiered Compaction**: Tables are grouped simply by their file size; smaller tables are recursively merged into larger ones.

**The RUM Conjecture & Amplification**
*   **RUM Overhead**: You can only optimize two out of three: **R**ead overhead, **U**pdate overhead, and **M**emory overhead. LSM Trees optimize for Updates, paying the price in Read overhead.
*   **Amplification Types**: 
    *   *Read*: Checking multiple tables for one lookup.
    *   *Write*: Compaction constantly rewriting the same data.
    *   *Space*: Storing multiple obsolete versions of the same key.

**Read Optimizations**
*   **Bloom Filters**: Probabilistic data structures that tell you if a key is "definitely absent" or "possibly present." This saves the database from wasting disk I/O searching tables that don't contain the key.
*   **Skiplists**: In-memory linked lists with layered "bridge" pointers that allow readers to skip over chunks of data for fast $O(log\ n)$ searches.

**Alternative LSM Architectures**
*   **Unordered LSM Storage**: Skips the memtable and sorting entirely, writing records in raw insertion order.
*   **Bitcask**: Writes directly to log files and keeps an in-memory "keydir" (hash map) pointing to the exact disk offsets. Incredibly fast for point queries, but cannot do range queries.
*   **WiscKey**: Keeps only the small keys sorted in the main LSM tree, while dumping the large data payloads into an unsorted, append-only "Value Log" (vLog). This heavily speeds up compaction by moving less data.
