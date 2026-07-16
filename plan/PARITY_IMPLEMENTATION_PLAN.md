# Qornl Parity Implementation Plan: Core 7FM Pipeline & Autonomic Reconciler
Version: 2.0.0
Classification: Core Design & Implementation Plan

---

## 1. Project Parity Audit: Scala `feature/production-grade` vs. Rust `qornl`

An in-depth analysis of the Scala codebase (`feature/production-grade` branch) reveals that the current Rust implementation of Qornl has omitted the core compilation and autonomic modules, replacing them with a simplified JSON message queue. 

To restore the original "Void" design, we must port the five-stage compiler pipeline and the MAPE-K control loop.

```
 [ .seed Text File ]
         |
         v
+------------------+
| 1. SeedParser    | (modules/ir/parse)  --> Parses text into VNode AST
+------------------+
         |
         v
+------------------+
| 2. Resolver      | (modules/ir/resolve) --> Validates parents & cycle checks
+------------------+
         |
         v
+------------------+
| 3. Normalizer    | (modules/ir/normal)  --> Catalyst-style rule-based rewriting
+------------------+
         |
         v
+------------------+
| 4. Planner       | (modules/ir/plan)    --> Compiles execution steps
+------------------+
         |
         v
+------------------+
| 5. Interpreter   | (modules/catalog)    --> Walk & execute behavior registry
+------------------+
```

---

## 2. Compilation Pipeline Parity

### 2.1 Stage 1: SeedParser (`modules/ir/src/parse.rs`)
* **Goal**: Parse `.seed` files containing domain declarations:
  ```seed
  define perishable-item extends item
    prop shelf-days = 14
    attach reorder-check
    clause has origin -> imported-item
  ```
* **Rust target**: Build a recursive-descent parser mapping text directly to the `VNode` AST enum.

### 2.2 Stage 2: Resolver (`modules/ir/src/resolve.rs`)
* **Goal**: Validate structure before database commits.
* **Rust target**: Traverse definitions and enforce:
  1. Every parent referenced by a child exists in the `Universe` table.
  2. No cyclical inheritance loops exist (fail-closed check).

### 2.3 Stage 3: Normalizer (`modules/ir/src/normal.rs` & `modules/ir/src/rule.rs`)
* **Goal**: Spark Catalyst-style rule optimization.
* **Rust target**: Implement a Rule Executor running rewrites to a fixed point:
  * Flat property mappings (overrides merge, last wins).
  * Ancestry path resolving.

### 2.4 Stage 4 & 5: Planner & Interpreter (`modules/ir/src/plan.rs` & `modules/catalog`)
* **Goal**: Map dynamic execution behavior.
* **Rust target**: 
  * Compile executable paths into a parallelizable `Plan` containing execution `PlanStep` nodes.
  * Implement the Interpreter walking the plan steps and resolving handler mappings against the dynamic Registry.

---

## 3. Autonomic Reconciler Loop (MAPE-K)

We must restore the structure-driven classification loop inside the authority:

```rust
// modules/reconcile/src/loop.rs

pub struct ControlLoop {
    // Backend Observed state prefix "placement/"
    keyspace: Keyspace,
}

impl ControlLoop {
    pub fn tick(&mut self, universe: &mut Universe) -> Result<TickReport, Error> {
        // 1. Monitor: Read current placement/ paths from observed store
        let observed = self.read_observed();
        
        // 2. Analyze: Ask the Classifier to compute parentage from current properties
        let desired = Classifier::classify_all(&universe.table)?;
        
        // 3. Plan: Diff desired vs. observed to find structural drift
        let moves = Drift::diff(&desired, &observed);
        
        // 4. Execute: Reparent drifted subtrees and commit ReHomed events to the ledger
        self.apply_moves(universe, moves)
    }
}
```

---

## 4. Execution Phases

| Phase | Milestone | Crates Involved |
| :--- | :--- | :--- |
| **Phase 1** | Implement `SeedParser` & `Resolver` AST layers. | `modules/ir` |
| **Phase 2** | Implement `Normalizer` (rule optimizer) & `Planner`. | `modules/ir` |
| **Phase 3** | Implement dynamic properties ancestry delegation & `Classifier`. | `modules/meta`, `modules/lattice` |
| **Phase 4** | Build the autonomic `ControlLoop` reconciling structural moves. | `modules/reconcile`, `modules/boot` |
| **Phase 5** | Expose seed loading, classification APIs, and Python client methods. | `modules/gate`, `qornl-sdks/python-client` |
