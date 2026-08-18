# 📖 ByteForge Database Engine Story: Amazon-Style Interview Guide

This guide presents ByteForge as a story-driven project for an SDE-2 interview. It connects the project to Amazon’s Leadership Principles while explaining the engineering journey from writing rows to CSV files to building a typed, indexed, and recovery-aware storage engine in Go.

---

## 🏗️ The Storage Engine Narrative

### Setting the Stage

> [!IMPORTANT]
> **Leadership Principles:** Dive Deep, Learn and Be Curious, Ownership
>
> “I was comfortable using databases from application code, but I wanted to understand what happens below SQL and database drivers. I started with a simple question: what changes when a program stops writing rows to a CSV file and starts behaving like a database?
>
> A CSV writer can append text, but a database engine must preserve types, record boundaries, schema metadata, physical locations, indexes, and recovery state. I built ByteForge in Go to explore that complete data path.”

---

## 1. Project Focus: Data Path First

ByteForge primarily focuses on the **database data path**.

The data path is responsible for moving and transforming data through the engine:

```text
Application values
        ↓
Schema validation
        ↓
TLV binary encoding
        ↓
Table and page storage
        ↓
Primary and full-text indexes
        ↓
Record parsing and query results
```

The engine also contains the foundation of a **durability path** through its Write-Ahead Log and restoration flow.

The next architectural phase is the **control path**, which will formalize how the engine selects, coordinates, explains, and measures these data-path operations.

### Data Path

The data path answers questions such as:

- How are typed values represented as bytes?
- How are record boundaries preserved?
- How is table schema restored after a restart?
- How does a primary key map to a physical record location?
- How can multiple indexes reference the same base record?
- How does the engine reconstruct usable state when it opens?

### Control Path

The control path answers questions such as:

- Which access path should a query use?
- How should execution plans be represented?
- How can the engine explain why it selected an index or table scan?
- When should indexes be rebuilt?
- When should recovery checkpoints be created?
- Which metrics should drive storage and cache policy decisions?

> ByteForge is a **data-path-first database engine**. The control path is the next evolution built on top of its storage and access-path abstractions.

---

## 2. Storage Format and Record Encoding

**Leadership Principle: Invent and Simplify**

> “One of my first design decisions was to use Type-Length-Value encoding for typed record serialization. I wanted the binary representation to make types and record boundaries explicit rather than delegating serialization to CSV, JSON, or a Go-specific object format.”

ByteForge uses **TLV encoding**:

```text
+-------------+--------------------+----------------------+
| Type: 1 B   | Length: 4 B        | Value: Length bytes  |
+-------------+--------------------+----------------------+
```

The encoding layer supports scalar values such as:

- `int64`
- `int32`
- `byte`
- `bool`
- `string`

It also provides encoders for nested structures such as:

- Lists
- Map-like objects
- Index entries
- Column definitions
- WAL metadata

### Why TLV?

TLV provided several advantages for the first implementation:

- Explicit type information
- Clear value boundaries
- Reusable marshaling and unmarshaling components
- Support for heterogeneous records
- A binary format independent of CSV delimiters
- A common encoding foundation for tables, indexes, and WAL structures

### The trade-off

TLV introduces metadata overhead because each value carries a type and length header.

It also requires sequential decoding when the location of a particular field is not already known.

Therefore, the design should not be described as “TLV versus slotted pages.” They solve different problems:

- **TLV** defines how fields inside a record are encoded.
- **Slotted pages** define how complete records are located inside a page.

A future ByteForge version can store TLV-encoded records inside slotted pages.

### Interview explanation

> “I chose TLV because it made types and record framing explicit and gave me reusable binary serialization primitives. The trade-off is metadata overhead and sequential field decoding. My future page-layout upgrade would preserve TLV as the row format while adding a slot directory for direct record-offset access.”

---

## 3. Page-Oriented Storage

**Leadership Principles: Dive Deep, Invent and Simplify**

ByteForge groups records under page metadata and associates index entries with page positions.

The current implementation uses a deliberately small page-capacity constant during development:

```go
const PageSize = 128
```

The purpose of the small value is to exercise page-boundary and page-allocation behavior with a small number of records.

It should not be presented as an SSD-aligned production page size.

### Why use a small development page capacity?

With large pages, a small test dataset may never trigger page-selection or boundary conditions. A small capacity makes those situations easier to observe during development.

It helps exercise:

