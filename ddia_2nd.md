### Chapter 1. Trade-Offs in Data Systems Architecture

[Page 1]
- Data-intensive architecture composes databases, caches, search indexes, stream processors, batch systems, queues, and stateless backends into a persistence/control plane where durable state lives outside request handlers.
- Single-node simplicity collapses once data volume, query rate, concurrency, or product complexity forces distribution; every added subsystem buys capability but creates consistency gaps, failure modes, operational coupling, and debugging surface area.
- The core FAANG/system-design pivot is workload decomposition: operational vs analytical paths, read/write patterns, latency SLOs, durability needs, cache invalidation risks, and when one datastore becomes multiple purpose-built systems.

[Page 2]

- Operational systems are write-serving, user-facing source-of-truth paths where backend services mutate state under latency, availability, concurrency, and correctness constraints.
- Analytical systems are read-optimized copies/derivations of operational data, isolated to protect production workloads from heavy scans, joins, ML feature extraction, BI queries, and exploratory access patterns.
- FAANG-scale design separates OLTP from OLAP: protect hot request paths from analyst/data-science workloads, then build reliable data pipelines, ownership boundaries, freshness SLAs, and derived-data contracts between service teams and data teams.

[Page 3]

- OLTP serves interactive product traffic via fixed, low-latency point reads/writes over current state; OLAP serves exploratory analytics via scans, joins, and aggregates over historical data at much larger scale.
- Mixing arbitrary analyst queries with production OLTP is an availability risk: unbounded scans, bad joins, lock contention, cache pollution, and I/O saturation can degrade user-facing SLOs.
- FAANG design pivot: keep hot transactional paths narrow and predictable, then replicate events/state into OLAP or real-time analytics systems like ClickHouse/Druid/Pinot when product features need low-latency aggregates over fresh data.

[Page 4]

- Data warehouses decouple OLTP sources from analytical access by extracting operational data from many service-owned databases/SaaS APIs, transforming or loading it into query-optimized schemas, and giving analysts a safe cross-domain read model.
- ETL/ELT solves data-silo and production-isolation problems but introduces freshness lag, schema drift, lineage complexity, duplicate truth, connector fragility, and governance/security risks across hundreds of independently owned source systems.
- FAANG-scale pivot: use service-local OLTP databases for ownership and blast-radius control, central warehouses/lakes for BI/ML, HTAP only for narrow workloads needing fresh aggregates plus point mutations, and raw data lakes when SQL schemas are too restrictive for ML, NLP, images, vectors, or feature engineering.

[Page 5]

- Modern analytical architecture extends beyond batch data lakes into governed DataOps pipelines, event streams, reverse ETL, and ML deployment loops where analytical outputs re-enter production serving paths.
- Systems of record own canonical facts; caches, indexes, materialized views, warehouses, feature stores, trained models, and denormalized tables are derived state—valuable for latency/read scalability but rebuildable and consistency-lagged.
- FAANG-scale design pivot: explicitly label source-of-truth vs derived systems, then engineer propagation paths, replayability, lineage, privacy controls, freshness SLAs, and failure recovery instead of pretending one database can safely serve every workload.

[Page 6]

- Cloud vs self-hosting is a control-plane trade-off: outsource undifferentiated operations to managed services when speed/elasticity matters; self-host when workload predictability, deep tuning, hardware control, compliance, or differentiation dominates.
- Managed services reduce operational burden but introduce vendor lock-in, opaque debugging, feature dependency, outage helplessness, migration risk, geopolitical exposure, and constrained observability into failure/performance internals.
- FAANG-scale pivot: choose cloud for elastic/bursty workloads like interactive analytics, early product velocity, and non-core infra; choose self-hosted/specialized platforms when latency, cost efficiency at steady scale, introspection, custom behavior, or blast-radius ownership becomes strategic.

[Page 7]

- Cloud-native data systems are built by composing managed lower-level cloud services—object storage, compute, networking, metadata/control planes—rather than treating VMs as physical servers with durable local disks.
- Separation of storage and compute enables elastic scaling, fast recovery, and cheaper durable storage, but moves the bottleneck to network I/O, service quotas, multitenancy isolation, tail latency, and cross-service integration complexity.
- FAANG-scale pivot: cloud removes machine-level toil, not operations; SRE work shifts to automation, cost/performance engineering, quota planning, observability, incident learning, security, service integration, and choosing the right abstraction level for each workload.

[Page 8]

- Distributed systems are justified by unavoidable networked users, cloud-service composition, HA/fault tolerance, horizontal scale, geo-latency, elasticity, specialized hardware, data residency, and sustainability-driven placement.
- Distribution converts local assumptions into partial failure: timeouts hide whether work happened, retries can duplicate side effects, network calls dominate latency, data movement can exceed compute cost, and debugging requires observability/tracing.
- FAANG-scale pivot: start single-node when it meets SLOs; distribute only for a concrete constraint, then design idempotency, retries, backpressure, tracing, ownership boundaries, and cross-service consistency explicitly instead of assuming microservices are free scalability.

[Page 9]

- Microservices decompose systems into independently owned network APIs with service-local databases, enabling team autonomy, isolated scaling, deploy independence, and implementation hiding; serverless pushes this further by metering execution and outsourcing runtime allocation.
- Distribution across services creates API-versioning risk, integration-test complexity, per-service observability/on-call overhead, network failure paths, cold starts, runtime limits, and cross-database consistency problems that monoliths avoid.
- FAANG-scale pivot: use microservices to solve organizational scale, not as default architecture; combine service ownership with explicit API contracts, backward compatibility, tracing, deployment automation, data minimization, privacy controls, deletion semantics, and compliance-aware derived-data lifecycle management.


## Add Distinguish between OLTP and OLAP



### Chapter 2 Defining Nonfunctional Requirements

[Page 10]
- Social Media Case Study
- Nonfunctional requirements define production viability: performance, reliability, scalability, and maintainability determine whether correct functionality survives real load, failures, growth, and long-term evolution.
- Home timelines expose the core read/write trade-off: compute timeline on read via joins/polling and overload OLTP paths, or materialize timelines on write using fan-out, queues, caches, and derived state.
- FAANG-scale pivot: optimize for the dominant path—precompute high-QPS reads, absorb write spikes asynchronously, handle celebrity/high-fanout accounts with hybrid fan-out-on-write plus fan-out-on-read, and treat materialized timelines as eventually consistent derived data.

