# Self-Study: Main-Memory Compiling Database System

A self-study reconstruction of TUM's **Practical Course: Database Implementation** (IMLAB, WS 26/27, Prof. Thomas Neumann's group, the people behind HyPer and Umbra).

- Course page: <https://db.in.tum.de/teaching/ws2627/imlab/?lang=en>
- Kickoff slides: <https://db.in.tum.de/teaching/ws2627/imlab/kickoff.pdf>

> **Note:** The official assignments and code templates live on TUM's internal GitLab and are not public. The weekly tasks below are a reconstruction based on the public course description, the kickoff slides, and the papers the group builds on. They are not the official handouts.

---

## 1. What the course is about

The goal is to build a tiny main-memory database system from scratch, with a focus on correctness and speed.

It uses a bottom-up approach. Rather than optimizing an existing system like PostgreSQL, which keeps you stuck in its old architecture, you:

1. **Hand-code** the core of TPC-C (an OLTP benchmark) in C++, focusing on how to store and index data efficiently.
2. **Hand-code** analytical queries over that storage format.
3. **Automate**: build compilers that *generate* the storage code and the query code you previously wrote by hand. Queries are compiled into tight loops (no iterator/Volcano model), using C++ as the intermediate language, compiled at runtime.

The official topics are:

- Data representation in memory and query execution
- Building a shell for the DBMS
- Lexing and parsing SQL
- Algebra trees and query optimization (predicate pushdown only)
- Code generation and execution
- Multithreading

The official structure is:

- ~7 weeks of weekly programming tasks, each building on the previous one
- ~7 weeks of an individual project (reproduce a paper, build a fancy feature, or a research topic)
- A final talk of 10–15 minutes presenting the project
- Official templates exist in C++ and Rust. Pick one early, since tasks build on each other.

**Language for this study: C++20.** Rust is a valid alternative.

---

## 2. Environment setup (Week 0)

- [ ] Use **Linux or WSL2**, not native Windows. Runtime compilation needs `dlopen`/`dlsym`, and profiling needs `perf`.
- [ ] Install a recent GCC or Clang, CMake, and GoogleTest; optionally Google Benchmark.
- [ ] Create the repo skeleton with sanitizers (ASan/UBSan) enabled in debug builds.
- [ ] Download the **TPC-C specification** (free at tpc.org) and read clauses 1–2: the schema and the NewOrder, Payment, and Delivery transactions.
- [ ] Get or generate TPC-C data (e.g., 5 warehouses) as CSV/TBL files.
- [ ] Read Hellerstein, Stonebraker & Hamilton, *Architecture of a Database System* (free PDF) for the big-picture map.

Suggested repo layout:

```
imlab/
  CMakeLists.txt
  data/              # TPC-C data files (gitignored)
  include/imlab/     # headers
  src/
    storage/         # tables, indexes
    tpcc/            # hand-written transactions (phase 1)
    schemac/         # schema compiler (phase 2)
    sql/             # lexer, parser
    algebra/         # operators, codegen
    runtime/         # compile + dlopen
    shell/           # REPL
  tools/             # benchmark drivers
  test/
  docs/              # notes, measurements, paper summaries
```

---

## 3. Phase 1: Hand-coded database (Weeks 1–3)

### Week 1: Storage and hand-written OLTP

**Build**

- [ ] C++ structs for all 9 TPC-C tables.
- [ ] A loader for the TPC-C data files.
- [ ] A primary-key index per table. Start with `std::unordered_map`, then write your own hash table.
- [ ] Hand-coded **NewOrder** transaction.
- [ ] A benchmark: run NewOrder 1,000,000 times with random parameters and report transactions/second.
- [ ] Experiment: **row store** (vector of structs) vs **column store** (struct of vectors) for the same table.

**Read:** Harizopoulos et al., *OLTP Through the Looking Glass, and What We Found There* (SIGMOD 2008).

**Done when:** NewOrder is correct (verified with tests), throughput is measured, and the row-vs-column results are written up in `docs/`.

### Week 2: More transactions, ordered indexes, deletes

**Build**

- [ ] Hand-coded **Delivery** (and optionally Payment).
- [ ] Delivery needs ordered access (the oldest new-order per district), so implement an ordered index: a **B+-tree** or an **Adaptive Radix Tree (ART)**.
- [ ] Delete support without excessive fragmentation (e.g., swap-with-last plus index update).
- [ ] Benchmark a mixed workload (e.g., 90% NewOrder / 10% Delivery).

**Read:** Leis, Kemper & Neumann, *The Adaptive Radix Tree: ARTful Indexing for Main-Memory Databases* (ICDE 2013).

### Week 3: Hand-written analytical query

**Build**

- [ ] Hand-code an analytical join over `customer ⋈ order ⋈ orderline` with a filter and aggregation, written as tight loops: build a hash table on the smaller side, probe with the larger, then aggregate.
- [ ] Run it while OLTP transactions execute, and document what breaks (consistency, interference). This is the hybrid OLTP/OLAP problem.

**Read:** Kemper & Neumann, *HyPer: A Hybrid OLTP&OLAP Main Memory Database System Based on Virtual Memory Snapshots* (ICDE 2011).