- Page selection
- Record-capacity checks
- Page-position calculation
- Multiple-page reads
- Cache keys based on page position
- Index references across pages

### Future page design

A production-oriented version would make page size configurable and benchmark candidates such as 4 KB, 8 KB, and 16 KB.

The selection would depend on:

- Record-size distribution
- Expected access patterns
- Filesystem behavior
- Operating-system page size
- Storage hardware
- Read amplification
- Internal fragmentation
- Cache efficiency

No single page size should be described as universally aligned with SSD internals.

### Slotted-page evolution

A future slotted-page layout could contain:

```text
+---------------------------+
| Page Header               |
| - Page ID                 |
| - Used Space              |
| - Slot Count              |
+---------------------------+
| Slot Directory            |
| - Record Offset           |
| - Record Length           |
+---------------------------+
| Free Space                |
+---------------------------+
| TLV-Encoded Records       |
+---------------------------+
```

If the index knows a page ID and slot ID, the slot directory enables direct access to the corresponding record offset.

### Interview explanation

> “The current engine uses page-position-based addressing and a small development page capacity to make boundary behavior visible with compact test data. The next storage upgrade is a fixed-size physical page abstraction with a slot directory, stable page identifiers, and TLV-encoded records stored as page payloads.”

---

## 4. Primary-Key Indexing

**Leadership Principles: Learn and Be Curious, Deliver Results**

ByteForge integrates an in-memory B-tree for primary-key access.

The logical mapping is:

```text
int64 primary key → physical page position
```

This allows an eligible primary-key query to locate the relevant page without performing a complete table scan.

### Persistence model

The B-tree itself is an in-memory structure. ByteForge persists the key-to-position mappings in a separate index file and reconstructs the B-tree when the database opens.

This distinction is important:

- The current design uses an **in-memory B-tree**.
- It does not implement a disk-resident B-tree with custom node pages.
- B-tree node splitting and balancing are provided by the external B-tree library.
- ByteForge owns the persistence and reconstruction of key-to-location mappings.

### Read and write trade-off

The in-memory lookup path provides logarithmic search behavior:

```text
Primary-key lookup: O(log n)
```

The current persistence strategy serializes the complete index mapping when it is updated. Therefore, the important write-path trade-off is fast in-memory lookup versus full index-file serialization during persistence.

### Future alternatives

#### Page-based persistent B+ tree

Useful when the workload requires predictable point reads, ordered traversal, range scans, and incremental node persistence.

Costs include page splits, node rebalancing, more complex recovery, and concurrent node management.

#### LSM-style index

Useful when the workload requires high write throughput, sequential writes, and batched persistence.

Costs include read amplification, space amplification, background compaction, and more complex read paths.

### Interview explanation

> “ByteForge currently uses an in-memory B-tree for efficient primary-key lookup and persists the key-to-page-position mappings separately. The read path is logarithmic in memory, while index persistence currently rewrites the serialized mappings. That makes persistent-index write amplification an important future optimization area.”

---

## 5. Full-Text Access Path

**Leadership Principles: Customer Obsession, Invent and Simplify**

ByteForge supports a separate full-text access structure for configured string columns.

The mapping is conceptually:

```text
indexed text value → list of (page position, record ID)
```

This gives the engine more than one physical access path:

- Primary-key B-tree lookup
- Full-text lookup
- Full table scan

### Why use a separate index?

Primary-key and text queries serve different access patterns. Keeping these structures separate provides clear responsibilities, independent persistence, easier inspection during development, and a foundation for future secondary indexes.

### Trade-off

A logical mutation may affect several persisted structures:

```text
Table file
Primary index
Full-text index
WAL
```

The broader engineering challenge is coordinating those structures so that they represent the same logical state.

### Interview explanation

> “Adding a second access path taught me that a database row can have several physical representations. The base record is authoritative, while indexes provide alternative ways to reach it. This is also where recovery becomes more interesting because all related structures must remain logically synchronized.”

---

## 6. Write-Ahead Logging and Recovery

**Leadership Principles: Ownership, Dive Deep**

ByteForge implements a Write-Ahead Log and restoration path for insert operations.

The logical sequence is:

```text
Encode operation
      ↓
Append WAL record
      ↓
Write table record
      ↓
Update indexes
      ↓
Record recovery progress
```

The current implementation uses ordinary Go file writes through `os.File`.

### Current focus

The WAL was introduced to explore operation framing, separate recovery metadata, WAL-before-table write ordering, database startup restoration, and recovery as part of the table lifecycle.