[Page 11]

- Performance must be modeled as distributions, not averages: response time = client-visible end-to-end delay, service time = active processing, latency/queueing = idle wait caused by network, scheduling, contention, GC, disk, or downstream bottlenecks.
- As throughput approaches capacity, queueing dominates and can trigger metastable failure via retry storms; production systems need backoff, jitter, circuit breakers, token buckets, load shedding, backpressure, and queue-aware load balancing.
- FAANG-scale pivot: optimize and set SLOs on p95/p99/p999 tail latency, not mean latency, because fan-out across many backend calls amplifies tails; measure with mergeable histograms/sketches, never averaged percentiles.

[Page 12]

- Reliability means faults do not become user-visible failures: design every critical path with explicit fault domains, redundancy, failover, idempotent recovery, and SLO-based definitions of “working correctly.”
- Hardware faults are routine at scale, but software faults are more dangerous because they correlate across homogeneous fleets, trigger cascading overload, exhaust shared resources, or activate dormant assumptions under rare production conditions.
- FAANG-scale pivot: tolerate machine/rack/AZ loss with replicated multi-node systems, rolling upgrades, and chaos/fault injection; prevent correlated software failure with isolation, observability, backpressure, safe retries, staged rollout, and continuous production validation.

[Page 13]

- Human reliability is a sociotechnical property: outages often originate from config changes, ambiguous tooling, weak rollout controls, poor observability, missing rollback paths, and incentive systems that prioritize feature velocity over resilience.
- “Human error” is usually the last visible action, not root cause; production safety comes from guardrails—tests, property checks, staged deploys, canaries, rollbacks, clear dashboards, safe defaults, and blameless postmortems.
- FAANG-scale pivot: reliability investment must match harm potential—temporary downtime may be tolerable, but data loss, corruption, incorrect financial/legal evidence, or unverifiable automated decisions require stronger correctness, auditability, recovery, and organizational accountability.

[Page 14]

- Scalability is workload-specific capacity planning, not a binary property: define load dimensions first—QPS, write volume, data growth, concurrent users, read/write ratio, cache hit rate, fan-out, and pathological heavy users.
- Premature scale engineering can freeze product evolution, but ignoring measured growth creates predictable SLO collapse; the real question is how performance changes under fixed resources and how much extra resource preserves the SLA.
- FAANG-scale pivot: model scalability as cost-to-serve under growth—linear scalability is ideal, superlinear cost growth exposes bottlenecks like coordination, hot keys, larger indexes, cache misses, compaction, cross-shard queries, or high-fanout writes.

[Page 15]

- Scaling choices form a spectrum: vertical/shared-memory buys simplicity but hits cost and contention walls; shared-disk centralizes storage but suffers locking/contention; shared-nothing enables horizontal scale by partitioning CPU/RAM/disk ownership per node.
- Shared-nothing can approach linear scalability and multi-region fault tolerance, but only if sharding, rebalancing, hot-key mitigation, distributed coordination, and failure handling are engineered explicitly.
- FAANG-scale pivot: there is no generic “scalable architecture”; redesign around each order-of-magnitude load jump, split components only along real independence boundaries, and prefer the simplest architecture that satisfies current + near-term SLOs.

[Page 16]

- Maintainability is long-term production leverage: most software cost is post-launch—debugging, operations, migrations, dependency/platform changes, feature evolution, incident response, and technical-debt repayment.
- Legacy pain comes from lost context, obsolete tech, inconsistent patterns, hidden coupling, and systems entangled with organizational workflows; today’s successful system becomes tomorrow’s maintenance burden.
- FAANG-scale pivot: design for operability, simplicity, and evolvability—strong observability, machine independence, predictable operational model, safe automation with manual override, clear defaults, documentation, and architectures new engineers can reason about quickly.

[Page 17]

- Simplicity is a scalability multiplier for engineering teams: complexity creates hidden coupling, unsafe assumptions, bug-prone changes, slow onboarding, and “big ball of mud” failure modes that dominate long-term maintenance cost.
- Abstraction is the main weapon against complexity, but only when it hides implementation detail behind stable semantics; bad abstractions leak behavior, constrain evolution, or make production failures harder to reason about.
- FAANG-scale pivot: build evolvable systems by minimizing irreversibility—prefer loose coupling, stable contracts, migrations with rollback paths, dual-writes/backfills where needed, refactoring-friendly boundaries, and architecture that can survive requirement, platform, legal, and scale shifts.

[Page 18]

- Data models are layered abstractions: application objects map to relational/document/graph/event/DataFrame models, which map to storage-engine bytes, which map to hardware; each layer shapes both implementation and reasoning.
- Declarative query languages decouple intent from execution plan, letting optimizers choose indexes, join order, parallelism, and distribution without forcing application code to encode physical access paths.
- FAANG-scale pivot: data-model choice is an architectural constraint—relational excels at structured joins and analytics, documents at locality/schema flexibility, graphs at relationship traversal, events at audit/replay, and query language semantics determine future evolvability more than syntax.

[Page 19]

- ORM/document/relational mismatch is a modeling-boundary problem: ORMs reduce boilerplate but cannot hide query planning, schema quality, N+1 access patterns, analytics needs, or non-relational storage/search/graph realities.
- Document models optimize locality for bounded one-to-few tree-shaped data; relational normalization optimizes canonical identity, consistency, reuse, localization/search, and update efficiency via joins.
- FAANG-scale pivot: denormalization is derived data—use it only when read-path savings justify write amplification, consistency machinery, storage cost, and crash recovery; keep fast-changing/high-fanout fields normalized and hydrate IDs at read time when joins parallelize cheaply.

[Page 20]

- Many-to-one/many-to-many relationships push systems toward normalized references plus secondary indexes; duplicating relationship edges inside documents improves locality but creates bidirectional-consistency hazards.
- Star/snowflake/OBT schemas optimize analytics by modeling immutable event facts plus descriptive dimensions; OLAP denormalization is safer than OLTP denormalization because historical facts are bulk-loaded and rarely mutated.
- FAANG-scale pivot: choose document models for bounded tree/locality workloads, relational models for joins/references/global identity, schema-on-read for heterogeneous/evolving external data, and schema-on-write when uniformity, validation, migrations, and long-term operability matter.

[Page 21]

