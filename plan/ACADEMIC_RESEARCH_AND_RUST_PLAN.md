# Qornl: Advanced Incremental Engine & Subsumption Lattice
### Academic Foundations & Rust Implementation Blueprint
Version: 1.0.0-ACADEMIC
Classification: Core Design Blueprint

---

## 1. Literature Review & Academic Foundations

The design of a self-maintaining dynamic concept lattice that relocates itself at runtime based on structural predicate checks is built on several key academic papers in computer science.

```
                  +-----------------------------------+
                  |      IBM Autonomic Computing      |
                  |          (MAPE-K Loop)            |
                  +-----------------------------------+
                                    |
                                    v
+-----------------------+ +-----------------------+ +-----------------------+
|    FCA Concept Math   | |   DBSP Stream Math    | |     Salsa Queries     |
|   (Ganter & Wille)    | |    (VMware Research)  | |    (Rust-Analyzer)    |
+-----------------------+ +-----------------------+ +-----------------------+
                                    \     |     /
                                     v    v    v
                  +-----------------------------------+
                  |           QORNL ENGINE            |
                  +-----------------------------------+
```

### 1.1 DBSP: A Declarative Foundation for Incremental Computation
* **Academic Reference**: *DBSP: A Declarative Foundation for Incremental Computation* (Budiu, Chajczyk, McSherry, Ryzhyk; VMware Research).
* **Key Concept**: DBSP defines a mathematical framework for incremental computation over collections. Instead of executing complete database queries over whole data tables on every update, DBSP models computations as **circuits** processing **streams of changes (deltas)**.
* **Qornl Relevance**: When a definition's characteristics change, the `IncrementalClassifier` propagates the delta dynamically. By calculating only the diff, Qornl avoids O(n) reclassification sweeps, reducing runtime relocations to O(k) where k is the size of the mutated subtree.

### 1.2 Dynamic Object Reclassification & Predicates
* **Academic Reference**: *Fickle: A Calculus of Dynamic Object Reclassification* (Drossopoulou, Damiani, de'Liguoro, Giannini; ACM).
* **Key Concept**: Fickle defines a type-safe mathematical calculus for objects that change their class membership at runtime while preserving their physical pointer identity. Predicate classes (Chambers, 1993) go further: class membership is not declared, but computed dynamically as a predicate over the object's properties.
* **Qornl Relevance**: A flow configuration is a predicate-dispatch object. If a property shifts, the predicate shifts, and the reclassification engine automatically re-parents the concept without memory ruptures or pointer invalidations.

### 1.3 Formal Concept Analysis (FCA) & Description Logics
* **Academic Reference**: *Formal Concept Analysis: Mathematical Foundations* (Ganter, Wille; Springer).
* **Key Concept**: FCA mathematically proves that any set of objects and attributes induces a unique, stable concept lattice. OWL ontology engines utilize Description Logic reasoners to classify definitions by calculating subsumption relationships (determining subsets and supersets).
* **Qornl Relevance**: Instead of developer-declared hierarchy folders, Qornl treats the entire system taxonomy as a concept lattice dynamically inferred by a reasoner.

---

## 2. Premium Corporate Product Analogs

### 2.1 Salesforce Metadata-Driven Architecture
Salesforce's multitenant database engine is a major corporate implementation of the **Adaptive Object-Model**:
* **Data-Driven Schemas**: Rather than compiling database tables for custom fields, Salesforce stores field layouts and object definitions as rows in metadata tables.
* **Runtime Materialization**: A virtual runtime engine parses metadata on the fly to build page layouts, run calculations, and evaluate custom validation rules.
* **EAV Mappings**: The underlying physical database uses a unified, indexes-heavy Entity-Attribute-Value (EAV) storage model to bypass traditional relational database limitations.

### 2.2 Rust-Analyzer & the Salsa Query Engine
Salsa is a pull-based incremental computation framework used by production-grade tools like Rust-Analyzer (the Rust language server):
* **On-Demand Memoization**: Instead of pushing updates blindly, Salsa tracks query dependencies. When a user types a character in their editor, the IDE requests compile details. Salsa queries walk up the AST dependency graph, checking if parent keys have changed. If unchanged, it returns cached calculations instantly.
* ** bening cycles detection**: Salsa catches compilation cycles and returns clean diagnostics instead of stack overflows.

---

## 3. High-Performance Rust Implementation Blueprint

To port 7FM's Scala classification engine to Rust, Qornl will combine **Salsa-like query tracking** with **DBSP-like delta propagation**:

```
 [ Input Definitions ] ---> [ Query Database ] ---> [ Dependency Graph ]
                                                          |
                                                 (Tracks triggers)
                                                          |
                                                          v
                                                 [ Re-classify Delta ]
```

### 3.1 The Memory Layout: Intrusive Indices
To meet millisecond performance constraints across millions of concepts, the Rust engine will bypass traditional tree pointers (`Rc<RefCell<Node>>`) in favor of a data-oriented **intrusive index layout**:
```rust
// modules/lattice/src/index.rs

pub struct LatticeIndex {
    // Dense flat arrays indexed by Name ID (interned u32)
    desired_parents: Vec<i32>,
    declared_parents: Vec<i32>,
    first_children: Vec<i32>,
    next_siblings: Vec<i32>,
    prev_siblings: Vec<i32>,
}
```
* **Performance Gain**: Moving a node between parent concepts is reduced to updating index slots in flat arrays, ensuring high cache locality and zero heap allocations.

### 3.2 The Dependency Graph & Query Registry
To implement incremental re-evaluation:
* **Forward Edges**: A dependency graph (`depGraph`) maps property keys to the list of definition IDs that read them.
* **Delta Propagation**: When a definition updates its property:
  1. The classifier queries the `depGraph` to find all dependent definitions.
  2. The system expands the affected region to include the changed node and its children.
  3. The classification pass is run only on the affected region.

### 3.3 Autonomic MAPE-K Controller
The control loop acts as the autonomic supervisor:
```rust
// modules/reconcile/src/controller.rs

pub struct ControlLoop {
    desired: LatticeIndex,
    observed: Keyspace,
}

impl ControlLoop {
    pub fn reconcile(&mut self, universe: &mut Universe) -> Result<(), Error> {
        // Monitor: Get placements from Keyspace
        let current_placements = self.observed.scan(b"placement/")?;
        
        // Analyze & Plan: Diff desired layouts with current placements
        let drift = self.calculate_drift(&universe.table, current_placements)?;
        
        // Execute: Relocate nodes dynamically and append ReHomed facts to RocksDB
        for relocation in drift {
            self.execute_move(universe, relocation)?;
        }
        Ok(())
    }
}
```
This reconciler loop ensures that the physical arrangement converges dynamically to the inferred lattice position calculated by the reasoner.