### Buffered I/O trade-off

A successful `os.File.Write` means the operating system accepted the bytes. It does not necessarily mean the bytes reached stable physical storage.

A strict durability mode would introduce an explicit synchronization point:

```text
Append WAL entry
      ↓
Sync WAL
      ↓
Apply table and index mutation
      ↓
Advance recovery metadata
```

### `fsync` and group commit

Calling `fsync` or `File.Sync` for every individual mutation can increase commit latency. Group commit can improve throughput by allowing several operations to share one durable synchronization.

The appropriate batch size and maximum wait time should be selected through measurement rather than assumed in advance.

Useful measurements include commit throughput, P50, P95, and P99 commit latency, WAL bytes per logical byte, recovery duration, and commits per synchronization.

### LSN and checkpoint evolution

ByteForge currently identifies WAL entries using generated IDs. A future recovery model can add a monotonically increasing **Log Sequence Number**.

A checkpoint can record the latest applied WAL position. Recovery can then begin from that boundary rather than searching the complete history.

LSNs provide ordering, but replay becomes idempotent only when the engine also records which LSN has already been applied to a page or record.

```text
Apply WAL record only when WAL.LSN > Page.PageLSN
```

### Interview explanation

> “The current WAL establishes the logical operation and restoration path. The next durability milestone is explicit WAL synchronization, followed by monotonic LSNs, checkpoint positions, checksummed records, and idempotent replay. Group commit would then optimize the synchronization cost after the correctness boundary is established.”

---

## 7. LRU Page Cache

**Leadership Principles: Deliver Results, Dive Deep**

ByteForge includes an LRU page cache for indexed reads.

The indexed read path is:

```text
Index lookup
      ↓
Page position
      ↓
LRU lookup
      ↓
Disk read on cache miss
      ↓
Record parsing
```

### Current scope

The current structure is best described as an **LRU page cache**, not a complete database buffer pool.

A complete buffer pool would typically also manage pin counts, dirty-page state, page frames, eviction eligibility, concurrent page ownership, background flushing, and writeback policy.

### Cache trade-off

A pure LRU policy can suffer from scan pollution. A future scan-resistant policy could use probationary and protected segments, with pages promoted only after reuse.

### Interview explanation

> “ByteForge currently uses an LRU page cache for indexed reads. A future buffer-pool implementation would add dirty-page tracking, pin counts, and safe eviction. I would also evaluate scan-aware admission so one large sequential query does not remove the working set of frequently accessed pages.”

---

## 8. Query Routing and the Control Path

**Leadership Principles: Invent and Simplify, Think Big**

ByteForge contains a simple access-path selection rule:

```text
WHERE contains primary ID?
        ├── Yes → B-tree lookup
        └── No
             ↓
WHERE contains full-text indexed column?
        ├── Yes → Full-text lookup
        └── No  → Full table scan
```

This is procedural query routing, not yet a formal optimizer.

### Existing execution metadata

Query results expose the selected access type, rows inspected, and page-cache usage information. This is a useful foundation for an `EXPLAIN`-style interface.

### Future control-path evolution

1. Represent execution choices as explicit plan nodes such as `TableScan`, `PrimaryKeyLookup`, `FullTextLookup`, and `Filter`.
2. Add EXPLAIN-style output showing the selected path, indexed predicates, residual filters, rows inspected, and cache behavior.
3. Formalize deterministic access-selection rules in a Rule-Based Optimizer.
4. Add a Cost-Based Optimizer after collecting reliable cardinality and access-cost statistics.

### Interview explanation

> “The current implementation has procedural access-path selection and execution metadata. I would evolve this first into explicit plan nodes and an EXPLAIN-style representation, then into a rule-based planner. A cost-based optimizer would come later after the engine collects reliable cardinality and access-cost statistics.”

---

## 🎯 SDE-2 Trade-Off Analysis

### Core Architectural Trade-Offs

| Choice | Appropriate when | Engineering cost |
| :--- | :--- | :--- |
| **In-memory B-tree** | Fast primary-key lookup is required within one process. | The tree must be reconstructed after restart, and persistence requires a separate strategy. |
| **Disk-resident B+ tree** | Point lookups and ordered range scans must work directly from persistent pages. | Requires node-page management, splitting, rebalancing, recovery, and concurrency control. |
| **LSM tree** | The workload is write-heavy and benefits from sequential persistence. | Introduces compaction, read amplification, and space amplification. |
| **TLV encoding** | Values need explicit types and self-describing boundaries. | Adds per-value metadata and sequential decoding overhead. |
| **Slotted pages** | The engine needs stable record references and direct offset access within a page. | Requires slot management, compaction, free-space tracking, and record-relocation logic. |
| **Separate index files** | Storage structures should remain independently inspectable and loadable. | A logical mutation must coordinate multiple persisted files. |
| **LRU cache** | Recently used pages are likely to be reused. | Sequential scans can pollute the cache without admission control. |

