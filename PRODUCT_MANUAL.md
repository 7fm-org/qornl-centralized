# Qornl: The Self-Maintaining Dynamic Concept Lattice
### Professional Product Manual & Technical Specification
Version: 1.0.0-PROD
Classification: Core Product Specification

---

## Executive Overview: The Philosophy of the Void

Traditional software engineering forces a rigid separation between **code** (compiled structures) and **data** (transient parameters). This design pattern breaks down under dynamic, hot-evolving, or low-code environments. When business rules, data schemas, or relationship hierarchies change, developers are forced to modify source code, rebuild binaries, and redeploy containers.

**Qornl** is a runtime container—referred to conceptually as the **Universe**—that operates as an empty canvas (the **Void**). It wires together domain-free mechanisms that remain completely static until fed definitions at runtime. Structure and behavior are represented entirely as data (Metadata), turning the system into a **Self-Maintaining Dynamic Concept Lattice**.

```
    [ Metadata Envelopes ] ---> [ Universe (The Void) ] <--- [ Ephemeral Queues ]
                                       |
                   +-------------------+-------------------+
                   |                   |                   |
                   v                   v                   v
            Lattice (FCA)      Ledger (RocksDB)      Reconciliation
             Subsumption          Event Log            MAPE-K Loop
```

---

## 1. Core Architectural Pillars

Qornl's product engine is built upon four fundamental architectural pillars:

### Pillar 1: The Adaptive Object-Model & EAV Core (The Void)
Instead of compiling classes for every resource, Qornl represents the structural schema itself as data.
* **Type-Object Pattern**: Separates an entity's structure (its `Definition` type-object) from its concrete instances (`Instance`). A change to the definition is immediately visible to all instances without code changes.
* **Property Pattern**: Objects carry an open-ended collection of name-value pairs (`Prop` nodes) instead of a fixed, pre-defined set of compiled fields.
* **Entity-Attribute-Value (EAV)**: Database schemas are represented as `(entity, attribute, value)` relations. In Qornl, this is materialized through the **EAV Graph Projection** in Neo4j, where all nodes and parameters translate to modular resource keys.

### Pillar 2: Prototype-Based Inheritance with Live Delegation
To enable zero-compile changes, Qornl utilizes prototype-based inheritance:
* **Delegation instead of Copying**: Derived definitions point upward to their parents. No fields or properties are ever copied down the chain.
* **Late Binding**: Lookups walk up parent delegation links (`Ancestry.chain`) at the exact moment of execution. If a base property at the root of the lattice changes, the modification propagates instantly to all descendants across any depth.

### Pillar 3: Self-Rearranging Concept Lattice (FCA & Subsumption)
A node's position in the hierarchy is never declared statically by a developer; it is dynamically inferred from what the node is:
* **Formal Concept Analysis (FCA)**: Properties on an object induce a concept lattice. The system computes the correct parent-child relationships mathematically.
* **Predicate Dispatch**: The moment a property is added or removed during execution, the system evaluates which parent subset the node belongs to and automatically relocates the node (and its entire subtree) to its rightful parent.
* **Incremental Classification**: Rather than recomputing the entire world, OWL-style reasoners calculate only the affected region of the lattice in milliseconds.

### Pillar 4: Autonomic Reconciler Loop (MAPE-K Paradigm)
Consistency is maintained by an autonomic feedback loop:
* **Desired vs. Observed**: The system perpetually compares the *desired state* (declared in metadata envelopes) against the *observed state* (reported from execution workers).
* **Merkle DAG Snapshots**: State revisions are recorded as copy-on-write snapshots with unique content hashes. Reparenting a node is as cheap as updating its Merkle root hash.
* **Event Sourcing**: Current state is derived by folding append-only transactional events over the journal ledger, preserving a historical timeline of structural mutations.

---

## 2. Layered Product Architecture

Qornl segregates responsibilities across 7 layers to protect performance, language neutrality, and data isolation:

```
+-----------------------------------------------------------------------------+
| Layer 7: Language Clients  --> Python, Node.js, Go SDK Client Runtimes      |
+-----------------------------------------------------------------------------+
| Layer 6: Worker Runner     --> Polling queue leases & running subprocesses  |
+-----------------------------------------------------------------------------+
| Layer 5: SDK Core Router   --> Translating endpoints (/poll, /complete)     |
+-----------------------------------------------------------------------------+
| Layer 4: Contract Gateway  --> Validating Envelope schemas and revisions   |
+-----------------------------------------------------------------------------+
| Layer 3: Bridge Router     --> UNIX socket binary headers & FNV-1a checksums|
+-----------------------------------------------------------------------------+
| Layer 2: Projection        --> Async streaming updates to Neo4j Graph DBs   |
+-----------------------------------------------------------------------------+
| Layer 1: Authority Core    --> Universe state machine & RocksDB ledger WAL   |
+-----------------------------------------------------------------------------+
```

---

## 3. Reference Implementation & Commands

### 3.1 Local Compilation
To build the compiled, cryptographically signed binaries of the Qornl stack:
```bash
cd qornl-stack
export QORNL_STACK_SOURCE_ROOT=$(pwd)/
cargo run -- build
```

### 3.2 Running the Void Server
Launch the Layer 1 authority core:
```bash
QORNL_SOCKET_PATH=/tmp/qornl.sock QORNL_STORAGE_DIR=/tmp/qornl-state ./bundle/bin/qornl
```

### 3.3 Dynamic Subprocess Execution
Workers pull task leases and execute commands using stdin/stdout streams. Save a behavior configuration (`worker.yaml`):
```yaml
name: core-worker
queue: work
socket: /tmp/qornl.sock
handlers:
  - behavior: run-command
    kind: command
    command: ["sh", "-c", "read input; echo $input | jq -r '.args'"]
```
Launch the worker runner:
```bash
./bundle/bin/qornl-worker-runner --config worker.yaml
```

---

## 4. Academic Precedents

Qornl's architecture builds directly upon decades of computer science research:
1. **Adaptive Object-Models**: *Riehle, D. (1997)*. Outlines storing metadata as a runtime data engine to build highly adaptable business software.
2. **Calculus of Fickle**: *Drossopoulou, S., et al. (2001)*. Establishes mathematical proofs for dynamic object reclassification under type-safe bounds.
3. **MAPE-K (Monitor-Analyze-Plan-Execute over Knowledge)**: *Kephart, J. O., & Chess, D. M. (2003)*. Defines self-adaptive software control loops.
