# Qornl: Virtual IR & Contract Specification
Version: 1.0.0
Classification: Core Schema Specification

---

## 1. Virtual Intermediate Representation (VIR)

The Virtual IR (VIR) acts as the AST (Abstract Syntax Tree) parsing and compilation layer of Qornl. It converts raw JSON envelopes submitted by clients into structured, type-safe in-memory objects.

```
       [ Client JSON ] ---> [ Tokenizer ] ---> [ VIR AST ]
                                                  |
                                       +----------+----------+
                                       |                     |
                                 Metadata Schema      Bindings Resolver
```

### 1.1 Envelope AST Definition
A compiled VIR envelope matches the following record structure:
* **`api_version`**: Must be declared as `qornl/v1`.
* **`kind`**: The type of resource structure (`Flow`, `Worker`, or `Adapter`).
* **`metadata`**:
  * `name`: Unique name identifier.
  * `revision`: Monotonically increasing version counter.
* **`spec`**: The domain-specific properties and execution parameters.
* **`bindings`**: The array of target mapping relationships.

---

## 2. Wire Contracts & Schema Verification

Qornl enforces strict schema verification (Contracts) at the gateway boundary to protect the ledger against malformed entries.

### 2.1 Validation Constraints:
1. **Revision Control**: Updating a flow requires incrementing the `revision` number strictly by 1. Replay or downgrade requests are rejected.
2. **Immutable Names**: The `metadata.name` acts as the persistent entity key. It cannot be mutated after admission.
3. **Property Checking**: Spec parameters must conform to the target shapes defined in the schema. Unmapped parameters are blocked by the gateway validator.

---

## 3. Bindings Compilation Logic

Bindings map runtime execution variables to active task behaviors. The Contract Gateway compiles these mappings into the internal model hierarchy:

```json
"bindings": [
    {
        "source": "worker.behavior",
        "target": "behavior",
        "value": "verify-transaction"
    }
]
```

* **`source`**: The origin fact resolver (`worker.behavior`).
* **`target`**: The variable mapped inside the authority namespace (`behavior`).
* **`value`**: The literal target behavior registered on the task queue.

When a task executes, the gateway resolves the active binding relationships. If a worker attempts to process an unbound behavior, the contract gateway immediately rejects the transaction.
