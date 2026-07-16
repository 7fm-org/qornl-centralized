# Qornl: Architecture & Design Manual
Version: 1.0.0
Classification: Core Architectural Specification

---

## 1. Executive Summary: The Universe Container

Qornl operates as an empty runtime container called the **Universe**. It coordinates several domain-free mechanisms that remain completely static until fed declarations (Metadata) at runtime. The architecture is built on a strict layered hierarchy where dependencies point exclusively downward, protecting the core from execution-level failures.

### The Universe Architecture Matrix:
* **The Realm**: A runtime partition identifier separating environments.
* **The Definition Table**: Represents the metadata storage where all runtime kind definitions (shapes, parent relations) live.
* **The Registry**: Houses registered worker behaviors and executable interfaces.
* **The Journal**: The append-only event log used to drive state updates.
* **The Content Store**: Content-addressed blob storage for code and script files.
* **The Reconciler Loop**: Autonomic feedback controller maintaining target state.

---

## 2. Core Architectural Layering

Qornl separates responsibilities across a strict **7-Layer Stack** to enforce system safety, high performance, and language neutrality:

```
┌───────────────────────────────────────────────────────────────────────┐
| Layer 7: Language SDKs    --> Python, Node.js, Go, Rust APIs          |
├───────────────────────────────────────────────────────────────────────┤
| Layer 6: Worker Runner    --> Polls queue leases & executes tasks     |
├───────────────────────────────────────────────────────────────────────┤
| Layer 5: SDK Core Router  --> Endpoint path resolution (/poll)        |
├───────────────────────────────────────────────────────────────────────┤
| Layer 4: Contract Gateway --> Validates schema constraints & revisions|
├───────────────────────────────────────────────────────────────────────┤
| Layer 3: Bridge Router    --> UNIX domain socket connections & checks  |
├───────────────────────────────────────────────────────────────────────┤
| Layer 2: Projection       --> Streams updates asynchronously to Neo4j |
├───────────────────────────────────────────────────────────────────────┤
| Layer 1: Authority Core   --> Universe state, RocksDB WAL, Replay loop |
└───────────────────────────────────────────────────────────────────────┘
```

---

## 3. Fundamental Design Principles

### Principle 1: Late Binding & Live Delegation
To enable instant system changes without code redeployment, Qornl replaces compiled inheritance with **prototype-based delegation**:
* **Referenced Parentage**: Derived definitions do not duplicate attributes or behaviors from their parents; they point directly upward via parent delegation links.
* **Call-Time Lookup**: Properties are resolved dynamically at the moment of read by walking up the ancestry chain.
* **Instant Propagation**: A modification to any base blueprint is instantly visible to all descendants at every level in the concept tree, with no code modifications required.

### Principle 2: Metadata-Driven Structure (AOM)
All system entities, properties, and relationships exist as dynamic metadata entries (the **Adaptive Object-Model**):
* **Kinds as Data**: Kinds of things (`user`, `inventory-item`, `vehicle`) are stored as runtime database records, not as hard-coded compiled classes.
* **Property Pattern**: Variables and object attributes are represented as open-ended name-value pairs, bypassing table column constraints.
* **Open-Closed Principle**: The running system is infinitely extensible by adding new metadata definitions, but the engine source code itself is never modified.

### Principle 3: Ephemeral Scheduling
Task delivery scheduling and worker leases are kept in-memory to maximize throughput. State durability is separated:
* **Durable Registry**: All transaction facts and completed outputs are durably written to the RocksDB journal.
* **Ephemeral Heap**: The active queue memory remains transient. If the authority restarts, the queue drains, and the client SDK re-queues pending items from the transaction checkpoint.
