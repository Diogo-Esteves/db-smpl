# imlab — a self-study main-memory compiling database

A bottom-up, hand-then-generated implementation of a tiny main-memory database
system in C++20: hand-coded OLTP storage and transactions, a hand-coded analytical
query, and then compilers that generate that same storage/query code and compile
queries to native code at runtime instead of interpreting a plan tree.

This project reconstructs the curriculum of TUM's IMLAB course (Prof. Thomas
Neumann's group — HyPer, Umbra). See [`references/README.md`](references/README.md)
for the full curriculum, papers, and background this implementation follows week by
week; this file covers the project itself.

## Status

Scaffolding only — CMake/C++20 build with GoogleTest and sanitizers wired up and
verified working; no storage/transaction code yet. Check `imlab/docs/weekNN.md` for
what's actually been built as weeks land.

## Repository layout

```text
imlab/
  CMakeLists.txt
  data/              # TPC-C data files (gitignored, generate locally — see below)
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
  docs/              # weekNN.md notes: what was built, benchmark numbers, design decisions
```

## Requirements

- Linux or WSL2 (not native Windows) — runtime compilation needs `dlopen`/`dlsym`,
  profiling needs `perf`.
- A recent GCC or Clang with C++20 support, CMake ≥ 3.20, GoogleTest.
- The [TPC-C specification](https://www.tpc.org/tpcc/) (free) for schema/transaction
  reference, and a generated TPC-C dataset (e.g. 5 warehouses) as CSV/TBL files under
  `imlab/data/` (gitignored).

## Build & test

```sh
cmake -S imlab -B imlab/build
cmake --build imlab/build
./imlab/build/imlab_tests
```

Debug builds (the default) compile with `-fsanitize=address,undefined`. Use
`-DCMAKE_BUILD_TYPE=Release` for benchmark runs — sanitizers materially affect timing.

```sh
cmake -S imlab -B imlab/build-release -DCMAKE_BUILD_TYPE=Release
cmake --build imlab/build-release
```

## Working conventions

- Language: **C++20**, built with CMake; tests with GoogleTest; target Linux/WSL2.
- Work **one week at a time**, in order (see the curriculum in
  [`references/README.md`](references/README.md)). Each week must build and pass its
  tests before moving on.
- No external DB libraries or parser generators. Small helper libraries (formatting,
  CLI parsing, testing, benchmarking) are fine.
- Prefer simple, measurable implementations first, then optimize with profiling
  (`perf`) evidence.
- Every performance claim needs a reproducible benchmark command recorded in
  `imlab/docs/`.
- Explain design trade-offs in the weekly notes (`imlab/docs/weekNN.md`) — the goal is
  learning, not just working code.
