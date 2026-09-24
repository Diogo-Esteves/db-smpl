# Reference material (not tracked in git)

This folder holds copyrighted PDFs (papers, *Database Internals*, Pavlo slides, etc.)
for personal study. Contents are gitignored — only this file and the structure survive.

Layout (files are symlinks into `~/Documents/books/db`, so nothing is duplicated on disk):

```
references/
  books/           # Database Internals, DDIA, DB System Concepts, Postgres internals, etc.
  papers/          # Architecture of a DB System, the "Red Book" (Readings in DB Systems)
  pavlo-15445/     # CMU 15-445 slide deck (relational model through indexes, so far)
  pavlo-15721/     # CMU 15-721 (Advanced DB Systems) — not yet fetched, see below
  tum-imlab/       # kickoff slides, course description snapshots — not yet fetched
```

## Currently in `books/`
- `database-internals-petrov.pdf` — Petrov, *Database Internals* (core reference for storage/indexing weeks)
- `database-system-concepts-{6e,7e}.pdf` — Silberschatz, Korth & Sudarshan
- `designing-data-intensive-applications.pdf` — Kleppmann
- `duckdb-in-action.pdf`
- `postgresql-internals-14.pdf`
- `postgresql-up-and-running.pdf`
- `pdm-public.pdf` — unidentified title (LaTeX, 506pp, covers relational model/queries/formal languages) — rename once identified
- `database-relational-theory.mobi`

(Left out of `books/` on purpose: the beginner SQL/MongoDB/MySQL/PostgreSQL "notes for professionals" PDFs in the source folder — not relevant to this course.)

## Currently in `papers/`
- `hellerstein-stonebraker-hamilton-2007-architecture-of-a-database-system.pdf` — the exact Week 0 reading
- `readings-in-database-systems-5th-edition-redbook.pdf` — the "Red Book", useful survey/paper collection for later weeks and the individual project

## Currently in `pavlo-15445/`
Slides 01–09 (relational model → hash tables → indexes 1/2). Good parallel reading for Weeks 1–2
(storage, hashing, B+-trees/ART).

## Still needed (not found locally — fetch when relevant)
- Leis, Kemper & Neumann — *The Adaptive Radix Tree* (ICDE 2013) — Week 2
- Harizopoulos et al. — *OLTP Through the Looking Glass* (SIGMOD 2008) — Week 1
- Kemper & Neumann — *HyPer* (ICDE 2011) — Week 3
- Neumann — *Efficiently Compiling Efficient Query Plans for Modern Hardware* (VLDB 2011) — Week 5
- Graefe — *Volcano* — Week 5 (contrast)
- Leis, Boncz, Kemper & Neumann — *Morsel-Driven Parallelism* (SIGMOD 2014) — Week 7
- Individual-project papers (Kersten et al., MonetDB/X100, Umbra, Adaptive Execution, MVCC SIGMOD 2015, DPccp, "How Good Are Query Optimizers, Really?")
- CMU 15-721 slides
- TUM IMLAB kickoff slides / course page snapshot

These are mostly paywalled VLDB/SIGMOD/ICDE papers — only add local copies here from a source
you're personally entitled to (institutional access, ACM/IEEE membership, or an author's own
postprint page). Otherwise link to them from the weekly docs instead of storing a copy.

Papers behind a paywall (VLDB/SIGMOD/ICDE proceedings) should only be added here from a
copy you're personally entitled to access (e.g. via institutional access, ACM/IEEE
membership, or author postprints on the paper's own page).