- Graph models represent relationship-dense domains as vertices plus edges, outperforming document/tree models when many-to-many links, recursive traversal, path queries, heterogeneous entities, or network algorithms dominate.
- Adjacency lists optimize traversal locality; adjacency matrices fit numerical/ML workloads, but graph workloads at scale face high-degree nodes, traversal fan-out, partitioning pain, and poor locality across distributed shards.
- FAANG-scale pivot: use graphs for social networks, knowledge graphs, fraud rings, recommendations, ranking, identity linkage, and route/path queries—but validate whether traversal depth, edge cardinality, and partition strategy can meet latency SLOs before choosing a graph database.

[Page 22]

- Property graphs generalize join tables into typed vertices and typed edges with properties, enabling bidirectional traversal via indexes on edge heads/tails and flexible modeling of heterogeneous entities in one structure.
- Flexibility comes with modeling and scale risks: unrestricted edges improve evolvability but can weaken schema guarantees, complicate validation, and make high-degree traversals or graph partitioning expensive.
- FAANG-scale pivot: property graphs fit domains where relationships evolve faster than schemas—identity graphs, knowledge graphs, fraud links, permissions, recommendations—but production success depends on indexing strategy, traversal bounds, edge cardinality control, and clear ownership of graph semantics.

[Page 23]

- Cypher/GQL encode graph traversal as declarative pattern matching: vertices, labeled edges, property predicates, and variable-length paths express relationship queries directly instead of forcing recursive join mechanics into application code or SQL CTEs.
- SQL can represent graph storage, but variable-depth traversal becomes verbose, optimizer-sensitive, and operationally tricky around cycles, traversal order, fan-out explosion, and index usage across repeated self-joins.
- FAANG-scale pivot: query-language fit matters as much as storage model—use graph query languages when product logic depends on path discovery, hierarchy traversal, identity resolution, or relationship predicates that would otherwise become brittle recursive SQL/application traversal code.

[Page 24]

- Triple stores model all facts as (subject, predicate, object), making properties and edges uniform; RDF adds URI-based namespaces for global semantic interoperability and collision avoidance across independently produced datasets.
- RDF/SPARQL trades simpler graph primitives for verbosity and semantic-web baggage, but gains standardized encodings, ontology compatibility, linked-data integration, and compact declarative path queries similar to Cypher.
- FAANG-scale pivot: use triple/RDF-style models when entity facts, metadata, ontologies, knowledge graphs, biomedical/catalog/search data, or cross-organization vocabularies matter more than rigid app-specific schemas; bound traversal cost and namespace governance early.

[Page 25]

- Datalog treats relations as facts and queries as composable recursive rules, making transitive closure, graph traversal, policy inference, lineage, and dependency reasoning expressible without hardcoding traversal algorithms.
- GraphQL is not a graph database language; it is a client-facing OLTP query shape language that trades expressive power for safety, predictable schemas, UI agility, and controlled server-side joins.
- FAANG-scale pivot: use Datalog/SPARQL/Cypher for trusted recursive relationship queries; use GraphQL at the API edge with strict authorization, query-depth/complexity limits, rate limiting, batching/caching, and resolver observability to avoid N+1 and DoS failure modes.

[Page 26]

- Event sourcing makes an immutable ordered event log the source of truth, then derives CQRS read models/materialized views optimized for different query paths, storage models, and latency requirements.
- Benefits are auditability, replay, deterministic rebuilds, high append throughput, burst absorption, and easier evolution; costs are event ordering, schema evolution, GDPR deletion, deterministic external data, side-effect suppression, and projection consistency.
- FAANG-scale pivot: use event logs/Kafka-style pipelines when write history, replayability, multiple read models, ML/analytics integration, or audit trails matter; use DataFrames/matrices/arrays when analytical/ML workflows need bulk transformations, sparse features, linear algebra, and scientist-controlled schema-on-read experimentation.


[Page 27]

- Storage engines reduce to two primitives: append/write bytes durably, then retrieve them efficiently; OLTP engines diverge mainly between log-structured immutable writes and in-place/update-oriented structures like B-trees.
- Append-only logs maximize write throughput and crash-recovery simplicity, but naïve reads become O(n); indexes convert scans into targeted lookup paths at the cost of extra disk, write amplification, maintenance, and recovery complexity.
- FAANG-scale pivot: indexes are derived data chosen from workload knowledge—add only those that materially improve hot read/query paths, because every secondary index taxes write latency, storage footprint, compaction/rebalancing, and operational tail latency.


[Page 28]

- LSM engines convert random writes into sequential appends via memtables, SSTables, sparse indexes, WALs, tombstones, and background compaction; reads search newest-to-oldest segments and rely on Bloom filters to skip definitely-absent keys.
- The core trade-off is write throughput vs read/space amplification: immutable sorted segments make writes/recovery efficient, but compaction burns CPU/I/O, tombstones delay reclamation, false positives waste reads, and old/missing keys may touch many files.
- FAANG-scale pivot: choose/tune LSMs for high-ingest workloads like logs, metrics, feeds, time-series, and wide-column KV stores; use size-tiered compaction for write-heavy paths, leveled compaction for read-heavy paths, and budget operationally for compaction debt, disk headroom, cache behavior, and tail-latency spikes.

[Page 29]

- B-trees store sorted key ranges in fixed-size pages with high branching factor, giving predictable O(log n) point/range lookup and efficient in-place updates via leaf modification, page splits, and balanced tree maintenance.
- In-place mutation makes B-trees read-friendly but crash-sensitive: multi-page updates, torn pages, and buffered writes require WAL/fsync or copy-on-write schemes to preserve consistency after failures.
- FAANG-scale pivot: choose B-tree-style engines for read-heavy OLTP, range scans, relational indexes, and predictable latency; watch write amplification from page splits/WAL, cache residency of upper levels, random I/O, fragmentation, and contention on hot key ranges.

[Page 30]

- B-trees optimize predictable reads via few page traversals and efficient range scans; LSM-trees optimize write throughput by batching random updates into sequential SSTable writes and background merges.
- LSMs trade higher write efficiency for read amplification, compaction debt, Bloom-filter false positives, range-query complexity, and latency stalls when flush/compaction cannot keep up; B-trees trade read predictability for random-write cost, WAL/page write amplification, fragmentation, and vacuum/rewrite overhead.
- FAANG-scale pivot: benchmark against the real workload—point vs range reads, overwrite rate, key/value size, SSD behavior, compaction steady state, deletion guarantees, backup/snapshot model, and tail latency under sustained ingest matter more than “B-tree vs LSM” ideology.

