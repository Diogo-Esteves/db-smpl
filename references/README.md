# Course background, curriculum & reading material

This folder holds the *why* behind the project's design: the course this project
reconstructs, its full weekly curriculum, and an inventory of local study material
(copyrighted PDFs, symlinked in — not tracked in git). See the top-level `README.md`
for the project itself (what's implemented, how to build/run it).

## What this reconstructs

A self-study reconstruction of TUM's **Practical Course: Database Implementation**
(IMLAB, WS 26/27, Prof. Thomas Neumann's group — the people behind HyPer and Umbra).

- Course page: <https://db.in.tum.de/teaching/ws2627/imlab/?lang=en>
- Kickoff slides: <https://db.in.tum.de/teaching/ws2627/imlab/kickoff.pdf>

> **Note:** The official assignments and code templates live on TUM's internal GitLab
> and are not public. The curriculum below is a reconstruction based on the public
> course description, the kickoff slides, and the papers the group builds on. It is
> not the official handout.

## What the course is about

The goal is to build a tiny main-memory database system from scratch, with a focus on
correctness and speed.

It uses a bottom-up approach. Rather than optimizing an existing system like
PostgreSQL, which keeps you stuck in its old architecture, you:

1. **Hand-code** the core of TPC-C (an OLTP benchmark) in C++, focusing on how to
   store and index data efficiently.
2. **Hand-code** analytical queries over that storage format.
3. **Automate**: build compilers that *generate* the storage code and the query code
   you previously wrote by hand. Queries are compiled into tight loops (no
   iterator/Volcano model), using C++ as the intermediate language, compiled at
   runtime.

Official topics: data representation in memory and query execution; building a shell
for the DBMS; lexing and parsing SQL; algebra trees and query optimization (predicate
pushdown only); code generation and execution; multithreading.

Official structure: ~7 weeks of weekly programming tasks (each building on the
previous one), ~7 weeks of an individual project (reproduce a paper, build a fancy
feature, or a research topic), a final talk of 10–15 minutes presenting the project.
Official templates exist in C++ and Rust — this reconstruction uses **C++20**.

## Curriculum

### Phase 1: Hand-coded database (Weeks 1–3)

**Week 1 — Storage and hand-written OLTP**
- C++ structs for all 9 TPC-C tables.
- A loader for the TPC-C data files.
- A primary-key index per table. Start with `std::unordered_map`, then write your own
  hash table.
- Hand-coded **NewOrder** transaction.
- Benchmark: run NewOrder 1,000,000 times with random parameters, report tx/sec.
- Experiment: **row store** (vector of structs) vs **column store** (struct of
  vectors) for the same table.
- **Read:** Harizopoulos et al., *OLTP Through the Looking Glass, and What We Found
  There* (SIGMOD 2008).
- **Done when:** NewOrder is correct (tests), throughput is measured, row-vs-column
  results are written up.

**Week 2 — More transactions, ordered indexes, deletes**
- Hand-coded **Delivery** (and optionally Payment).
- Delivery needs ordered access (the oldest new-order per district) → implement an
  ordered index: a **B+-tree** or an **Adaptive Radix Tree (ART)**.
- Delete support without excessive fragmentation (e.g., swap-with-last plus index
  update).
- Benchmark a mixed workload (e.g., 90% NewOrder / 10% Delivery).
- **Read:** Leis, Kemper & Neumann, *The Adaptive Radix Tree: ARTful Indexing for
  Main-Memory Databases* (ICDE 2013).

**Week 3 — Hand-written analytical query**
- Hand-code an analytical join over `customer ⋈ order ⋈ orderline` with a filter and
  aggregation, written as tight loops: build a hash table on the smaller side, probe
  with the larger, then aggregate.
- Run it while OLTP transactions execute, and document what breaks (consistency,
  interference) — the hybrid OLTP/OLAP problem.
- **Read:** Kemper & Neumann, *HyPer: A Hybrid OLTP&OLAP Main Memory Database System
  Based on Virtual Memory Snapshots* (ICDE 2011).

### Phase 2: Generating what you hand-wrote (Weeks 4–7)

**Week 4 — Schema compiler and shell**
- Hand-written **lexer + recursive-descent parser** for `CREATE TABLE` (types:
  integer, numeric(p,s), char(n), varchar(n), timestamp; primary keys; optional index
  declarations).
- A **code generator** that emits the C++ structs, loaders, and index code you wrote
  by hand in Week 1.
- Check: the generated code compiles and passes the Week 1–2 tests.
- A simple **REPL shell**.

**Week 5 — Algebra and produce/consume codegen (the core)**
- Operator tree: `TableScan`, `Selection`, `HashJoin`, `Aggregation` (optional),
  `Print`.
- Neumann's **produce/consume** model: each operator *emits C++ source* instead of
  interpreting tuples.
- Check: the generated code for the Week 3 query looks almost identical to your
  hand-written version, and it's equally fast.
- **Read, twice:** Neumann, *Efficiently Compiling Efficient Query Plans for Modern
  Hardware* (VLDB 2011).
- **Contrast with:** Graefe, *Volcano: An Extensible and Parallel Query Evaluation
  System* (the iterator model you are avoiding).

**Week 6 — SQL to algebra, and runtime compilation**
- Parse a `SELECT ... FROM ... WHERE ...` subset (conjunctive predicates, equi-joins).
- Build the algebra tree with **predicate pushdown**: turn equality predicates
  between tables into hash joins, and push single-table predicates to scans. The only
  optimization required.
- **Runtime compilation**: write the generated C++ to a file, invoke the compiler to
  build a `.so`, `dlopen` it, `dlsym` the entry point, and run it.
- Measure **compile time vs execution time**. Expect compilation to dominate for
  small queries; this motivates later projects.

**Week 7 — Multithreading**
- Parallel scans and hash-join builds using **morsel-driven parallelism**: tables are
  split into small chunks (morsels) that worker threads pull from a shared
  dispatcher.
- Compare a partitioned hash table against a shared one.
- Plot speedup vs thread count.
- **Read:** Leis, Boncz, Kemper & Neumann, *Morsel-Driven Parallelism: A NUMA-Aware
  Query Evaluation Framework for the Many-Core Age* (SIGMOD 2014).

### Phase 3: Individual project (~7 weeks)

Pick one, and finish with a 10–15 minute talk (slides plus benchmarks).

| Project | Key reference |
| --- | --- |
| Vectorized engine alongside the compiled one; compare them | Kersten et al., *Everything You Always Wanted to Know About Compiled and Vectorized Queries But Were Afraid to Ask* (VLDB 2018); Boncz et al., *MonetDB/X100* (CIDR 2005) |
| Faster compilation: direct LLVM IR, or an adaptive interpret-then-compile path | Neumann & Freitag, *Umbra: A Disk-Based System with In-Memory Performance* (CIDR 2020); Kohn, Leis & Neumann, *Adaptive Execution of Compiled Queries* (ICDE 2018) |
| MVCC for real concurrent OLTP | Neumann, Mühlbauer & Kemper, *Fast Serializable Multi-Version Concurrency Control for Main-Memory Database Systems* (SIGMOD 2015) |
| Join ordering (DPccp) evaluated on the JOB benchmark | Moerkotte & Neumann, *Analysis of Two Existing and One New Dynamic Programming Algorithm for the Generation of Optimal Bushy Join Trees without Cross Products* (VLDB 2006); Leis et al., *How Good Are Query Optimizers, Really?* (VLDB 2015) |

### Supporting material

- **CMU 15-445/645** (Andy Pavlo): database fundamentals, free lecture videos on
  YouTube.
- **CMU 15-721** (Advanced Database Systems): the closest public match to this lab,
  covering compilation, vectorization, and scheduling.
- **Alex Petrov, *Database Internals*** (O'Reilly): storage and indexing.
- **TUM IN2118**, *Database Systems on Modern CPU Architectures*: the recommended
  lecture for IMLAB; check for publicly available slides.

### Pace

A realistic pace alongside a full-time job is **1.5–2 calendar weeks per course
week**, or **about 4–5 months** in total.

---

## Local material inventory

Files below are symlinks into `~/Documents/books/db`, so nothing is duplicated on
disk. This whole `references/` folder is gitignored except this file.

```
references/
  books/           # Database Internals, DDIA, DB System Concepts, Postgres internals, etc.
  papers/          # Architecture of a DB System, the "Red Book" (Readings in DB Systems)
  pavlo-15445/     # CMU 15-445 slide deck (relational model through indexes, so far)
  pavlo-15721/     # CMU 15-721 (Advanced DB Systems) — not yet fetched, see below
  tum-imlab/       # kickoff slides, course description snapshots — not yet fetched
```

### Currently in `books/`
- `database-internals-petrov.pdf` — Petrov, *Database Internals* (core reference for
  storage/indexing weeks)
- `database-system-concepts-{6e,7e}.pdf` — Silberschatz, Korth & Sudarshan
- `designing-data-intensive-applications.pdf` — Kleppmann
- `duckdb-in-action.pdf`
- `postgresql-internals-14.pdf`
- `postgresql-up-and-running.pdf`
- `pdm-public.pdf` — unidentified title (LaTeX, 506pp, covers relational
  model/queries/formal languages) — rename once identified
- `database-relational-theory.mobi`

(Left out of `books/` on purpose: the beginner SQL/MongoDB/MySQL/PostgreSQL "notes
for professionals" PDFs in the source folder — not relevant to this course.)

### Currently in `papers/`
- `hellerstein-stonebraker-hamilton-2007-architecture-of-a-database-system.pdf` — the
  exact Week 0 reading
- `readings-in-database-systems-5th-edition-redbook.pdf` — the "Red Book", useful
  survey/paper collection for later weeks and the individual project

### Currently in `pavlo-15445/`
Slides 01–09 (relational model → hash tables → indexes 1/2). Good parallel reading
for Weeks 1–2 (storage, hashing, B+-trees/ART).

### Still needed (not found locally — fetch when relevant)
- Leis, Kemper & Neumann — *The Adaptive Radix Tree* (ICDE 2013) — Week 2
- Harizopoulos et al. — *OLTP Through the Looking Glass* (SIGMOD 2008) — Week 1
- Kemper & Neumann — *HyPer* (ICDE 2011) — Week 3
- Neumann — *Efficiently Compiling Efficient Query Plans for Modern Hardware* (VLDB
  2011) — Week 5
- Graefe — *Volcano* — Week 5 (contrast)
- Leis, Boncz, Kemper & Neumann — *Morsel-Driven Parallelism* (SIGMOD 2014) — Week 7
- Individual-project papers (Kersten et al., MonetDB/X100, Umbra, Adaptive Execution,
  MVCC SIGMOD 2015, DPccp, "How Good Are Query Optimizers, Really?")
- CMU 15-721 slides
- TUM IMLAB kickoff slides / course page snapshot

These are mostly paywalled VLDB/SIGMOD/ICDE papers — only add local copies here from a
source you're personally entitled to (institutional access, ACM/IEEE membership, or
an author's own postprint page). Otherwise link to them from the weekly docs instead
of storing a copy.
