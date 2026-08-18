
# ByteForge

## From CSV File Writing to a Database Storage Engine

*Amazon-style project narrative and interview playbook*

> **Project focus**
>
> ByteForge focused first on the database **data path**: transforming typed application values into a binary representation, persisting them, locating them through indexes, and restoring usable state when the database opens. The **control path** is the next evolution: formal query planning, execution introspection, policy decisions, and operational coordination.

**DIVE DEEP • OWNERSHIP • LEARN AND BE CURIOUS • INVENT AND SIMPLIFY • DELIVER RESULTS**

---

# Executive Overview

ByteForge began with a simple engineering question: what changes when a program stops writing rows to a CSV file and starts behaving like a database? A CSV writer can append text, but a database engine must define types, encode records, preserve schema, locate rows efficiently, coordinate multiple persisted structures, and reopen its state in a usable form.

The project therefore started from the write path and grew into a modular storage engine in Go. Its central focus was the **data path**, the route through which application values become database structures and return as query results. The architecture separates binary encoding, schema metadata, table storage, primary indexing, full-text lookup, and recovery so that each subsystem can evolve independently.

> **One-sentence interview summary**
>
> I built ByteForge to understand the transition from appending CSV rows to managing typed, indexed, recoverable database state, with the first version focused on the storage-engine data path and a clear path toward the database control path.

# Data Path and Control Path

| Path | Primary responsibility | ByteForge direction |
| --- | --- | --- |
| **Data path** | Moves and transforms data through encoding, storage, lookup, and recovery. | Primary focus of the current engine. |
| **Control path** | Chooses and coordinates how work should execute through planning, policies, observability, and lifecycle management. | Planned evolution built on top of stable access paths. |

ByteForge did not begin as a SQL parser or optimizer. It began below that layer, where the engine must answer foundational questions:

- How should a typed value be represented in bytes?
- How should a table schema survive a restart?
- How can a primary key locate a physical record efficiently?
- How should a secondary lookup structure reference the same row?
- How should the engine reconstruct its in-memory state when reopened?

# 1. Origin Story: From CSV to a Database Engine

## The starting point

The project started from a familiar application pattern: take structured values and write them to a CSV file. That approach works for export and simple append-only workflows, but it quickly exposes limitations when the file must support database-like behavior.

| CSV-style requirement | Database-engine question |
| --- | --- |
| Append a row | How is a typed record encoded and framed? |
| Read a row | How is the record located without scanning everything? |
| Change a value | How are updates represented and associated indexes maintained? |
| Delete a row | How is deletion expressed without losing record boundaries? |
| Restart the program | How are schema, indexes, and recovery state reconstructed? |
| Search by another field | How is a second access path represented and persisted? |

## The goal

The goal was to progress deliberately from file writing to database internals. Instead of immediately building SQL syntax, joins, or a distributed system, ByteForge concentrated on the mechanics that make higher-level database behavior possible:

- Typed schemas and column metadata
- Binary record serialization and parsing
- File-backed table storage
- Primary-key-to-position indexing
- Full-text access paths
- Write-ahead logging and restoration
- Modular boundaries between storage subsystems

## The engineering thesis

> **Core thesis**
>
> A database is not merely a file containing rows. It is a set of coordinated representations that preserve meaning, identity, location, and recoverability as data moves between memory and storage.

# 2. Architecture: The ByteForge Data Path

A write begins as a map of typed application values. The engine validates those values against column definitions, converts them into TLV-encoded bytes, writes the record through the table layer, and updates access structures that connect logical keys to physical positions. When the database opens, persisted schema and index data are read to reconstruct the usable in-memory view.

| Stage | Responsibility | Representative component |
| --- | --- | --- |
| 1. Schema | Defines column names, types, and indexing options. | `column` |
| 2. Encoding | Transforms scalar and nested values into TLV bytes. | `parser/encoding` |
| 3. Storage | Writes and reads records in table files. | `table` |
| 4. Primary access | Maps integer IDs to physical positions. | `index` |
| 5. Text access | Maps indexed text values to record references. | `fulltext` |
| 6. Recovery | Records operations and restores engine state. | `wal` |
| 7. Database lifecycle | Creates, opens, discovers, and closes tables. | `internal/database` |

## Write-path walkthrough