[Page 31]

- Index value placement defines read/write locality: clustered indexes store rows with the primary access path, heap files store rows separately with indexes pointing to locations/PKs, and covering indexes duplicate selected columns to satisfy hot queries index-only.
- Covering/clustered indexes reduce lookup hops and random I/O but increase write amplification, storage usage, update complexity, and relocation/forwarding-pointer costs when row size changes.
- FAANG-scale pivot: optimize physical layout for dominant access paths—use clustered/covering indexes for latency-critical reads, heap/secondary indexes for flexibility, and in-memory databases when the working set fits RAM and disk serialization overhead—not disk reads—is the bottleneck.

[Page 32]

- Analytical storage decomposes into query engine, columnar file format, table format, and catalog; cloud warehouses exploit object storage + separated compute to scale scans elastically across petabyte fact tables.
- Column stores accelerate OLAP by reading only referenced columns, compressing repeated values, using bitmap/run-length encodings, sorting blocks by common predicates, and batching writes into immutable column files.
- FAANG-scale pivot: choose columnar layouts for scan-heavy BI/ML/product analytics, but design around bulk ingestion, late-arriving updates/deletes, object-store metadata, sort-key choice, compression efficiency, catalog governance, and query engines that merge recent row-oriented writes with immutable columnar data.

[Page 33]

- Analytical query engines decompose SQL into distributed operator pipelines; performance hinges on CPU-efficient execution—JIT query compilation or vectorized batch operators—rather than merely reducing disk reads.
- Compilation removes interpretation overhead by generating tight machine-code loops; vectorization processes column batches/bitmaps with cache-friendly sequential access, SIMD, fewer branches, and minimal materialization.
- Materialized views/data cubes precompute expensive aggregates for repeated OLAP queries, trading read latency gains for write amplification, freshness maintenance, storage overhead, and reduced flexibility versus querying raw facts.

[Page 34]

- Single-column indexes fail when predicates span multiple independent dimensions; concatenated indexes only help ordered-prefix queries, while spatial/R-tree/BKD/grid indexes prune multidimensional spaces such as geo, color, time-temperature, or map viewport queries.
- Full-text search uses inverted indexes from term → postings list/bitmap, enabling fast Boolean/set operations over high-dimensional sparse text features; n-grams, edit-distance automata, stemming, tokenization, and language handling trade recall against index size and CPU.
- Semantic/vector search converts text/images/audio into high-dimensional embeddings and uses flat, IVF, or HNSW indexes to find nearest neighbors; FAANG-scale design must explicitly tune recall-vs-latency, freshness, memory footprint, sharding, reranking, and hybrid lexical+vector retrieval for RAG/search quality.

[Page 35]

- Schema evolution is a production compatibility problem: rolling deploys, stale mobile clients, old stored records, and mixed service versions require backward compatibility, forward compatibility, and preservation of unknown fields during read-modify-write cycles.
- Encoding bridges in-memory object graphs and portable byte sequences; language-native serializers are dangerous for durable/networked data because they couple to one runtime, weaken versioning, bloat payloads, and can enable arbitrary-code-execution attacks during decoding.
- JSON/XML/CSV maximize interoperability but suffer from schema ambiguity, numeric precision traps, binary-data hacks, and verbose encodings; JSON Schema adds validation/governance but open content models, remote refs, and conditional schemas make compatibility reasoning operationally hard at scale

[Page 36]

- Protocol Buffers encode schema-defined records compactly using numeric field tags and wire types; schema evolution depends on never reusing tags, preserving unknown fields, adding fields with safe defaults, and treating type changes as compatibility hazards.
- Avro removes field tags and resolves evolution by comparing writer schema with reader schema, making it strong for dynamically generated schemas, data lake files, Kafka/schema-registry pipelines, and relational exports where schemas change by name.
- FAANG-scale pivot: schema-based binary formats give compact payloads, generated types, compatibility checks, and living documentation; use schema registries and automated compatibility gates, but minimize concurrently active schema versions to reduce operational/debugging complexity.

[Page 37]

- Dataflow mode determines compatibility pressure: databases require backward + forward compatibility because data outlives code and mixed app versions may read/write the same records during rolling deploys.
- Service APIs encode organizational boundaries: REST/gRPC/OpenAPI/Protobuf expose constrained business operations instead of arbitrary queries, enabling independent team ownership only if request/response schemas remain compatible.
- RPC must not pretend network calls are local calls—timeouts, retries, duplicate side effects, variable latency, encoding cost, service discovery, load balancing, and partial failure require explicit API contracts, idempotency keys, versioning, observability, and rollout discipline.

[Page 38]

- Durable execution models workflows as task graphs with an orchestrator/executor and a persisted execution log, enabling crash recovery by replaying prior RPC/state outcomes instead of blindly re-running side effects.
- Exactly-once workflow semantics still depend on idempotent external APIs, deterministic replay, stable task ordering, versioned workflow definitions, and avoiding nondeterminism from clocks, randomness, or reordered calls.
- Event-driven systems replace direct RPC coupling with brokered asynchronous messages, improving buffering, redelivery, fan-out, and decoupling; FAANG-scale design requires schema registries, compatibility checks, unknown-field preservation, durable retention policy, consumer idempotency, and clear queue/topic semantics.

[Page 39]

- Replication keeps full dataset copies on multiple networked nodes to reduce geo-latency, increase availability/durability, and scale read throughput; hard part is propagating writes, not copying static data.
- Replication is not backup: replicas rapidly copy corruption/deletes too, while backups preserve historical recovery points; production systems need both replication logs/snapshots for HA and independent archival restore paths.
- Single-leader replication serializes writes through one primary and replays an ordered change log on followers; FAANG-scale pivot is choosing sync/async replication, follower-read freshness, failover semantics, and per-shard leadership without confusing read scaling with write scalability.

[Page 40]

- Synchronous replication confirms writes only after at least one follower/quorum has persisted them, improving failover durability; asynchronous replication preserves write availability under slow/failed/geographically distant followers but risks acknowledged-write loss.
- Adding followers requires a consistent snapshot plus exact replication-log position, then catch-up replay; production failure modes include stale snapshots, missing WAL/binlog retention, manual bootstrap errors, and lagged replicas promoted with incomplete history.
- Object-storage-backed databases shift durability/replication into S3/GCS/Blob-style systems and enable stateless compute, tiered storage, and zero-disk architectures; FAANG-scale trade-off is lower storage cost/ops simplicity versus higher latency, API-call cost, immutability constraints, WAL placement, batching, and cache design.