### Durability and Recovery Trade-Offs

| Choice | Appropriate when | Engineering cost |
| :--- | :--- | :--- |
| **OS-buffered writes** | Development prioritizes a simple and fast logical write path. | A successful write is not a stable-storage durability guarantee. |
| **Per-operation WAL sync** | Each acknowledged operation requires a strong persistence boundary. | Higher commit latency and lower throughput. |
| **Group commit** | Multiple operations can share one durable WAL synchronization. | Adds batching delay and coordination complexity. |
| **LSN-based recovery** | WAL ordering and replay boundaries must be explicit. | Requires persistent LSN metadata and applied-state tracking. |
| **Checkpointing** | Recovery should avoid scanning the complete WAL history. | Introduces checkpoint coordination and temporary I/O pressure. |
| **Index reconstruction** | Correctness is more important than fast startup. | Increases recovery time for large tables. |
| **Incremental index recovery** | Startup time must remain bounded for large datasets. | Requires stronger index logging and multi-file recovery coordination. |

### Concurrency Trade-Offs

The current ByteForge implementation should be presented as a single-process prototype rather than a concurrent transactional engine.

| Choice | Appropriate when | Engineering cost |
| :--- | :--- | :--- |
| **Global mutex** | The first priority is a simple correctness baseline. | Limits concurrency because independent operations share one lock. |
| **Read-write mutex** | Reads should proceed concurrently while writes remain exclusive. | Writers can still become bottlenecks under heavy contention. |
| **Page-level latching** | Independent pages should be accessed concurrently. | Requires a stable locking hierarchy and deadlock analysis. |
| **Latch crabbing** | A custom concurrent B+ tree needs safe node traversal and splitting. | Complex implementation and race-condition testing. |
| **Two-phase locking** | Transaction isolation is needed with pessimistic conflict management. | Can produce deadlocks and requires a lock manager. |
| **MVCC** | Readers should observe snapshots without blocking writers. | Requires row versions, visibility rules, garbage collection, and vacuuming. |

---

## 📊 Observability and Engineering Health

A storage engine should be evaluated through measurements rather than fixed universal thresholds.

### Data-path metrics

- Record encode and decode latency
- Bytes written per logical row
- Page utilization
- Records per page
- Full-scan throughput
- Primary-key lookup latency
- Full-text lookup latency

### Cache metrics

- Cache hit and miss ratios
- Eviction count
- Reuse distance
- Indexed-read latency on hit and miss
- Scan-related cache pollution

### Recovery metrics

- WAL append and synchronization latency
- Recovery duration
- Number of WAL records replayed
- Checkpoint age
- Incomplete WAL-tail count
- Index reconstruction duration

### Write-amplification metrics

```text
Write Amplification = Physical bytes written / Logical bytes inserted
```

For ByteForge, this measurement should include table-file writes, primary-index persistence, full-text-index persistence, WAL writes, and recovery-metadata writes.

### Interview explanation

> “I would not assume that an 80% cache-hit ratio is always healthy or unhealthy. The correct target depends on the workload. I would compare hit ratio with lookup latency, scan behavior, working-set size, and eviction patterns before changing cache capacity or policy.”

---

## 🎙 Deep-Dive Interview Playbook

### Why did you start from CSV?

> “CSV provided the simplest baseline for persisting rows. Once I needed typed values, stable record boundaries, schema reconstruction, indexed lookup, deletion, and recovery, it became clear that I was no longer solving a file-export problem. I was building the data path of a database.”

### Why focus on the data path first?

> “Every query planner eventually invokes a storage access path. I wanted to understand and implement those foundational mechanisms before adding SQL parsing or optimizer abstractions.”

### Why use TLV?

> “TLV made types and record framing explicit and allowed me to reuse the same encoding primitives across records, schema metadata, indexes, and WAL structures. The trade-off is metadata overhead and sequential decoding.”

### Is the B-tree stored on disk?

> “The current B-tree is an in-memory primary access structure. ByteForge persists the key-to-page-position mappings separately and reconstructs the tree when the database opens.”

