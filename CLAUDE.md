# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RookieDB is a bare-bones relational DBMS implementation for UC Berkeley's CS186 course. Students complete 5 projects: B+ tree indices (Proj1), query operators/optimization (Proj2), concurrency control (Proj3), database recovery (Proj4), and integration (Proj5).

## Build and Test Commands

```bash
# Compile
mvn clean compile

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=TestBPlusTree

# Run a single test method
mvn test -Dtest=TestBPlusTree#testSimpleBulkLoad

# Run tests for a specific project
mvn test -Dgroups=edu.berkeley.cs186.database.categories.Proj1Tests

# Run only public tests
mvn test -Ppublic

# Run only student tests
mvn test -Pstudent
```

Project categories: `Proj0Tests` through `Proj5Tests`, plus `PublicTests`, `HiddenTests`, `StudentTests`.

## Architecture

### Storage Layer (bottom-up)

**`io/`** — `DiskSpaceManager` manages raw file pages, assigning virtual page numbers.

**`memory/`** — `BufferManager` caches pages in `BufferFrame` objects. Pages must be pinned before use and unpinned after. Eviction policies (LRU, Clock) are in this package.

**`table/`** — `Table` implements a heap file on top of `PageDirectory`. Each `Record` is a list of `DataBox` values; rows are identified by `RecordId(pageNum, slotNum)`. `Schema` describes column names and `Type`s.

**`databox/`** — Custom type system. `DataBox` is the abstract value class; `Type` describes the type (int, float, bool, string, byte array). Do not confuse with Java types — always use `DataBox` subclasses for values.

### Index Layer

**`index/`** — `BPlusTree` stores `(DataBox key → RecordId)` mappings. `InnerNode` and `LeafNode` both extend `BPlusNode`. Nodes serialize/deserialize themselves from a single `Page`. Leaf nodes are linked for sequential scans.

### Query Layer

**`query/`** — Volcano-model iterator pipeline. `QueryOperator` is the base class; each operator wraps a source operator and yields `Record`s via `hasNext()`/`next()`. `QueryPlan` builds the operator tree from a `Transaction`. Join algorithms are in `query/join/`.

### Transaction and Concurrency

**`Transaction` / `TransactionContext`** — `Transaction` is the public API; `TransactionContext` is the internal interface used by storage components. Status lifecycle: `RUNNING → COMMITTING/ABORTING → COMPLETE`.

**`concurrency/`** — Multi-granularity locking. `LockManager` is the central lock table. `LockContext` wraps a named resource in a hierarchy (database → table → page). Lock types: IS, IX, S, SIX, X. `LockUtil` has convenience escalation helpers.

### Recovery

**`recovery/`** — ARIES recovery protocol. `ARIESRecoveryManager` maintains a dirty page table and transaction table, writes log records via `LogManager`, and performs analysis/redo/undo on restart.

### Entry Points

- `Database.java` — top-level facade; holds references to all subsystems.
- `cli/` — SQL parser (javacc-generated) and command-line REPL; not part of student projects.

## Student Work Conventions

- Stubs marked `// TODO(proj#_part#)` indicate where to implement.
- `DummyLockManager` and `DummyRecoveryManager` are no-ops used in earlier projects before those systems are implemented.
- Tests use JUnit 4 (`@Test`, `@Before`, `@After`); categories are marker interfaces in `categories/`.
