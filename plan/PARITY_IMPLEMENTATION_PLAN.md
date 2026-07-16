# Qornl Parity Implementation Plan: Dynamic Subsumption & Live Delegation
Version: 1.0.0
Classification: Core Design & Implementation Plan

---

## 1. Executive Summary & Project Parity Analysis

This plan defines the development lifecycle for migrating the core dynamic classification and prototype delegation features from the original **7FM Scala Core** to the Rust-based **Qornl Core** (`qornl`).

During the initial Scala-to-Rust migration, previous AI agents focused on socket framing (`qornl-bridge`), task workers (`qornl-workers`), and basic RocksDB snapshot persistence (`qornl`). Due to context truncation, the **Self-Rearranging Concept Lattice (FCA)** and **Live Prototype Delegation Lookup** features were omitted or simplified. 

This document outlines the exact modules, math models, and code transitions required to bridge these gaps, ensuring full feature parity across all 7 layers of the system.

---

## 2. Technical Parity Matrix: Scala vs. Rust

The table below contrasts the original Scala implementation details with our planned Rust counterparts:

| Feature | 7FM Scala Module & Code | Qornl Rust Crates (Target) | Parity Implementation Details |
| :--- | :--- | :--- | :--- |
| **Lattice Classification** | `Classifier.scala` | `modules/lattice` | Evaluates membership conditions dynamically to decide a definition's parent concept. |
| **Predicate Evaluation** | `Predicates.scala` | `modules/lattice` | Implements conditional checks (`Eq`, `Gt`, `StartsWith`, `And`, `Not`) over resolved properties. |
| **Bulk Resolution** | `BulkClassifier.scala` | `modules/lattice` | Dynamic programming lookup. Walks up parent chains, resolves props, memoizes shared ancestors, and chunks work across OS thread pools. |
| **Live Delegation** | `Properties.resolved` | `modules/state` & `modules/record` | Reads properties by walking parent delegation links at call time instead of statically caching them on boot. |
| **MAPE-K Reconciler** | `ControlLoop.scala` | `modules/reconcile` & `modules/boot` | Autonomic feedback controller. Compares structural desired layouts with actual observed layouts, issuing delta re-parent updates. |

---

## 3. Structural Specification of the Missing Components

### 3.1 Predicate Logic & Expression Tree (`modules/lattice`)
We must define a syntax for expressions in Rust to support predicate matching.

```rust
// modules/lattice/src/predicate.rs

#[derive(Clone, Debug, Serialize, Deserialize)]
pub enum Predicate {
    Has(String),
    Absent(String),
    Eq(String, Scalar),
    Neq(String, Scalar),
    Lt(String, Scalar),
    Lte(String, Scalar),
    Gt(String, Scalar),
    Gte(String, Scalar),
    StartsWith(String, String),
    EndsWith(String, String),
    Contains(String, String),
    And(Box<Predicate>, Box<Predicate>),
    Or(Box<Predicate>, Box<Predicate>),
    Not(Box<Predicate>),
}

#[derive(Clone, Debug, Serialize, Deserialize, PartialEq)]
pub enum Scalar {
    Num(i64),
    Real(f64),
    Text(String),
    Bool(Boolean),
}
```

### 3.2 The Classification Reasoner (`modules/lattice/src/classify.rs`)
The reasoner evaluates the logical predicates against a definition's resolved properties:
1. Fetch all conditional membership clauses (`Clause { when: Predicate, parent: String }`).
2. Evaluate predicates sequentially. The first predicate that returns `true` dictates the new parent concept.
3. If no predicate evaluates to `true`, fall back to the declared `metadata.parent` or the default Root concept (`⊥`).

### 3.3 Bulk & Incremental Classification (`modules/lattice/src/bulk.rs`)
To support re-classification in milliseconds across thousands of concepts:
* **Memoization Table**: Cache resolved attributes (`Map<String, Scalar>`) and inherited clauses (`Vec<Clause>`) for every visited concept.
* **Upward Memoized Walk**: When resolving a concept, walk upward until we hit a cached ancestor, then fold the properties back down.
* **Work-Stealing Fanout**: Chunk the list of definitions and resolve them in parallel using Rust's `tokio` multi-threaded executor or standard OS thread-stealing pools.
* **Incremental Scope**: Reclassify only the changed definition and its direct down-tree descendants (its delta region), leaving the rest of the lattice untouched.

---

## 4. Phase-by-Phase Integration Plan

```
[ Phase 1: Core Lattice Math ] -> [ Phase 2: Live Chain Resolution ] -> [ Phase 3: Autonomic Reconciler ] -> [ Phase 4: API & Client Parity ]
```

### Phase 1: Core Lattice Math & Predicate Dispatch (Layer 1)
* **Goal**: Build the predicate engine and classification reasoner in Rust.
* **Tasks**:
  1. Create `modules/lattice/src/predicate.rs` and `modules/lattice/src/classify.rs`.
  2. Implement FNV-1a checksum validation and AST builders for predicate expressions.
  3. Write unit tests for matching ranges, string checks, and logical expressions.

### Phase 2: Live Chain Resolution & Snapshot Sharing (Layer 1)
* **Goal**: Integrate late-binding property lookup inside the authority's `Universe`.
* **Tasks**:
  1. Modify the `Universe` model in `modules/record` to support dynamic delegation lookups.
  2. Implement copy-on-write structural sharing for in-memory property maps.
  3. Implement the memoized upward walk algorithm in `modules/lattice/src/bulk.rs`.

### Phase 3: Autonomic Reconciler Integration (Layer 1 & 2)
* **Goal**: Connect the concept reasoner to the feedback control loop.
* **Tasks**:
  1. Modify `modules/reconcile` to compare the *computed* structural placements with the *actual* RocksDB ledger parentage.
  2. Implement delta moves to relocate subtrees.
  3. Verify that Neo4j graph projections update automatically when a concept relocates.

### Phase 4: Sockets, Contracts, and Client SDK Parity (Layers 3–7)
* **Goal**: Expose the dynamic re-classification APIs all the way to client code.
* **Tasks**:
  1. Add UNIX socket routing endpoints `/queues/work/reclassify` inside `modules/gate`.
  2. Register validation contract schemas in `qornl-contract`.
  3. Add programmatic execution methods (e.g. `client.reclassify(definition_name)`) in `qornl-sdks/python-client`.
  4. Write a comprehensive execution example proving that adding a property to a base concept dynamically shifts all children instantly without code recompiles or redeployments.
