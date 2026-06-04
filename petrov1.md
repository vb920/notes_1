

## 🏗️ Chapter 1: Database Architecture & Storage Fundamentals

### **The Query Lifecycle**
1.  **Transport Subsystem**: Handles the networking and protocol (e.g., MySQL or Postgres protocol) to receive queries from the client.
2.  **Query Processor**: 
    * **Parser**: Checks for correct SQL syntax.
    * **Optimizer**: The "brain" that calculates the most efficient way to fetch data (e.g., choosing an index vs. a full table scan).
3.  **Execution Engine**: Takes the plan from the optimizer and carries out the operations, interacting with the storage engine.
4.  **Storage Engine**: The component responsible for reading data from and writing data to the physical disk.



### **Workload Categories**
* **OLTP (Online Transaction Processing)**: Optimized for many small, fast transactions (e.g., an ATM withdrawal). Focuses on high concurrency and low latency.
* **OLAP (Online Analytical Processing)**: Optimized for complex, long-running queries over large datasets (e.g., "What were the total sales in Q3?"). Focuses on throughput.
* **HTAP (Hybrid Transactional/Analytical Processing)**: A modern approach trying to perform both real-time transactions and analytics on the same data.

### **Storage Layouts**
* **Row-Oriented**: Stores data as complete rows. Best for **OLTP** because you usually want to fetch a specific user's entire profile at once. 
* **Column-Oriented**: Stores each column separately. Best for **OLAP** because it allows the CPU to skip unnecessary columns and provides high compression ratios for similar data.



---

## 🌳 Chapter 1 & 2: Tree-Based Storage

### **Why BSTs Fail on Disk**
* **BST (Binary Search Tree)**: A node has only two children.
* **The Problem**: Disk access is slow. A BST is very "tall," meaning you have to jump to many different disk locations ($O(\log n)$ levels) to find a key.
* **The B-Tree Solution**: A **B-Tree** has high **Fanout** (hundreds of children per node). This makes the tree "flat," requiring only 3–4 disk seeks even for millions of rows.

### Note: “BST is tall → O(log n) levels → many disk seeks”

### **Hardware Realities**
* **HDD (Hard Disk Drive)**: Mechanical platters. Sequential access (reading data in a line) is much faster than random access (jumping around).
* **SSD (Solid State Drive)**: Uses Flash memory. 
    * **The Quirk**: You can read/write in small **Pages**, but you must erase in large **Blocks**. This leads to **Write Amplification**, where the drive does more work than requested to move data around.

---

## 🛡️ Chapter 5: Transactions & Recovery

### **ACID Properties**
* **Atomicity**: "All or nothing." If one part of a transaction fails, the whole thing is rolled back.
* **Consistency**: The database moves from one valid state to another, following all predefined rules/constraints.
* **Isolation**: Transactions running at the same time shouldn't "see" each other's partial work.
* **Durability**: Once a transaction is "committed," it stays saved even if the power goes out.

### **WAL (Write-Ahead Logging)**
To ensure **Durability** without being slow, databases don't write the whole data file to disk immediately. Instead:
1.  They append the change to a sequential **WAL** file (very fast).
2.  They update the data in RAM (**Page Cache**).
3.  Later, a background process "checkpoints" those changes into the main data files.



### **Recovery with ARIES**
**ARIES** (Algorithms for Recovery and Isolation Exploiting Semantics) is the standard recovery method:
* **Analysis**: Identifies which pages were "dirty" (modified in RAM but not disk) and which transactions were active during the crash.
* **Redo**: Replays history from the log to bring the database to the state it was in exactly at the moment of the crash.
* **Undo**: Rolls back any transactions that never finished (committed) before the crash.

---

## 🚀 Chapter 6: LSM Trees & Advanced Structures

### **LSM (Log-Structured Merge-Tree)**
Unlike B-Trees which update data in-place, **LSM Trees** always append data. This turns random writes into fast sequential writes.
* **Memtable**: An in-memory buffer where new writes go first.
* **SSTable (Sorted String Table)**: When the Memtable is full, it is flushed to disk as a sorted, immutable file.
* **Compaction**: A background process that merges multiple SSTables, keeping only the newest version of a key and removing deleted ones (**Tombstones**).

### B-Tree vs LSM (When to Use What)

| Feature        | B-Tree                  | LSM Tree                  |
|---------------|------------------------|---------------------------|
| Writes        | Slower (random I/O)    | Fast (sequential I/O)     |
| Reads         | Fast                   | Slower (merge reads)      |
| Space         | Efficient              | Higher (duplicates)       |
| Use Case      | OLTP, read-heavy       | Write-heavy, logs, streams|

### **The RUM Conjecture**
A rule stating you can only optimize two of these at once:
* **Read Overhead (R)**: How long it takes to find data.
* **Update Overhead (U)**: How long it takes to write/change data.
* **Memory/Storage Overhead (M)**: How much extra space is used for metadata or wasted "garbage" data.

### **Advanced Optimization Terms**
* **Bloom Filter**: A small, probabilistic data structure that quickly tells the engine if a key is *definitely not* in an SSTable, preventing useless disk reads.
* **CoW (Copy-on-Write)**: Instead of changing a page, the engine copies it, changes the copy, and points the parent to the new version.
* **CAS (Compare-and-Swap)**: A CPU instruction used in "lock-free" structures like **Bw-Trees** to update pointers without needing heavy database locks.

---