[Page 41]
- Leader-based HA handles follower outages via log-position catch-up, but leader failover requires failure detection, leader election/appointment, client rerouting, replica reconfiguration, and fencing of stale leaders.
- Failover is where replication semantics become operationally dangerous: async lag can lose acknowledged writes, stale followers can corrupt external systems, split brain can accept divergent writes, and aggressive timeouts can trigger cascading failovers under load.
- Replication-log design controls evolvability: statement replication is compact but fragile under nondeterminism; physical WAL shipping is exact but storage-version-coupled; logical row logs enable rolling upgrades, CDC, external indexes/caches, warehouses, and safer cross-version replication.

[Page 42]
- Asynchronous follower reads create replication-lag anomalies: users may miss their own writes, observe time moving backward across replicas, or see causally dependent writes out of order across shards.
- Stronger read guarantees map to concrete routing/state mechanisms: read-your-writes via leader/session-LSN reads, monotonic reads via sticky replica selection, and consistent-prefix reads via ordered replication or causal-dependency tracking.
- FAANG-scale pivot: read replicas scale throughput and reduce geo-latency only if product semantics tolerate staleness; otherwise use leader reads, bounded-lag replica selection, session consistency tokens, or strongly consistent transactional databases instead of silently exposing eventual-consistency bugs.

[Page 43]
- Multi-leader replication lets each region/device accept local writes and asynchronously exchange changes, improving write latency, offline availability, and regional/network-partition tolerance at the cost of weaker global constraints.
- The central failure mode is concurrent writes: LWW gives convergence by silently dropping updates, manual sibling resolution burdens users/apps, and automatic merge algorithms require datatype-aware semantics to avoid anomalies like resurrected deletes.
- FAANG-scale pivot: use multi-leader/local-first only when conflicts are acceptable or mergeable; enforce hard invariants like unique usernames, account balances, room booking exclusivity, and quota limits through single-leader routing, consensus, escrow/partitioned ownership, or explicit conflict-detection workflows.

[Page 44]
- Leaderless replication removes failover by letting clients/coordinators write/read from multiple replicas directly; quorum parameters n, w, and r trade latency/availability against probability of stale reads, with w + r > n giving overlap between read and write replica sets.
- Dynamo-style repair is eventual, not magic: read repair fixes hot keys, hinted handoff covers temporarily unavailable replicas, and anti-entropy reconciles background drift—but failed writes may partially persist, clocks may drop updates, rebalancing can break assumptions, and concurrent writes still conflict.
- FAANG-scale pivot: leaderless systems optimize tail latency and gray-failure resilience via parallel requests/request hedging, but they weaken linearizability; use them for availability-first workloads that tolerate stale/conflicting values, and monitor staleness/repair debt instead of treating quorum reads as strict freshness guarantees.

[Page 45]
- Multi-region leaderless replication routes writes through local coordinators and configurable consistency levels, trading cross-region latency against freshness: local quorum is fast but more stale-prone; global/regional quorums improve convergence visibility but pay WAN cost.
- Concurrent-write detection requires causal metadata, not wall-clock timestamps: happens-before defines whether one write supersedes another or whether both are siblings requiring merge; LWW converges cheaply but loses causally valid updates.
- FAANG-scale pivot: version vectors/dotted version vectors preserve causality across replicas and clients, enabling safe read-from-one/write-to-another workflows; use them when correctness requires distinguishing overwrites from conflicts, otherwise expect silent data loss under concurrent writes.

[Page 46]
- Sharding partitions a dataset/write workload across independent shards, while replication copies each shard for fault tolerance; together they form the standard shared-nothing scale-out pattern where each shard has leadership, replicas, ownership, and failure boundaries.
- Sharding buys horizontal write/data scalability but introduces partition-key coupling, cross-shard query fan-out, secondary-index complexity, distributed transactions, rebalancing pain, and hard-to-change placement decisions.
- Senior design rule: avoid sharding until single-node or single-leader limits are real; use it deliberately for write/data scale, tenant isolation, cell-based blast-radius control, residency/compliance, gradual migrations, and per-tenant recovery.

[Page 47]
- Shard-key choice determines whether scale-out works: range sharding preserves locality/range scans but risks hot shards on sequential keys; hash sharding spreads keys but destroys natural range locality.
- Rebalancing strategy is the operational core: hash(key) % N causes massive reshuffles, fixed shard counts make node changes cheaper but require upfront sizing, and hash-range/consistent hashing minimizes movement while allowing adaptive shard splits.
- Senior design rule: design partition keys around dominant access paths and hot-key behavior—timestamp-first keys melt one shard, celebrity/tenant skew breaks uniformity, oversized shards slow recovery, undersized shards add overhead, and resharding later is one of the most expensive distributed-database migrations.

[Page 48]
- Consistent hashing balances key count, not request heat; skewed workloads from celebrities, viral posts, large tenants, and ultra-hot objects can overload one shard even when hash distribution is uniform.
- Hot-key mitigation requires infrastructure-level isolation/adaptive capacity or application-level key splitting/salting; salting spreads writes but forces read fan-in, merge logic, hot-key metadata, lifecycle cleanup, and recovery complexity.
- Senior design rule: rebalancing is not free autoscaling—shard moves consume CPU/network/IO/cache and can amplify overload, so use throttling, observability, hysteresis, failure containment, and human-gated rebalancing for high-risk events.

[Page 49]
- Request routing maps key → shard → replica/node; unlike stateless service load balancing, sharded databases require shard-aware routing through forwarding nodes, routing tiers, or smart clients backed by authoritative placement metadata.
- Placement metadata is correctness-critical: ZooKeeper/etcd/Raft-backed config services prevent split-brain shard ownership, while weak gossip-based routing is only safe in systems designed for weak consistency.
- Senior design rule: shard movement is a protocol, not a config edit—copy data, replay logs, validate state, update routing atomically, fence old owners, monitor lag, and preserve rollback paths.

