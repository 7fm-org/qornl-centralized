# Qornl: Concept Subsumption & The Dynamic Lattice
Version: 1.0.0
Classification: Core Mathematical Logic

---

## 1. The Dynamic Concept Lattice

In Qornl, a node's position in the inheritance tree is **never static**. Position is computed dynamically from the node's current attributes, creating a self-rearranging classification hierarchy.

```
       [ Node State Changes ] (Property added/removed)
                 |
                 v
       +--------------------+
       |  Predicate check   | (Fickle Object Reclassification)
       +--------------------+
                 |
       +----------v----------+
       | Subsumption Engine  | (Incremental OWL Classification)
       +---------------------+
                 |
                 v
       [ Path Re-routed ] (Ancestry is updated in milliseconds)
```

---

## 2. Mathematical Foundations

### 2.1 Formal Concept Analysis (FCA)
FCA proves that any collection of objects and attributes induces exactly one correct concept lattice:

$$\text{Concept} := (A, B) \quad \text{where} \quad A' = B \quad \text{and} \quad B' = A$$

Qornl leverages this mathematical bedrock. Your hierarchy is a computed object that converges dynamically as properties shift during execution, guaranteeing structural correctness.

### 2.2 Subsumption Hierarchy
A subsumption hierarchy calculates relationships based on set subsets and supersets:
* A parent node is simply the broader set containing its children's attributes.
* Computing where a concept belongs in this lattice is managed by a classification reasoner, eliminating manual parent declarations.

---

## 3. Runtime Relocation Logic

When an object's characteristics change (a property is added, removed, or modified):

1. **Predicate Classes**: The class membership is evaluated as a predicate over the object's current state. The object satisfies the class criteria, therefore it becomes that class.
2. **Dynamic Object Reclassification**: If the state change violates its current parent, the subsumption engine calculates the new parentage.
3. **Subtree Relocation**: The node dynamically updates its ancestry pointers. Because lookups delegate live, its entire subtree is re-routed back to the root instantly without breaking links.
4. **Incremental Classification**: To meet millisecond performance constraints, the system recomputes only the affected region of the concept lattice, ignoring unchanged modules.
