# Qornl Centralized Repository

Welcome to the centralized entry point and monorepo pointer directory for the **Qornl** orchestration ecosystem. This repository aggregates all components of Qornl (from Layer 1 database engines to Layer 7 client SDK runtimes) as **Git Submodules**.

---

## 🗺️ System Codebase Map & Submodules

| Submodule Folder | Architectural Layer | Remote Git Origin | Key Responsibilities |
| :--- | :--- | :--- | :--- |
| **`qornl/`** | Layer 1 & 2 (Core & Projections) | `https://github.com/7fm-org/qornl.git` | RocksDB ledger snapshots, event replay, and Bolt Neo4j updates. |
| **`qornl-bridge/`** | Layer 3 (IPC IPC Framing) | `https://github.com/7fm-org/qornl-bridge.git` | Unix domain sockets, checksums, and conduit message buffers. |
| **`qornl-contract/`** | Layer 3/4 (Schemas & AST) | `https://github.com/7fm-org/qornl-contract.git` | Versioned envelope configurations and AST parse tokens (`apiVersion: qornl/v1`). |
| **`qornl-sdk-core/`** | Layer 4/5 (Gateway Foundation) | `https://github.com/7fm-org/qornl-sdk-core.git` | Common retry logic, request IDs, and gateway route translators. |
| **`qornl-workers/`** | Layer 6 (Out-of-process Executors) | `https://github.com/7fm-org/qornl-workers.git` | Queue leases, Command runner processes, and content-addressed Units. |
| **`qornl-stack/`** | Orchestrator (Builder & Launcher) | `https://github.com/7fm-org/qornl-stack.git` | Building binaries, verifying checksums (`SHA256SUMS`), and launch orchestration. |
| **`qornl-clients/`** | Layer 7 (Product Clients) | `https://github.com/7fm-org/qornl-clients.git` | Reference CLI implementations for product integrations. |
| **`qornl-sdks/`** | Layer 7 (Developer APIs) | `https://github.com/7fm-org/qornl-sdks.git` | Python, Node.js, Go, and Rust standard libraries. |

*Note: Sibling development environments can bypass submodule checkouts by setting `export QORNL_STACK_SOURCE_ROOT=/home/sagarjv/` in local workspaces.*

---

## 📖 Centralized Architectural Index

We have added comprehensive, Azure-grade architectural guides to the submodules. Refer to the specific documents below for low-level design specifications:

### 1. Database & Projection Indexes (Layer 1 & 2)
Read **[qornl/docs/ARCHITECTURE_L1_L2.md](qornl/docs/ARCHITECTURE_L1_L2.md)** to understand:
* In-memory universe struct mappings.
* RocksDB snapshot logic and WriteBatch commits.
* Failsafe journal verification playbacks.
* Unix projection sockets (`QPRJ`) and Bolt Neo4j Cypher insertions.

### 2. Unix Framing & Schemas (Layer 3 & 4)
Read **[qornl-bridge/docs/ARCHITECTURE_L3_L4.md](qornl-bridge/docs/ARCHITECTURE_L3_L4.md)** to understand:
* 14-byte `QORN` socket headers and Fowler-Noll-Vo FNV-1a checksums.
* Conduit message stream ceilings and memory allocations.
* Virtual IR (VIR) AST tokenizer syntax rules.
* Flow behavior binding compilations.

### 3. API Routers & Subprocess Execution (Layer 5 & 6)
Read **[qornl-workers/docs/ARCHITECTURE_L5_L6.md](qornl-workers/docs/ARCHITECTURE_L5_L6.md)** to understand:
* Endpoint route matching rules.
* Split-state scheduling: durable RocksDB WAL transactions vs. in-memory ephemeral heaps.
* Subprocess piping for system command executors.
* Content-addressed cache materialization and environment variable stripping for Unit runtimes.

### 4. Client APIs & Resumption (Layer 7)
Read **[qornl-sdks/docs/CLIENT_SDK_REFERENCE.md](qornl-sdks/docs/CLIENT_SDK_REFERENCE.md)** to understand:
* Complete API references for Go, Node, Python, and Rust.
* Language-native socket framing and FNV-1a checksum loops.
* Best practices for designing client-side queue recovery states.

---

## 🛠️ Monorepo Quickstart

To clone this centralized repository and fetch all submodules automatically, run:

```bash
# Clone the central hub
git clone --recursive <repository-url>

# If already cloned, initialize the submodules
git submodule update --init --recursive
```