[Page 50]
- Secondary indexes break clean key-based sharding: local indexes make writes cheap but require scatter-gather reads, while global indexes make targeted lookups possible but introduce cross-shard write amplification, stale indexes, distributed transactions, and postings-list merge costs.
- Cross-shard queries and transactions are the tax paid for partitioning: they amplify tail latency, partial failure, coordination cost, and result-merge complexity.
- Senior design rule: shard for the operation that must be cheap; keep hot paths single-shard where possible, and treat cross-shard access as an explicitly budgeted exception.
[Page 51]
- Transactions group multiple reads/writes into a commit-or-abort unit, shifting partial failure, crash recovery, and many concurrency hazards from application code into database-managed safety guarantees.
- ACID is precise only in implementation details: atomicity means abortability, isolation ranges from weak levels to serializability, durability is risk reduction via WAL/fsync/replication/backups, and consistency mostly means application/database-declared invariants are preserved.
- Senior design rule: use transactions when denormalized state, secondary indexes, foreign keys, balances, counters, uniqueness, or cross-record invariants must move together; otherwise partial writes, duplicate retries, lost updates, and race conditions become application-owned distributed-systems debt.

[Page 52]
- Weak isolation is where “ACID” stops being enough: read committed prevents dirty reads/writes but still permits nonrepeatable reads, read skew, lost updates, write skew, and predicate-level concurrency bugs.
- Snapshot isolation/MVCC gives each transaction a consistent point-in-time database view while preserving “readers don’t block writers, writers don’t block readers” through row versions, transaction IDs, visibility rules, and delayed garbage collection.
- Senior design rule: use read committed only when temporary inconsistency is harmless; use snapshot isolation for backups, analytics, and long invariant-sensitive reads, but remember it is weaker than serializability and can permit write skew.

[Page 53]
- Isolation names are nonportable: “repeatable read,” “snapshot isolation,” and “serializable” differ across databases, so correctness must be based on actual anomaly guarantees, not product terminology.
- Lost updates arise from unsafe read-modify-write cycles; prevent them using atomic database operations, explicit locks, automatic lost-update detection, CAS/version columns, or commutative/merge semantics.
- Senior design rule: ORM-generated read-modify-write code is a silent correctness risk; counters, balances, JSON mutations, wiki edits, and replicated writes need explicit atomicity or conflict handling because LWW and weak MVCC can silently discard successful user updates.

[Page 54]
- Write skew occurs when concurrent transactions read the same predicate/invariant but update different rows, allowing each transaction to commit locally while the combined result violates a cross-row rule.
- Phantoms are predicate-level conflicts: an insert/update/delete changes the set of rows another transaction used to make a decision, and row locks cannot lock rows that do not yet exist.
- Senior design rule: prefer declarative constraints when possible; otherwise use serializable isolation, predicate/range locks, exclusion constraints, or carefully designed materialized conflict rows for invariants like uniqueness, booking exclusivity, nonnegative balances, and minimum on-call coverage.

[Page 55]
- Serializable isolation eliminates race conditions by guaranteeing equivalence to one-at-a-time execution; the cost depends on implementation: actual serial execution, 2PL, or SSI.
- Actual serial execution is simple and fast only for short, deterministic, stored-procedure-like, in-memory, single-shard transactions; cross-shard coordination and long waits destroy linear scalability.
- Two-phase locking provides true serializability by blocking conflicting readers/writers and preventing phantoms through predicate/index-range locks, but causes deadlocks, lock contention, unstable tail latency, and write stalls under long reads.
- Serializable Snapshot Isolation keeps MVCC-style nonblocking reads/writes while detecting stale reads and dangerous read-write dependency structures at commit; it trades blocking for abort/retry overhead and is valuable when retries are cheaper than production write-skew bugs.

[Page 56]
- Distributed transactions extend atomic commit across shards, global secondary indexes, databases, or brokers; a single-node commit record becomes a distributed atomic-commit protocol where partial commit is catastrophic.
- Two-phase commit turns prepare votes into durable participant promises and the coordinator’s logged decision into the commit point, but it is blocking: coordinator failure leaves prepared participants in doubt while holding locks.
- Senior design rule: internal distributed DB transactions can be engineered with consensus and shared protocols, but heterogeneous XA/2PC across unrelated systems is fragile due to fsync/network overhead, orphaned locks, coordinator failure, weak cross-system conflict detection, and manual recovery pressure.

[Page 57]
- Database-internal distributed transactions avoid XA’s weakest points because one system controls coordinator replication, shard replication, participant communication, consensus failover, timestamp assignment, and cross-shard concurrency control.
- NewSQL-style systems combine atomic commit with consensus-backed replication to provide snapshot isolation or serializability across shards when all participants share one database protocol.
- Senior design rule: exactly-once message effects usually do not require broker+database 2PC; store processed message IDs transactionally with business writes, enforce uniqueness, acknowledge after commit, and make retries idempotent. Distributed DB transactions matter when dedupe state and business state cannot be colocated.

[Page 58]
- Distributed systems fail by partial failure, not clean total failure: some nodes, links, queues, disks, or processes degrade while others continue, making outcomes nondeterministic and hard to reason about.
- Asynchronous networks provide no delivery, ordering, or timing guarantees; a timeout tells you only “no response yet,” not whether the request was lost, delayed, processed, or whether the response was lost.
- FAANG-scale pivot: every cross-node call must be designed for ambiguity—timeouts, retries, idempotency, deduplication, request tracing, backpressure, and failure injection are mandatory because one-in-a-million faults happen daily at fleet scale

[Page 59]
- TCP gives reliable byte-stream mechanics within one connection—packet ordering, retransmission, checksums, congestion control—but it does not give application-level certainty that a remote operation executed.
- A failed/closed connection leaves ambiguity: the request may be unsent, queued, received by kernel, processed by app, committed, or response-lost; reconnecting and resending can duplicate side effects unless the protocol is idempotent.
- FAANG-scale pivot: treat networks as arbitrarily delayed, asymmetric, and failure-prone—even inside datacenters; define and test timeout behavior, retries, recovery, partition handling, and fault injection because untested network-failure paths become outage multipliers.

[Page 60]
- Fault detection is probabilistic in asynchronous systems: absence of response cannot distinguish crash, overload, GC pause, packet loss, queueing, asymmetric link failure, or delayed response.
- Timeouts are a correctness/performance trade-off: short timeouts cause false failure detection and cascading failovers; long timeouts delay recovery, while packet-switched networks and shared CPUs provide no hard upper bound on delay.
- FAANG-scale pivot: tune failure detectors from observed latency distributions, jitter, and workload headroom—not intuition; use adaptive detectors, hysteresis, backoff, hedged requests, and chaos testing because overload-induced false positives can be worse than the original fault.