---

## 4. Phase 2: Generating what you hand-wrote (Weeks 4–7)

### Week 4: Schema compiler and shell

**Build**

- [ ] Hand-written **lexer + recursive-descent parser** for `CREATE TABLE` (types: integer, numeric(p,s), char(n), varchar(n), timestamp; primary keys; optional index declarations).
- [ ] A **code generator** that emits the C++ structs, loaders, and index code you wrote by hand in Week 1.
- [ ] Check: the generated code compiles and passes the Week 1–2 tests.
- [ ] A simple **REPL shell**.

### Week 5: Algebra and produce/consume codegen (the core)

**Build**

- [ ] Operator tree: `TableScan`, `Selection`, `HashJoin`, `Aggregation` (optional), `Print`.
- [ ] Neumann's **produce/consume** model: each operator *emits C++ source* instead of interpreting tuples.
- [ ] Check: the generated code for the Week 3 query looks almost identical to your hand-written version, and it's equally fast.

**Read, twice:** Neumann, *Efficiently Compiling Efficient Query Plans for Modern Hardware* (VLDB 2011).
**Contrast with:** Graefe, *Volcano: An Extensible and Parallel Query Evaluation System* (the iterator model you are avoiding).

### Week 6: SQL to algebra, and runtime compilation

**Build**

- [ ] Parse a `SELECT ... FROM ... WHERE ...` subset (conjunctive predicates, equi-joins).
- [ ] Build the algebra tree with **predicate pushdown**: turn equality predicates between tables into hash joins, and push single-table predicates to scans. This is the only optimization required.
- [ ] **Runtime compilation**: write the generated C++ to a file, invoke the compiler to build a `.so`, `dlopen` it, `dlsym` the entry point, and run it.
- [ ] Measure **compile time vs execution time**. Expect compilation to dominate for small queries; this motivates later projects.

### Week 7: Multithreading

**Build**

- [ ] Parallel scans and hash-join builds using **morsel-driven parallelism**: tables are split into small chunks (morsels) that worker threads pull from a shared dispatcher.
- [ ] Compare a partitioned hash table against a shared one.
- [ ] Plot speedup vs thread count.

**Read:** Leis, Boncz, Kemper & Neumann, *Morsel-Driven Parallelism: A NUMA-Aware Query Evaluation Framework for the Many-Core Age* (SIGMOD 2014).

---

## 5. Phase 3: Individual project (~7 weeks)

Pick one, and finish with a 10–15 minute talk (slides plus benchmarks).

| Project | Key reference |
| --- | --- |
| Vectorized engine alongside the compiled one; compare them | Kersten et al., *Everything You Always Wanted to Know About Compiled and Vectorized Queries But Were Afraid to Ask* (VLDB 2018); Boncz et al., *MonetDB/X100* (CIDR 2005) |
| Faster compilation: direct LLVM IR, or an adaptive interpret-then-compile path | Neumann & Freitag, *Umbra: A Disk-Based System with In-Memory Performance* (CIDR 2020); Kohn, Leis & Neumann, *Adaptive Execution of Compiled Queries* (ICDE 2018) |
| MVCC for real concurrent OLTP | Neumann, Mühlbauer & Kemper, *Fast Serializable Multi-Version Concurrency Control for Main-Memory Database Systems* (SIGMOD 2015) |
| Join ordering (DPccp) evaluated on the JOB benchmark | Moerkotte & Neumann, *Analysis of Two Existing and One New Dynamic Programming Algorithm for the Generation of Optimal Bushy Join Trees without Cross Products* (VLDB 2006); Leis et al., *How Good Are Query Optimizers, Really?* (VLDB 2015) |

---

## 6. Supporting material

- **CMU 15-445/645** (Andy Pavlo): database fundamentals, with free lecture videos on YouTube.
- **CMU 15-721** (Advanced Database Systems): the closest public match to this lab, covering compilation, vectorization, and scheduling. Slides and videos are online.
- **Alex Petrov, *Database Internals*** (O'Reilly): storage and indexing.
- **TUM IN2118**, *Database Systems on Modern CPU Architectures*: the recommended lecture for IMLAB; check for publicly available slides.

---

## 7. Pace and tracking

- A realistic pace alongside a full-time job is **1.5–2 calendar weeks per course week**, or **about 4–5 months** in total.
- For each week, keep in `docs/weekNN.md`:
  - what was built
  - benchmark numbers (hardware, data size, throughput/latency)
  - design decisions and alternatives rejected
  - a short summary of the paper read
- Keep a running `docs/measurements.md` so progress across weeks is visible.

---

## 8. Instructions for a coding agent working on this repo

- Language: **C++20**, built with CMake; tests with GoogleTest; target Linux/WSL2.
- Work **one week at a time**, in order. Each week must build and pass its tests before moving on.
- Don't use external DB libraries or parser generators. Small helper libraries (formatting, CLI parsing, testing, benchmarking) are fine.
- Prefer simple, measurable implementations first, then optimize with profiling (`perf`) evidence.
- Every performance claim needs a reproducible benchmark command recorded in `docs/`.
- Explain design trade-offs in the weekly notes. The goal is learning, not just working code.
