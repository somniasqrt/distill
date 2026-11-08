# distill

A symbolic regression engine in Rust, based on genetic programming.

> **Status:** work in progress. The core data structures are in place; the evolutionary loop is not implemented yet. The API will change.

## Motivation

Most regression methods fit parameters to a model you choose in advance. Symbolic regression searches for the model itself: given measurements of a system, it looks for a closed-form expression such as `y = 2.5·sin(x) + x²` that explains them.

The project is inspired by Schmidt & Lipson, [*Distilling Free-Form Natural Laws from Experimental Data*](https://doi.org/10.1126/science.1165893) (Science, 2009), where analytical laws were recovered from raw experimental data with no prior knowledge of the underlying physics. `distill` is a personal project to implement this approach from scratch: small, dependency-light and easy to read.

## How it works

Candidate solutions are expression trees:

```
        +
       / \
      ^   x        ⇔   (2 ^ x) + x
     / \
    2   x
```

The engine evolves a population of such trees:

1. **Initialize** a population of random trees (Full / Grow).
2. **Evaluate** each tree on the dataset and compute its fitness (currently MSE).
3. **Select** parents, favoring lower error.
4. **Recombine** by swapping random subtrees between parents (crossover) and **mutate** by replacing random subtrees.
5. **Repeat** until the target fitness or the generation limit is reached.

A tree's size (node count) is tracked as its complexity, so the search can prefer simpler expressions among equally accurate ones.

Steps 1–2 are implemented; 3–5 are in progress.

### Repository layout

```
crates/
  distill-core/     core library: AST, dataset, config, engine
  distill-studio/   placeholder for a future frontend
web/                placeholder for a future web interface
```

## Getting started

Requires Rust 1.85+ (edition 2024).

```sh
git clone https://github.com/somniasqrt/distill.git
cd distill
cargo build
cargo test
cargo run -p distill-core --example demo
```

The library is not published on crates.io. To use it from another project, add a git dependency:

```toml
[dependencies]
distill-core = { git = "https://github.com/somniasqrt/distill" }
```

Minimal example (current API):

```rust
use distill_core::{config::Config, dataset::Dataset, engine::Engine};

let dataset = Dataset::new(
    vec!["x".into()],
    vec![vec![1.0], vec![2.0], vec![3.0]],
    "y".into(),
    vec![2.0, 4.0, 6.0],
)?;

let config = Config { seed: Some(42), ..Default::default() };
let engine = Engine::new(config, dataset)?;
// engine.run() is not implemented yet
```

## Features

- [x] Expression trees with variables, constants, unary (`neg`, `abs`, `sin`, `cos`, `tan`, `cot`, `exp`, `ln`) and binary (`+`, `-`, `*`, `/`, `^`) operators
- [x] Tree evaluation and human-readable `Display`
- [x] Random tree generation: *Full* and *Grow* methods with configurable max depth
- [x] Subtree access by index (get, get mut, replace) as a basis for crossover and mutation
- [x] Protected operators: division by zero, `ln(0)`, overflowing `exp`/`pow` etc. return finite values instead of `NaN`/`inf`
- [x] Multivariable input
- [x] Validated `Dataset` with error reporting and table-style `Display`
- [x] MSE fitness evaluation, node count as complexity
- [x] Validated `Config` with defaults and seeded, reproducible RNG
- [ ] Ramped half-and-half population initialization
- [ ] Tournament selection and elitism
- [ ] Subtree crossover and mutation
- [ ] Main evolution loop with stopping criteria
- [ ] Other fitness metrics (MAE, log error, R²); the config options already exist
- [ ] Parallel fitness evaluation with `rayon`
- [ ] Pareto front over accuracy and complexity
- [ ] Expression simplification
- [ ] CSV input
- [ ] `distill-studio`: interactive frontend (separate future project)

## References

- M. Schmidt, H. Lipson. *Distilling Free-Form Natural Laws from Experimental Data.* Science 324(5923), 81–85, 2009. [doi:10.1126/science.1165893](https://doi.org/10.1126/science.1165893)
- J. R. Koza. *Genetic Programming: On the Programming of Computers by Means of Natural Selection.* MIT Press, 1992.