[Page 61]
- Distributed time splits into monotonic clocks for local durations/timeouts and time-of-day clocks for wall-clock timestamps; confusing them creates timeout bugs, negative durations, bad ordering, and broken observability.
- Time-of-day clocks are unreliable coordination primitives: NTP drift, clock jumps, leap seconds, VM pauses, misconfigured servers, firewalls, GPS/PTP failure, and user-controlled device clocks can make timestamps wrong or non-monotonic.
- FAANG-scale pivot: never base correctness on unsafely synchronized wall clocks; use monotonic clocks for elapsed time, logical clocks/version vectors for causality, server-assigned timestamps for audit/order where possible, and monitor clock skew as production infrastructure health.

[Page 62]
- Wall-clock timestamps are unsafe for ordering distributed events: clock skew can make causally later writes appear older, causing LWW systems to silently drop valid updates; use logical clocks/version vectors when causality matters.
- Clock uncertainty must be treated as an interval, not a point; systems like Spanner/TrueTime use bounded uncertainty and commit-wait to ensure timestamp order reflects causal order across datacenters.
- Leases, leadership, and timeouts are fragile under process pauses: GC, VM suspension, paging, disk stalls, scheduler delays, or SIGSTOP can make a node act on an expired lease after the cluster has already moved on.
- FAANG-scale pivot: never assume a process runs continuously between two lines of code; design leadership with fencing tokens, monitor clock skew, avoid correctness based on client clocks, drain traffic around GC/restarts, and treat long pauses as distributed-system failures, not local runtime trivia.

[Page 63]
- Distributed systems operate on inference, not certainty: nodes know remote state only through unreliable messages, so quorum decisions replace individual judgment when declaring leaders, failures, or ownership.
- Distributed leases are unsafe without fencing: paused clients, delayed packets, or zombie former leaders can act after lease expiry and corrupt state unless every operation carries monotonically increasing fencing tokens rejected by downstream services.
- FAANG-scale pivot: assume non-Byzantine but unreliable nodes—protect against stale actors with quorums, fencing tokens, CAS/preconditions, epoch/term numbers, and validation; reserve Byzantine fault tolerance for mutually untrusted or safety-critical systems because its cost is usually prohibitive in datacenter software.

[Page 64]
- System models make distributed reasoning tractable: synchronous assumes bounded delay/pause/skew, asynchronous assumes none, and partial synchrony with crash-recovery faults best matches production reality.
- Correctness separates safety from liveness: safety must never be violated even during partitions/crashes; liveness may depend on eventual recovery, majority availability, and bounded instability.
- FAANG-scale pivot: prove protocols against explicit assumptions, then attack implementations with TLA+/model checking, Jepsen-style fault injection, and deterministic simulation testing because real outages live in the gap between clean theory and messy runtime nondeterminism.

[Page 65]
- Linearizability makes replicated data appear as a single atomic copy: once any client observes a write, every later read by any client must observe that write or something newer, eliminating stale-replica surprises.
- Linearizability is a recency guarantee for single objects, while serializability is an isolation guarantee for multi-object transactions; strict serializability combines both but requires stronger coordination.
- FAANG-scale pivot: use linearizable operations for locks, leases, leader election, uniqueness, CAS, metadata, and user-visible “must be latest” reads; avoid paying its coordination cost on paths where read-after-write, monotonic reads, or bounded staleness are sufficient.

[Page 66]
- Linearizability is required for coordination-critical operations: leader leases, distributed locks, uniqueness constraints, CAS, stock/seat/account invariants, and cross-channel workflows where a message may race ahead of replicated storage.
- Implementation reality: single-leader systems are linearizable only if all reads/writes hit the true leader and failover cannot lose acknowledged writes; consensus systems can provide it, while multi-leader and Dynamo-style leaderless quorums generally cannot without extra coordination.
- FAANG-scale pivot: CAP is best read as “during a partition, choose linearizability or availability,” but the everyday cost of linearizability is latency—coordination delay grows with network uncertainty, so reserve it for correctness boundaries and use weaker/session/causal guarantees where product semantics allow.

[Page 67]
- Distributed ID generation trades uniqueness, ordering, latency, and fault tolerance: sharded IDs, preallocated blocks, UUIDs, Snowflake/ULID/ObjectID-style timestamp IDs scale well but do not provide linearizable creation order.
- Logical clocks order events by causality rather than wall time: Lamport clocks give compact total order consistent with happens-before, hybrid logical clocks approximate physical timestamps while preserving causal order, and vector clocks detect true concurrency at higher metadata cost.
- FAANG-scale pivot: use random/time-sortable IDs for scalable object creation, logical/HLC timestamps for MVCC and causality-aware ordering, but use a linearizable timestamp oracle or consensus-backed primitive when permissions, uniqueness, locks, leases, or cross-shard invariants require global real-time ordering.

[Page 68]
- Consensus turns “single-node primitives” into fault-tolerant distributed primitives: leader election, CAS, locks/leases, uniqueness, atomic increment, shared logs, state-machine replication, and atomic commit are all equivalent forms.
- FLP says deterministic consensus cannot guarantee termination in a fully asynchronous crash-prone model, but practical systems exploit partial synchrony, timeouts, quorums, and sometimes randomness to make progress in real deployments.
- Production consensus is usually implemented as a replicated shared log via Raft, Multi-Paxos, Zab, or Viewstamped Replication; every committed entry is synchronously replicated to a quorum before acknowledgement.
- Consensus is “single-leader replication done correctly”: epochs/terms prevent stale leaders, quorum overlap preserves committed log entries, and leader failover avoids split brain if the new leader is sufficiently up-to-date.
- The cost is quorum coordination on every write/linearizable read, majority availability requirement, timeout sensitivity, leader-election instability under bad links, and no throughput gain from simply adding more voting nodes.
- FAANG-scale pivot: use ZooKeeper/etcd/Consul-style coordination only for small, slow-changing metadata—leader leases, fencing tokens, shard ownership, config, membership—not high-QPS application data; cache service discovery because it rarely needs linearizability.

