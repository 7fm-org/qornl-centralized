# Qornl: Storage & Reconciliation Engine
Version: 1.0.0
Classification: Core Persistence & Control Loop

---

## 1. Storage Parity & Snapshot Model

Qornl separates state persistence from state delivery. It utilizes an immutable, copy-on-write image snapshot model mapped onto RocksDB.

```
       [ Universe State Machine ]
                   |
        (Serialized Snapshots)
                   |
                   v
       +-------------------------+
       |   RocksDB WriteBatch    | (Atomic STATE and DIGEST key updates)
       +-------------------------+
                   |
                   v
       +-------------------------+
       |   Durable Journal Log   | (Append-only write-ahead log)
       +-------------------------+
```

### 1.1 Structural Sharing
To enable fast memory snapshots:
* Nodes are represented as copy-on-write layered snapshots. A child does not duplicate its parent's data; it layers on top of it, sharing unchanged bytes and recording only its structural delta.
* Content-addressable storage identifies every snapshot by the hash of its content. If two nodes have the same properties, they share the same physical memory space.

### 1.2 Merkle DAG Verification
Qornl models the lineage tree as a **Merkle Directed Acyclic Graph (DAG)**:
* A change in any leaf node recalculates its hash signature, which in turn recalculates the parent's hash, rippling the fingerprint updates all the way back to the root of the lattice.
* Because parentage is content-addressed, verifying that a node matches its parent is as cheap as comparing their calculated hash digests.

---

## 2. Autonomic Reconciliation Loop (MAPE-K)

State consistency is enforced by an autonomic control feedback loop (`ControlLoop`) operating on the **MAPE-K (Monitor-Analyze-Plan-Execute over Knowledge)** paradigm:

```
        +--------------+      Desired State (Flow definition)
        |  Reconciler  | <-----------------+
        +-------+------+                   |
                |                          |
          (Diffs states)                   |
                |                          |
                v                          |
        +--------------+                   |
        |  Controller  | ------------------+
        +-------+------+
                |
          (Issues moves)
                |
                v
        Observed Layout (Queues and workers)
```

1. **Monitor**: Tracks the actual runtime topology (where task workers and leases sit).
2. **Analyze**: Diffs the *observed state* against the *desired state* (declared in flow envelopes).
3. **Plan**: Computes the minimal set of transitions (enqueuing, re-leasing, or re-parenting relocations) required to align the observed state with the desired state.
4. **Execute**: Issues the required execution actions to the gateways.

---

## 3. Event Sourcing & Recovery

All changes to the state are captured as an append-only transaction ledger (Journal):
* **No In-Place Edits**: The authority never updates historical values in place; it appends a new event fact to the ledger and writes the successor snapshot.
* **State Parity Recovery**: During startup or crash recovery, the authority loads the RocksDB snapshot, replays the journal log, and folds each event sequentially to rebuild the lineage state.
