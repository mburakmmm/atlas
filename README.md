# Atlas 🧭

**Pure-Nox Tabular Data Analysis, Reshaping & High-Performance Streaming Engine**

Atlas is a lightweight, zero-dependency, statically typed tabular processing library crafted natively for the **[Nox programming language](https://noxlang.com)**. Inspired by the ergonomics of Pandas and Polars, Atlas delivers $C$-like analytical performance, rigorous 64-bit checked arithmetic, memory-bounded streaming pipelines, and guaranteed numerical stability across macOS, Linux, and Windows.

---

[![CI Matrix](https://github.com/mburakmmm/atlas/actions/workflows/ci.yml/badge.svg)](https://github.com/mburakmmm/atlas/actions/workflows/ci.yml)
[![Release Gate](https://github.com/mburakmmm/atlas/actions/workflows/release.yml/badge.svg)](https://github.com/mburakmmm/atlas/actions/workflows/release.yml)
[![Version](https://img.shields.io/badge/version-v0.9.5-orange.svg)](nox.json)
[![Platform Support](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-blue.svg)](#-cross-platform--compatibility-matrix)
[![Tests](https://img.shields.io/badge/tests-39%20passed%20%28100%25%29-brightgreen.svg)](tests/)
[![Compiler](https://img.shields.io/badge/noxc-2.0.86-purple.svg)](https://github.com/mburakmmm/nox-lang)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

---

## 📑 Table of Contents

- [Why Atlas?](#-why-atlas)
- [Architecture Overview](#-architecture-overview)
- [Key Features](#-key-features)
- [Installation](#-installation)
- [Quickstart Guide](#-quickstart-guide)
  - [1. Reading CSV & Inspecting Schema](#1-reading-csv--inspecting-schema)
  - [2. Filtering & Column Projection](#2-filtering--column-projection)
  - [3. Multi-Metric Hash GroupBy](#3-multi-metric-hash-groupby)
  - [4. High-Performance Window Pipeline](#4-high-performance-window-pipeline)
  - [5. Reshaping: Melt & Pivot Table](#5-reshaping-melt--pivot-table)
  - [6. Relational Joins & Full Outer Join](#6-relational-joins--full-outer-join)
  - [7. Memory-Bounded CSV Streaming](#7-memory-bounded-csv-streaming)
- [Performance Benchmarks](#-performance-benchmarks)
- [Cross-Platform & Compatibility Matrix](#-cross-platform--compatibility-matrix)
- [Documentation Index](#-documentation-index)
- [Automated Test Suite](#-automated-test-suite)
- [Contributing & Community](#-contributing--community)
- [Security & License](#-security--license)

---

## 💡 Why Atlas?

- **100% Pure Nox**: Zero external C/C++ dependencies or brittle ABI bindings. Compiles cleanly with standard `noxc`.
- **Deterministic & Numerically Sound**: Prevents floating-point catastrophic cancellation using **Neumaier compensated summation** ($[10^{16}, 1.0, -10^{16}] = 1.0$) and sample variance via **Welford's algorithm**.
- **Checked 64-bit Integer Arithmetic**: Built-in `checked_*_int64` suite across summation, grouping, and windows protects against silent hardware integer overflow, raising actionable `ShapeError` exceptions.
- **Single-Sort Window Pipeline**: `WindowContext` clusters partitioning and sorting once, evaluating operations across lazy column buffers for a **~2.9x median speedup** on 100,000 rows.
- **Strictly Bounded Memory Streaming**: Process multi-gigabyte CSVs with $O(B + R \times S + L)$ heap utilization via non-allocating state machines.
- **Cross-Platform Native**: Tested and verified across Ubuntu Linux, macOS (Apple Silicon & Intel), and Windows.

---

## 🏛️ Architecture Overview

```text
                             User Application / CLI
                                        │
                                 Atlas Public API
               (atlas.table, atlas.window, atlas.streaming, atlas.stats)
                                        │
      ┌─────────────────────┬───────────┴───────────┬─────────────────────┐
      ▼                     ▼                       ▼                     ▼
 WindowContext          GroupedTable            Transformations      Streaming Engine
┌──────────────┐     ┌──────────────┐        ┌───────────────────┐  ┌──────────────────┐
│ single sort  │     │ O(N) Hash    │        │ filter, select    │  │ persistent RFC   │
│ lazy buffers │     │ partitioning │        │ melt, pivot_table │  │ state machine    │
│ to_table()   │     │ Welford aggs │        │ full outer join   │  │ bounded memory   │
└──────────────┘     └──────────────┘        └───────────────────┘  └──────────────────┘
      │                     │                       │                     │
      └─────────────────────┼───────────────────────┴─────────────────────┘
                            ▼
               Core Data Model (values & schema)
        ┌────────────────────────────────────────────────────────┐
        │ Cell (int64, float64, string, bool, null, NaN, ±Inf)   │
        │ Schema (ColumnDef, checked 64-bit arithmetic, typing)   │
        └────────────────────────────────────────────────────────┘
                            │
               Adapters & Serialization Engine
        ┌────────────────────────────────────────────────────────┐
        │ nox.csv (RFC 4180) │ nox.json (bidirectional) │ nox.fs │
        └────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

| Capability | Module | Description |
| :--- | :--- | :--- |
| **Checked Arithmetic** | [`values.nox`](values.nox) | Centralized `checked_add_int64`, `checked_sub_int64`, `checked_mul_int64`, `checked_div_int64` with $\text{INT64\_MAX}$ and $\text{INT64\_MIN}$ bounds checking. |
| **Compensated Statistics** | [`stats.nox`](stats.nox) | Neumaier compensated summation for `stat_sum`/`stat_mean`; online Welford algorithm for sample/population variance and standard deviation. |
| **Window Pipeline** | [`window.nox`](window.nox) | `WindowContext` fluent pipeline (`with_row_number`, `with_rank`, `with_dense_rank`, `with_lag`, `with_lead`, `with_cumulative_sum`, `with_moving_average`) with single-pass sort and lazy buffering. |
| **Table Reshaping** | [`reshape.nox`](reshape.nox) | Wide-to-long format transformation (`melt`), long-to-wide format pivoting (`pivot`), and aggregate reshaping (`pivot_table`). |
| **Relational Joins** | [`join.nox`](join.nox) | High-speed hash joins (`join_inner`, `join_left`), SQL-compliant Full Outer Join (`join_full`), and Cartesian Product Cross Join (`join_cross`). |
| **Streaming Reader** | [`streaming.nox`](streaming.nox) | State machine streaming parser (`StreamingCsvReader`) supporting chunked reads, online filtering, projections, and online GroupBy in bounded RAM. |
| **Delimiters & TSV** | [`delimited.nox`](delimited.nox) | RFC 4180 parsing for commas, tabs (`\t`), semicolons (`;`), and arbitrary multi-character delimiters (`\|\|`, `::`). |
| **Declarative Filtering** | [`expr.nox`](expr.nox) | Type-safe AST expressions (`col_eq`, `col_gt`, `col_in`, `expr_and`, `expr_or`, `expr_not`). |
| **SQL & Validation** | [`sql_adapter.nox`](sql_adapter.nox) | SQLite/PostgreSQL DDL schema generation, identifier escaping, parameterized INSERT generation, and `TableValidator`. |

---

## 📦 Installation

Add Atlas to your Nox package dependencies in `nox.json`:

```json
{
  "name": "my-data-app",
  "version": "0.1.0",
  "entry": "main.nox",
  "requires": [
    { "name": "atlas", "path": "../atlas" }
  ]
}
```

Then fetch dependencies using the Nox package manager:

```bash
noxc fetch
```

---

## 🚀 Quickstart Guide

### 1. Reading CSV & Inspecting Schema

```nox
import atlas
from atlas import Table, Schema, read_csv, read_csv_file

# Parse CSV text directly with automated type inference
raw_csv: str = "id,name,dept,salary,active\n" + \
               "101,Alice,Eng,115000.5,true\n" + \
               "102,Bob,Marketing,85000.0,true\n" + \
               "103,Charlie,Eng,95000.0,false\n"

t: Table = read_csv(raw_csv, has_header=True)

# Inspect schema and row count
print("Rows: " + str(t.row_count()))       # 3
print("Cols: " + str(t.column_count()))    # 5
t.print_preview()
```

### 2. Filtering & Column Projection

```nox
from atlas import col_gt_float, col_eq_str, expr_and

# Filter active Engineering staff earning > $90k
high_earning_eng: Table = t.filter(expr_and(
    col_eq_str("dept", "Eng"),
    col_gt_float("salary", 90000.0)
)).select(["name", "salary"])

high_earning_eng.print_preview()
```

### 3. Multi-Metric Hash GroupBy

```nox
from atlas import group_by, GroupedTable, agg_count, agg_mean, agg_std, agg_sum

grouped: GroupedTable = group_by(t, ["dept"])
dept_summary: Table = grouped.aggregate([
    agg_count("headcount"),
    agg_mean("salary", "avg_salary"),
    agg_std("salary", "salary_std", is_sample=True),
    agg_sum("salary", "total_payroll")
])

dept_summary.print_preview()
```

### 4. High-Performance Window Pipeline

Atlas provides a fluent `WindowContext` pipeline that partitions and sorts rows **only once**, evaluating all window functions over lightweight memory buffers:

```nox
from atlas import Table, WindowContext, cell_float

# Rank salaries within each department, calculate cumulative sum, and compute 3-period moving average
t_window: Table = t.window(partition_by=["dept"], order_by=["salary"], ascending=True) \
                   .with_row_number("rn") \
                   .with_rank("salary", "salary_rank") \
                   .with_dense_rank("salary", "dense_rank") \
                   .with_cumulative_sum("salary", "cum_salary") \
                   .with_lag("salary", "prev_salary", offset=1, default_val=cell_float(0.0)) \
                   .with_moving_average("salary", window_size=3, output_col="rolling_avg_3") \
                   .to_table()

t_window.print_preview()
```

### 5. Reshaping: Melt & Pivot Table

```nox
from atlas import melt, pivot_table

# Wide to Long (Unpivot)
long_df: Table = t.melt(id_vars=["id", "name"], value_vars=["salary"], var_name="metric", value_name="val")

# Long to Wide with Aggregation (Pivot Table)
pivot_df: Table = t.pivot_table(index_col="dept", columns_col="active", values_col="salary", agg_func="mean")
```

### 6. Relational Joins & Full Outer Join

```nox
from atlas import join_inner, join_left, join_full

# Hash Joins (O(N + M))
inner_res: Table = join_inner(employees_tbl, departments_tbl, on=["dept_id"])
left_res: Table = join_left(employees_tbl, departments_tbl, on=["dept_id"])

# Full Outer Join (preserves unmatched rows on both sides with null padding)
full_res: Table = join_full(employees_tbl, departments_tbl, on=["dept_id"])
```

### 7. Memory-Bounded CSV Streaming

Process multi-million row datasets that exceed available RAM using the streaming pipeline:

```nox
from atlas import open_csv_stream, StreamingCsvReader, col_gt_float, agg_sum, agg_mean

summary_result: Table = open_csv_stream("large_dataset.csv", chunk_rows=2000) \
    .filter(col_gt_float("revenue", 1000.0)) \
    .select(["region", "revenue"]) \
    .group_by(["region"]) \
    .aggregate([
        agg_sum("revenue", "total_rev"),
        agg_mean("revenue", "avg_rev")
    ])

summary_result.print_preview()
```

---

## ⚡ Performance Benchmarks

All benchmarks measured with `noxc 2.0.86` on macOS Darwin (Apple Silicon) and verified for cross-platform parity on Linux and Windows.

### 1. Window Pipeline Benchmark (100,000 Synthetic Rows)
*Source: [`benchmarks/benchmark_window.nox`](benchmarks/benchmark_window.nox)*

| Scenario | Execution Strategy | Sort Operations | Median Latency | Min Latency | Max Latency | Speedup |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| **Control Group** | 4 Standalone Window Calls | $4\times$ | **363 ms** | 351 ms | 398 ms | Baseline (1.0x) |
| **Window Pipeline** | `WindowContext` (Lazy Buffers) | **$1\times$** | **123 ms** | 112 ms | 138 ms | **~2.95x Faster** |

*Differential Verification*: 100% cell-by-cell equivalence verified across all 100,000 rows and 7 columns.

### 2. Partition Cardinality Scaling (50,000 Synthetic Rows)
*Source: [`benchmarks/benchmark_window.nox`](benchmarks/benchmark_window.nox)*

| Cardinality Density ($K$) | Control Group (ms) | Window Pipeline (ms) | Empirical Speedup |
| :--- | :---: | :---: | :---: |
| $K = 10$ partitions | 207 ms | 66 ms | **3.14x** |
| $K = 100$ partitions | 186 ms | 64 ms | **2.91x** |
| $K = 1,000$ partitions | 178 ms | 68 ms | **2.62x** |

### 3. High-Cardinality Hash GroupBy (100,000 Rows, 50,000 Distinct Groups)
*Source: [`tests/high_cardinality_test.nox`](tests/high_cardinality_test.nox)*

- **Ingestion & Hash Grouping**: **37 ms**
- **Collision Buckets & Aggregations**: $O(N)$ linear time complexity.

---

## 🌐 Cross-Platform & Compatibility Matrix

Atlas is continuously tested on every commit and pull request via GitHub Actions across:

| Platform | Architecture | Toolchain / Backend | Status |
| :--- | :--- | :--- | :---: |
| **Ubuntu Linux** | x86_64, aarch64 | `noxc 2.0.86` (LLVM & freestanding math) | ✅ Passing |
| **macOS** | Apple Silicon (arm64), Intel (x86_64) | `noxc 2.0.86` (Darwin Clang/LLVM) | ✅ Passing |
| **Windows 10/11** | x86_64, ARM64 | `noxc 2.0.86` (MSVC / Windows POSIX emulation) | ✅ Passing |

### Cross-Platform Engineering Invariants
- **Line Ending Agnostic**: Accepts both CRLF (`\r\n`) and LF (`\n`) seamlessly; serialization defaults to standard LF.
- **Portable Filesystem Paths**: All temporary files use relative path schemes (`temp_*.csv`), strictly avoiding hardcoded Unix `/tmp/` paths.
- **Resource Cleanup**: Stream file descriptors are wrapped in `try...finally: self.close()` blocks to prevent Windows file locking / sharing violation errors during file deletion.

---

## 📚 Documentation Index

Comprehensive guides and architectural specifications are located in the [`docs/`](docs/) directory:

| Guide | Description |
| :--- | :--- |
| [**API Reference**](docs/api.md) | Exhaustive signatures and syntax examples for all public Atlas classes and functions. |
| [**Architecture**](docs/architecture.md) | Detailed layer architecture, module boundaries, and memory ownership model. |
| [**Capabilities & Audit**](docs/capabilities.md) | Verification audit against the Nox standard library (`nox.csv`, `nox.json`, `nox.fs`, `nox.mathx`). |
| [**Data Model**](docs/data-model.md) | Deep dive into `Cell`, `ColumnDef`, `Schema`, null semantics, and type promotion rules. |
| [**CSV & Streaming**](docs/csv.md) | RFC 4180 parsing, streaming state machines, chunk bounds, and formula injection defense. |
| [**Custom Delimiters**](docs/delimiters.md) | TSV, PSV, and multi-character delimiter parsing and escaping. |
| [**Filtering AST**](docs/filtering.md) | Declarative expression trees, type-safe predicates, and logical compositions. |
| [**GroupBy & Aggregation**](docs/grouping.md) | Grouping semantics, stable ordering, composite hashing, and aggregation functions. |
| [**Numerical Statistics**](docs/statistics.md) | Welford algorithm, Neumaier compensated summation, quantiles, and IQR outlier detection. |
| [**Missing Values**](docs/missing-values.md) | Missing vs null vs NaN semantics, imputation policies, and dropping rules. |
| [**Relational Joins**](docs/joins.md) | Inner, left, full outer, and cross joins, hash collision safety, and null matching rules. |
| [**Performance Guide**](docs/performance.md) | Computational complexity guarantees, memory profiling, and optimization best practices. |
| [**Security Policy**](SECURITY.md) | Vulnerability disclosure, spreadsheet formula injection defense, and overwrite protections. |

---

## 🧪 Automated Test Suite

Atlas maintains a **100% passing automated test suite** across **39 specialized test suites**:

```bash
noxc test
```

### Test Suite Highlights
- **Backend Compatibility Matrix** ([`tests/backend_matrix_test.nox`](tests/backend_matrix_test.nox)): INT64 extreme boundaries, IEEE 754 float/NaN/Inf semantics, hash determinism, stable sort tie preservation, and multi-byte UTF-8 (Turkish, CJK, Emojis).
- **Ground-Truth Reference Verification** ([`tests/reference_comparison_test.nox`](tests/reference_comparison_test.nox)): Verified cell-for-cell against precomputed reference outputs from Pandas 2.x and Polars 1.x.
- **High Cardinality Scaling** ([`tests/high_cardinality_test.nox`](tests/high_cardinality_test.nox)): 50,000 distinct groups on 100,000 rows.
- **Differential Streaming** ([`tests/streaming_differential_test.nox`](tests/streaming_differential_test.nox)): Comparing streaming chunks against in-memory parsers across 10 varied chunk buffer sizes.

---

## 🤝 Contributing & Community

We welcome contributions, bug reports, and analytical feature proposals!

- Please review [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`AGENTS.md`](AGENTS.md) before submitting pull requests.
- All participants are expected to adhere to the [Code of Conduct](CODE_OF_CONDUCT.md).
- To report a bug or suggest a feature, use our [GitHub Issue Templates](.github/ISSUE_TEMPLATE/).

---

## 🛡️ Security & License

- **Security Policy**: See [`SECURITY.md`](SECURITY.md) for vulnerability reporting procedures.
- **License**: Atlas is licensed under the permissive [MIT License](LICENSE). Copyright © 2026 Melih Burak Memiş.