[Page 69]
- Batch processing treats large immutable inputs as read-only and produces fully regenerated derived outputs, enabling replay, rollback, time travel, human-fault tolerance, and side-effect-free recovery from bad code.
- Unlike online systems optimized for response time, batch systems optimize throughput and resource efficiency over minutes/hours/days, but pay with delayed outputs and expensive full recomputation when inputs change.
- FAANG-scale pivot: Unix pipelines preview the batch model—compose small deterministic stages, sort/merge when working set exceeds memory, spill sequentially to disk, and scale the same idea to Spark/Flink/warehouse engines over object storage for petabyte ETL, analytics, logs, and ML feature generation.

[Page 70]
- Distributed batch systems are “distributed operating systems”: they combine shared storage, schedulers, task executors, resource managers, and dataflow channels to run large jobs across many machines.
- Distributed filesystems shard large immutable files into large blocks across data nodes, replicate or erasure-code them for fault tolerance, and expose storage through HDFS/S3/NFS-like protocols.
- Object stores decouple compute from storage, scale cheaply, and fit immutable batch outputs, but lack POSIX semantics like atomic rename, file handles, locks, symlinks, and efficient random mutation.
- Job orchestrators such as YARN/Kubernetes schedule tasks under CPU/memory/disk/GPU constraints using heuristics, because optimal multi-resource allocation across competing jobs is computationally intractable.
- FAANG-scale pivot: batch fault tolerance comes from deterministic recomputation—retry failed/preempted tasks, isolate partial outputs, persist workflow boundaries, and choose between DFS materialization, Spark lineage recompute, or Flink checkpointing based on cost, latency, and failure rate.

[Page 71]
- MapReduce formalizes batch processing as input records → mapper key/value extraction → distributed sort/shuffle → reducer aggregation, using sorting to group identical keys without holding global state in memory.
- Dataflow engines like Spark and Flink generalize MapReduce into whole-workflow execution graphs, avoiding forced materialization between every stage, fusing operators, pipelining work, exploiting locality, and keeping intermediate state in memory/local disk.
- FAANG-scale pivot: shuffle is the core expensive primitive behind distributed joins, grouping, sorting, and aggregation; production performance depends on partitioning strategy, skew handling, spill behavior, network bandwidth, reducer sizing, intermediate-data durability, and avoiding unnecessary reshuffles.

[Page 72]
- Sort-merge joins scale distributed joins by shuffling both datasets on the join key so each reducer sees one key’s dimension row plus all matching fact rows, keeping memory bounded to one group at a time.
- Grouping and aggregation reuse the same primitive: repartition by grouping key, sort/group equal keys together, then reduce locally; most batch performance problems reduce to shuffle volume, skew, and intermediate-state size.
- FAANG-scale pivot: SQL/DataFrame APIs sit atop the same physical operators—shuffle, join, group-by, sort, scan—letting optimizers choose join order/algorithm and execution placement; use warehouses for convenience/interactive analytics, Spark/Flink for custom, iterative, multimodal, or cost-sensitive pipelines.

[Page 73]
- Batch use cases dominate where freshness can lag: ETL/ELT, reconciliation, analytics, ML feature engineering/training/inference, graph processing, LLM data prep, and periodic pre-aggregation all exploit immutable inputs and replayable outputs.
- Modern batch pipelines converge with lakehouse/warehouse systems: Spark/Flink/Trino/DuckDB/warehouse SQL, DataFrames, Iceberg/Delta catalogs, workflow schedulers, and BI tools all sit on the same scan–shuffle–join–aggregate execution foundation.
- FAANG-scale pivot: never let batch jobs write record-by-record into production serving stores; publish bulk outputs through streams, staged datasets, or bulk-import/version-swap mechanisms to preserve backpressure, isolation, all-or-nothing visibility, replayability, and production SLOs.

[Page 74]
- Stream processing is batch processing over unbounded input: events arrive continuously, so systems process each immutable event as it appears instead of waiting for a finite dataset/window to complete.
- Messaging systems decouple producers and consumers through topics/streams; the core production trade-off is overload behavior—drop, buffer, or backpressure—and durability behavior under consumer/broker failure.
- Traditional brokers use destructive delivery with acknowledgments/redelivery, making crashes cause duplicates and load-balanced consumers cause reordering; DLQs are mandatory to prevent poison messages from blocking progress forever.
- Log-based brokers like Kafka/Kinesis store events as append-only sharded logs with offsets, enabling fan-out, replay, independent consumers, high throughput, and batch-like recomputation of derived data.
- FAANG-scale pivot: choose partition keys to preserve required ordering, monitor consumer lag against retention, design consumers idempotently because offsets can replay messages, and use log retention/tiered object storage to turn streams into durable organizational data infrastructure.

[Page 75]
- Databases and streams are duals: mutable database state is the materialized result of an ordered changelog, while CDC/event sourcing expose that changelog as a durable stream for downstream systems.
- Dual writes to DB/cache/search/warehouse are unsafe because concurrent updates can arrive in different orders or partially fail; CDC fixes this by making the database the single leader and downstream systems ordered followers.
- CDC is operationally cheaper than event sourcing for existing apps, but it turns internal database schemas into public data contracts; use outbox tables to publish stable event schemas transactionally with domain writes.
- Immutable logs enable replay, auditability, derived views, schema evolution, CQRS, and recovery from bad code, but asynchronous consumers introduce lag and read-your-writes problems.
- FAANG-scale pivot: treat search indexes, caches, warehouses, timelines, and feature stores as rebuildable views over a compacted/replayable log; design for lag, schema compatibility, deletion/privacy, compaction limits, and versioned view cutovers

[Page 76]
- Stream processors consume immutable input streams read-only and emit derived output streams append-only; unlike batch, they must operate forever without global sort completion or “rerun from start” recovery.
- CEP stores long-lived queries/patterns and matches each incoming event against them, while stream analytics maintains rolling/windowed aggregates, metrics, percentiles, sketches, and anomaly signals over high-volume event flows.
- Correct stream-time semantics require event time, not processing time; processing-time windows create artifacts during lag, redeploys, backlogs, retries, and reprocessing.
- Windowing makes unbounded streams finite enough to aggregate: tumbling/hopping/sliding/session windows trade smoothing, latency, state size, and late-event handling.
- Stream joins require state: stream–stream joins buffer bounded windows, stream–table joins maintain local CDC-fed replicas, and table–table joins maintain materialized views from changelogs.
- FAANG-scale pivot: exactly-once stream processing requires checkpointed state, replayable logs, atomic offset/output commits or idempotent writes, deterministic processing, fencing on failover, and recoverable local state via snapshots, changelog topics, or replay.
