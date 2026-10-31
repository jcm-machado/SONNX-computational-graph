# SONNX Computational Graph — Formal Specification and Verification

This repository accompanies a dissertation on the **formal specification and verification of the SONNX computational graph**. It contains the Why3/WhyML developments associated with the dissertation, including the specification of graph entities, execution semantics, and refinement between abstract and concrete representations.

The work constitutes **a focused fragment of the ongoing SONNX proposal** to establish a safety-related profile of ONNX for use in safety-critical systems. In particular, the repository addresses the computational graph and its formal properties; it is **not** intended to represent the complete SONNX specification or the full set of ONNX operators.

## Repository organization

```text
.
├── tensor/
│   ├── libtensor.mlw
│   ├── libvector.mlw
│   ├── std.mlw
│   ├── layout.mlw
│   └── tensor.mlw
└── graph/
    ├── add.mlw
    ├── matmul.mlw
    ├── sigmoid.mlw
    ├── cRefinement.mlw
    ├── entities.mlw
    ├── centities.mlw
    ├── cGraphExec.mlw
    ├── inductive.mlw
    ├── lemmas.mle
    ├── memory.mlw
    ├── sgraph.mlw
    └── utils.mlw
```

### `graph/` — Computational graph formalisation

This directory contains the **core WhyML development of the dissertation**. The files are organised around an abstract specification of the computational graph, a concrete C-compatible representation, and the relationship between their execution semantics.

**Abstract specification and verification**

| File            | Purpose                                                                                                                                                                                                                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `entities.mlw`  | Defines the abstract entities: supported tensor values, tensor identifiers, graph state, the operator type (`Add`, `Matmul`, `Sigmoid`), and computation nodes with input/output arity invariants.                                                                                                            |
| `utils.mlw`     | Provides auxiliary functions for manipulating nodes, tensor identifiers, and graph state, alongside graph-theoretic definitions for edges, reachability, and cycle detection.                                                                                                                                 |
| `sgraph.mlw`    | Specifies the abstract graph and its **structural invariants**, including uniqueness, input/output closure, acyclicity, and reachability. It also defines operator dispatch, node executability, state updates, and the **nondeterministic transition relation** for individual and multiple execution steps. |
| `lemmas.mlw`    | Develops supporting execution lemmas and the proofs of **confluence** and **uniqueness of reachable final configurations**, using results such as commutativity of independent executions, the diamond property, and the strip lemma.                                                                         |
| `inductive.mlw` | Formalises initial graph configurations and auxiliary/main **inductive invariants**, with preservation and reachability properties used to reason about executions.                                                                                                                                           |

**Concrete representation and refinement**

| File              | Purpose                                                                                                                                                                                                                                                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `centities.mlw`   | Defines concrete tensor values (with computation-status flags), computation nodes, tensor identifiers, and graph state, using pointers and fixed-width integers.                                                                                                                                                         |
| `cGraph.mlw`      | Defines the concrete graph record that holds the concrete state, the computation-node array, and its size.                                                                                                                                                                                                               |
| `memory.mlw`      | Provides pointer-based access and update operations for tensor values and computation nodes, together with logical views of memory regions and supporting lemmas.                                                                                                                                                        |
| `cRefinement.mlw` | Defines the **refinement mapping** from concrete tensor values, state, and computation nodes to their abstract counterparts. It also specifies concrete graph validity conditions and constructs the corresponding abstract graph.                                                                                       |
| `cGraphExec.mlw`  | Implements **concrete node and graph execution**, including executability checks and the traversal of computation nodes. Its contracts relate successful concrete node executions to abstract transitions, preserve graph conditions, and establish properties of the resulting execution trace and final configuration. |

**Operator definitions**

The three operator files provide both abstract, mathematical definitions and concrete implementations, together with graph-level interfaces and contracts relating the two representations:

- `add.mlw` — Element-wise addition.
- `matmul.mlw` — Matrix multiplication, including its dot-product definition.
- `sigmoid.mlw` — Element-wise sigmoid.

Taken together, these developments provide an abstract view of graph structure and execution, reasoning about execution-order independence and reachable final configurations, and a concrete representation intended to support subsequent C code extraction. The repository focuses on this **graph-level fragment of SONNX**, rather than the complete operator catalogue or the full SONNX proposal.

### `tensor/` — Supporting tensor library

This directory provides the tensor and vector abstractions, operations, and layout-related definitions required by the graph specification. **The development of the tensor library itself falls outside the scope of the dissertation**; it is included as a necessary dependency so that the graph formalisation can be inspected and its proof obligations can be reproduced.

## Verification environment

The developments use the following tool versions:

| Tool     | Version    |
| -------- | ---------- |
| Why3     | **1.8.2**  |
| Alt-Ergo | **2.4.3**  |
| CVC5     | **1.2.1**  |
| Z3       | **4.13.4** |

## Running proofs

The verification conditions are discharged interactively through the **Why3 IDE**. The commands below assume that the Why3 and prover versions listed above are installed and configured.

### Graph files

Navigate to the `graph/` directory:

```bash
cd graph
```

To open **any graph specification file** in the Why3 IDE, use:

```bash
why3 ide -L ./ -L ../tensor/tensor <file_name>.mlw
```

For example:

```bash
why3 ide -L ./ -L ../tensor/tensor sgraph.mlw
why3 ide -L ./ -L ../tensor/tensor lemmas.mlw
why3 ide -L ./ -L ../tensor/tensor inductive.mlw
why3 ide -L ./ -L ../tensor/tensor cGraphExec.mlw
why3 ide -L ./ -L ../tensor/tensor cRefinement.mlw
```

The library paths are necessary to resolve the dependencies between WhyML modules:

- `-L ./` makes modules in **`graph/`** available, including dependencies on other graph specification files.
- `-L ../tensor/tensor` makes the supporting modules in **`tensor/tensor`** available to the graph development.

Once the file is open in the IDE, select the configured prover (Alt-Ergo, CVC5, or Z3) to attempt its verification conditions.

### Tensor library files

To inspect the supporting tensor library in the Why3 IDE, navigate to `tensor/` from the repository root:

```bash
cd tensor
why3 ide -L ./ tensor.mlw
```

Replace `tensor.mlw` with another file in the same directory (for example, `libtensor.mlw`, `libvector.mlw`, `std.mlw`, or `layout.mlw`) as needed. Here, `-L ./` resolves dependencies on other modules in the tensor library.