### Why use page-position-based addressing?

> “It separates a record’s logical identity from its physical location. The primary index can map an ID to the relevant position, avoiding a complete table scan for eligible point queries.”

### What does the WAL guarantee?

> “The current WAL provides the logical logging and restoration foundation. Because it uses ordinary buffered file writes, I describe it as a recovery prototype rather than claiming stable-storage durability. The next milestone is explicit WAL synchronization, followed by LSNs, checksums, checkpointing, and idempotent replay.”

### Why not use `O_DIRECT`?

> “Direct I/O is primarily a cache-management and alignment decision. It is not a replacement for defining durable write ordering. I would first establish correct WAL synchronization semantics, then evaluate direct I/O separately through benchmarks.”

### How does query routing work?

> “The current engine uses procedural rules. Primary-key predicates use the B-tree, configured text predicates use the full-text structure, and other queries use a full scan. The next step is to represent those choices as explicit plan nodes and expose them through EXPLAIN-style introspection.”

---

## 🌉 Bridge Statement

> “ByteForge began as an exploration of what lies beyond writing rows to CSV. I focused first on the database data path: typed schema persistence, TLV record encoding, file-backed table storage, primary and full-text access paths, page caching, and recovery-aware startup.
>
> The next evolution is the control path. I want to formalize access paths as execution-plan nodes, expose EXPLAIN-style introspection, introduce rule-based selection, and add operational policies around checkpointing, index reconstruction, and cache behavior.
>
> This progression moves ByteForge from a functional storage-engine prototype toward a measurable and policy-driven database system.”

---

## 🛠️ Source-Aligned Upgrade Table

| Feature | Current ByteForge design | Next engineering milestone | Trade-off |
| :--- | :--- | :--- | :--- |
| **Record encoding** | TLV-encoded typed values | TLV records stored inside slotted pages | Adds slot and page metadata but enables direct record-offset access. |
| **Page management** | Page-position-based addressing with a small development capacity | Fixed-size physical pages with stable page IDs | Requires free-space management and page-format versioning. |
| **Primary index** | In-memory B-tree with persisted key-position mappings | Page-based persistent B+ tree or rebuildable index generations | Adds node persistence, split handling, and recovery complexity. |
| **Full-text lookup** | Persisted map from text values to record references | Tokenization, normalization, and incremental persistence | Increases index size and mutation cost. |
| **WAL identity** | Generated entry identifiers | Monotonic LSNs and persisted replay position | Requires ordering metadata and applied-state tracking. |
| **Recovery** | WAL restoration during database startup | Checksummed, bounded, and idempotent replay | Adds metadata and recovery-state coordination. |
| **Durability** | OS-buffered file writes | WAL synchronization and configurable group commit | Trades commit latency for stronger persistence guarantees. |
| **Deletion** | Tombstone record markers | Page compaction or background vacuuming | Reclaims space but consumes CPU and I/O resources. |
| **Caching** | LRU page cache | Buffer pool with pinning, dirty-page tracking, and scan-aware admission | Adds memory-management and concurrency complexity. |
| **Query routing** | Procedural access-path selection | Explicit plans and Rule-Based Optimizer | Adds planner abstractions and deterministic rule management. |
| **Query costing** | Execution metadata such as rows inspected and cache use | Statistics-driven Cost-Based Optimizer | Requires cardinality estimates and calibrated cost models. |
| **Observability** | Query-result execution metadata | Metrics for I/O, cache, WAL, recovery, and amplification | Adds instrumentation overhead and metric-lifecycle management. |

---

---

## 🎯 Thirty-Second Interview Version

> “ByteForge began as an exploration of what comes after writing rows to CSV. I built the database data path in Go: typed schema persistence, custom TLV record encoding, file-backed storage, a B-tree primary access path, full-text lookup, an LRU page cache, and WAL-based restoration during startup.
>
> I deliberately focused on the data path first because query planners and optimizers ultimately depend on reliable storage access paths. The next phase is the control path, where I will formalize execution plans, add EXPLAIN-style introspection, introduce rule-based access-path selection, and measure cache, I/O, and recovery behavior.”

---

## Final Project Positioning

ByteForge should be presented as:

> **A data-path-first database storage engine that grew from a CSV-writing baseline into typed binary storage, persisted schema, logical-to-physical addressing, multiple query access paths, page caching, and recovery-aware startup, with the control path defined as its next architectural evolution.**