1. The caller provides a row as typed values keyed by column name.
2. The table schema provides the expected order and type of each column.
3. The encoding layer converts each value into a type, length, and value representation.
4. The table layer records the binary row and associates it with a physical position.
5. The primary index maps the row ID to that position for efficient point lookup.
6. Configured text fields update the full-text access structure.
7. The recovery subsystem records operation context for restoration when the engine opens.

## Read-path walkthrough

1. A primary-key query uses the B-tree mapping to locate the relevant physical position.
2. The table reader seeks to the stored location and parses the binary record.
3. The schema supplies column names so decoded values can be reconstructed as a logical row.
4. A query without an eligible access path can use the table-scan path.
5. Text lookup uses the separate full-text mapping to identify matching record references.

# 3. Key Design Decisions

## Custom TLV encoding

ByteForge uses a custom **Type-Length-Value** format for heterogeneous data. The type identifies how to decode a value, the length frames its payload, and the value contains the binary representation. Common marshaling primitives are reused for records, schema metadata, index items, lists, and map-like structures.

> **Why it mattered**
>
> Starting with a binary format forced the project to treat record boundaries, types, lengths, and restart compatibility as explicit engineering concerns rather than delegating them to a text format.

## Persisted schema metadata

Column definitions are persisted with the table. This allows the database to reopen a table and rebuild the relationship between encoded values and logical column names. Schema persistence marks a key step beyond CSV writing because the file carries a formal interpretation contract, not merely delimited text.

## Primary-key index

The primary access path uses an in-memory B-tree that maps integer identifiers to physical record positions. ByteForge serializes these mappings to a separate file and reconstructs the tree when the database opens. The result is a clear separation between logical identity and physical location.

## Full-text access path

A separate full-text structure maps indexed text values to record references. This demonstrates how one table can support multiple access paths while keeping record storage independent from query-specific lookup structures.

## WAL and restoration

The recovery subsystem gives the engine an explicit place to represent operations that must survive beyond the immediate method call. During database initialization, tables restore their WAL state before serving normal operations. This makes recovery part of the database lifecycle rather than an external repair utility.

## Modular subsystem boundaries

Encoding, parsing, columns, table storage, indexes, full-text lookup, and WAL handling are implemented as separate packages. This structure supports incremental evolution. A new page layout, index-persistence strategy, or recovery protocol can be introduced behind a focused boundary while preserving the rest of the data path.

# 4. Amazon-Style STAR Narrative

## Situation

I was comfortable using databases from application code, but I wanted to understand what sits between a structured object and a persisted, queryable record. I began with the simple model of writing rows to CSV and used its limitations to define the requirements of a small database engine.

## Task

I set out to build a file-backed engine in Go that could persist typed schemas and records, provide primary and text-based access paths, and reconstruct its operational state when reopened. I intentionally focused on the storage data path before introducing higher-level query-control features.

## Action

- Designed a reusable TLV encoding layer for typed scalar values and nested structures.
- Persisted column definitions so stored records retain a consistent interpretation across restarts.
- Implemented table-level record parsing and page-position-based addressing.
- Integrated a B-tree primary access path and persisted the key-to-location mappings.
- Added a separate full-text lookup structure for text-oriented access.
- Created a WAL and restoration flow integrated into database startup.
- Separated the engine into focused packages so storage, indexing, and recovery could evolve independently.

## Result

The project grew from a file-writing exercise into a functional storage-engine prototype. It can represent typed tables, translate rows into binary records, maintain distinct lookup structures, reopen persisted table metadata, and restore recovery state. More importantly, it gave me an end-to-end mental model of how data moves through a database below the SQL layer.

## Learning

The project showed me that database engineering is largely about preserving relationships between representations: schema and values, logical IDs and physical locations, base records and indexes, and runtime structures and persisted state. That understanding now guides the next phase of ByteForge.

# 5. Leadership Principle Mapping

| Leadership Principle | How ByteForge demonstrates it |
| --- | --- |
| **Dive Deep** | Traced the journey from typed application values to encoded bytes, physical positions, indexes, and restored state. |
| **Learn and Be Curious** | Moved below database APIs to study serialization, file formats, access paths, and recovery. |
| **Ownership** | Designed the database lifecycle so schema loading, index reconstruction, restoration, and resource closing are engine responsibilities. |
| **Invent and Simplify** | Created reusable encoding and parsing primitives and separated subsystem responsibilities into focused packages. |
| **Deliver Results** | Produced a working storage-engine prototype rather than stopping at a conceptual design. |
| **Think Big** | Established a foundation on which page management, query planning, observability, and recovery policies can be developed. |

