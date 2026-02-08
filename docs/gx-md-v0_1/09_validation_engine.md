# 09_validation_engine.md

## Table of Contents
- [1. Validation Inputs](#1-validation-inputs)
- [2. Rule Engine Structure](#2-rule-engine-structure)
- [3. Required Relationship Checks](#3-required-relationship-checks)
- [4. Cardinality and Target Checks](#4-cardinality-and-target-checks)
- [5. Why this matters](#5-why-this-matters)

## 1. Validation Inputs
Validation requires:

- immutable `&Ontology`
- immutable `&Graph`
- mutable diagnostics sink (`&mut Vec<Diagnostic>`)

This borrowing pattern avoids copying large structures while keeping side effects explicit.

## 2. Rule Engine Structure
Model rules as independent functions for composability.

```rust
pub fn validate_all(onto: &Ontology, graph: &Graph) -> Vec<Diagnostic> {
    let mut diags = Vec::new();
    rule_unknown_types(onto, graph, &mut diags);
    rule_duplicate_ids(graph, &mut diags);
    rule_invalid_targets(onto, graph, &mut diags);
    rule_required_relationships(onto, graph, &mut diags);
    rule_cardinality(onto, graph, &mut diags);
    diags
}
```

## 3. Required Relationship Checks
For each node type, verify required outgoing relations exist.

```rust
fn rule_required_relationships(onto: &Ontology, graph: &Graph, out: &mut Vec<Diagnostic>) {
    for node in graph.nodes.values() {
        if let Some(tdef) = onto.types.get(&node.type_name) {
            for rel in tdef.allowed_outgoing.iter().filter(|r| r.required) {
                let found = graph.edges.iter().any(|e| e.source == node.id && e.rel == rel.rel);
                if !found {
                    out.push(Diagnostic::error(
                        node.source_path.clone(),
                        format!("node {} missing required relation {}", node.id, rel.rel),
                    ));
                }
            }
        }
    }
}
```

## 4. Cardinality and Target Checks
Cardinality check counts matching edges per `(node, relation)` pair. Target check validates both target existence and type compatibility.

Alternative design: perform validation during parse/build. Rejected because it couples phases and reduces rule testability. A dedicated rule engine is clearer.

## 5. Why this matters
The validator is the type-checker core of gx.md. A rule-oriented architecture makes semantics explicit, testable, and extensible.