# 6. Next Evolution: Building the Control Path

With the fundamental data path in place, the next evolution is the control path. The control path does not replace storage. It decides how the existing storage and access capabilities should be used, coordinated, and observed.

| Control-path capability | Purpose | Dependency on the data path |
| --- | --- | --- |
| Execution-plan model | Represents table scans, primary lookups, and text lookups as explicit plan nodes. | Requires stable access-path interfaces. |
| EXPLAIN-style introspection | Shows which access path a query will use and why. | Requires the plan to match actual execution. |
| Rule-based planning | Selects an access path from predicate shape and available indexes. | Requires known index capabilities. |
| Statistics and costing | Uses cardinality and selectivity to compare alternatives. | Requires measured storage and access behavior. |
| Operational observability | Reports scan counts, lookup counts, recovery time, and I/O behavior. | Requires instrumentation across data-path stages. |
| Lifecycle policy | Coordinates checkpoints, index reconstruction, and maintenance work. | Requires defined recovery and persistence boundaries. |

> **Project positioning**
>
> ByteForge is best presented as a **data-path-first database engine**. The work begins where CSV writing ends: typed binary records, persisted schema, logical-to-physical addressing, multiple access paths, and recovery-aware startup. The control path is the deliberate next phase.

# 7. Interview Deep-Dive Playbook

### Why did you start from CSV?

CSV provided the simplest baseline for persisting rows. Its limitations made the database requirements concrete: types, schema, record framing, efficient lookup, updates, multiple access paths, and restart reconstruction.

### Why focus on the data path first?

Every planner or optimizer eventually invokes a storage access path. I wanted those foundational mechanisms to exist before adding query-control abstractions.

### What does page-position-based addressing provide?

It separates a row’s logical identity from its physical location. The primary index can map an ID to a stored position, allowing the reader to seek directly to the relevant region.

### Why use TLV instead of CSV or JSON?

TLV made types and record framing explicit while providing reusable binary-marshaling primitives. It also made serialization a first-class part of the engine rather than an external library call.

### What is the role of the B-tree?

It is the primary in-memory access structure for mapping integer IDs to physical positions. Its persisted mappings allow the structure to be reconstructed when the database opens.

### Why separate the full-text index?

A text lookup and a primary-key lookup serve different query patterns. A separate structure demonstrates how multiple access paths can reference the same base records.

### What does the WAL contribute?

It gives the engine a dedicated recovery representation and makes restoration part of table initialization. It is the foundation for evolving operation recovery and lifecycle coordination.

### What comes next?

First, formalize execution paths as plan nodes. Then add EXPLAIN-style introspection and rule-based selection. In parallel, evolve page management, recovery metadata, and operational measurements.

# 8. Final Resume Entry

## Recommended four-bullet version

- Built a file-backed storage engine in Go with typed schemas, custom **TLV binary serialization**, record parsing, and page-position-based record addressing.
- Integrated an in-memory **B-tree primary index** and implemented custom persistence of key-to-file-position mappings for efficient point lookups across database restarts.
- Implemented a **Write-Ahead Log and recovery path** using OS-buffered file I/O, establishing a foundation for recovery-aware database startup and operation restoration.
- Designed a modular architecture separating **encoding, storage, primary and full-text indexing, and recovery**, creating a clear path toward slotted-page storage and query-control capabilities.



# 9. Final Interview Positioning

> **Thirty-second version**
>
> ByteForge began as an exploration of what lies beyond writing rows to CSV. I built the database data path in Go: typed schema persistence, TLV record encoding, file-backed storage, primary and full-text access structures, and recovery-aware startup. The project now has a clear evolution toward the control path through explicit execution plans, EXPLAIN-style introspection, rule-based access-path selection, and operational observability.

The most effective way to present ByteForge is as a deliberate progression:

1. CSV writing established the baseline.
2. Typed binary storage established the record model.
3. Persisted schema established interpretation across restarts.
4. Indexes established efficient alternative access paths.
5. WAL restoration established recovery-aware lifecycle behavior.
6. The control path will coordinate, explain, and optimize those capabilities.

