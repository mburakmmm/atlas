# Atlas 🧭

**Atlas** is a lightweight, pure-Nox tabular data analysis and statistics library designed to make data cleaning, transformation, and analytical aggregation fast, predictable, and memory-safe.

Inspired by Pandas but crafted natively for the **Nox** static type system, Atlas avoids bloated dependencies and delivers native $C$-like performance on medium-sized datasets.

[![Nox Compiler](https://img.shields.io/badge/noxc-2.0.86-blue.svg)](https://github.com/mburakmmm/nox-lang)
[![Version](https://img.shields.io/badge/version-v0.9.5-orange.svg)](nox.json)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-39%20passed-success.svg)](tests/)

---

## Features

- 🔢 **Full 64-bit Checked Arithmetic Suite (`values.nox`, `atlas.nox`)**: Centralized `checked_add_int64`, `checked_sub_int64`, `checked_mul_int64`, and `checked_div_int64` with $\text{INT64\_MAX}$ and $\text{INT64\_MIN}$ bounds protection, division-by-zero checks, and two's complement boundary overflow prevention (`INT64_MIN // -1`), raising actionable `ShapeError` across all summation and arithmetic pathways (`groupby.agg_sum`, `StreamingCsvReader.sum`, `StreamingGroupedTable.aggregate`, `table.sum_val`, `stats.stat_sum_int`, `pivot_table`, `cumulative_sum`).
- 🧮 **Neumaier Compensated Summation (`stats.nox`)**: High-precision compensated summation in `stat_sum` and `stat_mean` eliminating catastrophic cancellation ($10^{16} + 1.0 - 10^{16} = 1.0$) and preserving floating-point precision across disparate numerical magnitudes.
- 🎯 **Ground-Truth Analytical Reference Verification**: Validated cell-for-cell against Pandas 2.x, Polars 1.x, and standard SQL for multi-key GroupBy aggregations, window ranking with tie skipping, pivot tables, melt transformations, and full outer joins.
- ⚡ **Cross-Backend Determinism Matrix**: Comprehensive tests covering 64-bit integer limits, IEEE 754 float/NaN/Inf semantics, composite hashing across types, stable sorting, multi-byte UTF-8 encoding (Turkish, CJK, emojis), and exception hierarchies.
- 🪟 **Window Functions & WindowContext Pipeline (`window.nox`)**: Complete analytical window functions (`row_number`, `rank`, `dense_rank`, `lag`, `lead`, `cumulative_sum`, `moving_average`) with non-destructive row ordering. High-performance `WindowContext` pipeline partitions and sorts once with lazy column buffering and single-pass row materialization (**~2.9x median speedup** across 10 iterations on 100k rows, with 100% cell-by-cell differential equivalence). Option A independent pipeline contract with informative `SchemaError` reporting. Strict order key validation (`SortError`) and multi-column tie evaluation.
- 🔄 **Table Reshaping (`reshape.nox`)**: Wide-to-long format transformation (`melt`), long-to-wide format transformation (`pivot`), and aggregate reshaping (`pivot_table` with mean, sum, count, min, max, first, last).
- 🔗 **Advanced Relational Joins (`join.nox`)**: Full Outer Join (`join_full`) preserving unmatched rows on both sides with schema nullability guarantees, and Cartesian product Cross Join (`join_cross`).
- 🌊 **Streaming Transform & Online GroupBy Pipeline**: Fluent, low-memory query engine (`open_csv_stream.filter(...).select([...]).group_by([...]).aggregate(...)`) processing multi-million row datasets in strictly bounded memory ($O(G \times K + B + R \times S)$).
- 📑 **RFC 4180 CSV, TSV & Custom Delimiters**: Robust parsing of commas, tabs, semicolons, and multi-character delimiters (`||`, `::`) with streaming state machines and automatic schema inference.
- ⚡ **O(N + M) Hash Join**: Blazing fast inner and left-outer joins (`join_inner`, `join_left`) with hash indexing and null key isolation.
- ⚡ **O(N) Hash GroupBy**: High-cardinality aggregation engine indexing tens of thousands of unique groups in single-digit milliseconds.
- 🗄️ **Relational SQL & Feather Bridge**: Generate SQLite/PostgreSQL DDL schemas (`table_to_create_table_sql`), safe quoted identifiers (`quote_identifier`), and parameterized INSERT statements (`table_to_insert_sql`).
- ✅ **Tabular Data Validation**: Configurable `TableValidator` (email, range, length, non-empty, allowed values, explicit types) with strict numeric validation.
- 🔗 **Relational Joins**: Inner and left-outer joins (`join_inner`, `join_left`) with collision suffixes and null key isolation.
- 🧱 **Table Concatenation**: Vertical row concatenation (`concat_rows`) with schema alignment, and horizontal concatenation (`concat_cols`).
- 🔍 **Declarative Filtering**: Type-safe expression trees (`Expr`) with comparisons, string matches, set membership, and logical operators (`AND`/`OR`/`NOT`).
- 📈 **Quantiles & Outlier Detection**: Continuous linear interpolation quantiles (`table.quantile`), IQR, and Tukey's fences outlier filtering (`table.filter_outliers_iqr`).
- 📊 **Summary Statistics**: Mean, median, sample & population variance/stddev implemented via numerically stable **Welford's algorithm**.
- 🗂️ **Grouping & Aggregations**: Deterministic multi-key group-by with count, sum, mean, median, min, and max aggregations.
- 🔀 **Stable Multi-key Sorting**: Guaranteed $O(N \log N)$ stable merge sort with per-key ascending/descending flags and null positioning.
- 🧹 **Missing Value Management**: Explicit `Cell(TYPE_NULL)` tracking, inspection (`count_missing`), replacement (`fill_missing`), and row dropping (`drop_missing`).
- 🖥️ **Pretty Grid Formatter**: Clean ASCII terminal grid display (`table.print_grid()`, `table.to_grid_string()`) with auto-width adjustment and truncation.
- 🔄 **JSON ↔ CSV Conversion**: Direct tabular bidirectional round-trips for JSON array-of-objects representations via `nox.json` and `JsonWriter`.
- 🛡️ **Security Built-in**: Protection against spreadsheet formula injection (`=`, `+`, `-`, `@`) and accidental file overwrites.

---

## Installation

Add Atlas to your project's `nox.json`:

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

Then run:
```bash
noxc fetch
```

---

## Quick Example

```nox
import atlas
from atlas import Table, read_csv, col_gt_int, group_by, GroupedTable, agg_count, agg_mean

# 1. Parse CSV string
csv_data: str = "name,department,salary\n" + \
"Alice,Engineering,110000.0\n" + \
"Bob,Engineering,85000.0\n" + \
"Charlie,Marketing,72000.0\n" + \
"Diana,Marketing,68000.0\n"

t: Table = read_csv(csv_data, True)

# 2. Filter high earners (> 80k)
seniors: Table = t.filter(col_gt_int("salary", 80000))

# 3. Group by department and compute average salary
grouped: GroupedTable = group_by(t, ["department"])
summary: Table = grouped.aggregate([
    agg_count("headcount"),
    agg_mean("salary", "avg_salary")
])

summary.print_preview()

# Low-Memory Streaming Pipeline (v0.8.0)
stream_result: Table = open_csv_stream("data/sales.csv") \
    .filter(col_gt_float("salary", 50000.0)) \
    .select(["department", "salary"]) \
    .group_by(["department"]) \
    .aggregate([
        agg_sum("salary", "total_payroll"),
        agg_mean("salary", "avg_salary")
    ])

stream_result.print_preview()
```

---

## Detailed Capabilities

### 1. CSV Reading and Schema Inference

```nox
import atlas
from atlas import Table, read_csv, read_csv_file

# In-memory CSV parsing
t: Table = read_csv(csv_text, has_header=True)

# Direct file reading
t_file: Table = read_csv_file("data/sales.csv", has_header=True)
```

Atlas automatically scans non-empty values across each column and infers the appropriate type:
- `TYPE_INT` (integers)
- `TYPE_FLOAT` (floating-point numbers)
- `TYPE_BOOL` (true/false)
- `TYPE_STR` (strings)

### 2. Column Selection, Renaming, and Dropping

```nox
# Select specific columns in desired order
t_sub: Table = t.select(["name", "salary"])

# Drop columns
t_dropped: Table = t.drop(["internal_id"])

# Rename column
t_renamed: Table = t.rename("salary", "annual_compensation")
```

### 3. Filtering Rows

```nox
from atlas import col_gt_int, col_eq_str, col_contains, expr_and, expr_or

# Single condition
tech_only: Table = t.filter(col_eq_str("department", "Engineering"))

# Compound condition (Dept == "Engineering" AND salary > 90,000)
lead_eng: Table = t.filter(expr_and(
    col_eq_str("department", "Engineering"),
    col_gt_int("salary", 90000)
))
```

### 4. Missing Data Handling

```nox
from atlas import cell_float

# Check missing count
missing_salaries: int = t.count_missing("salary")

# Fill missing values
t_filled: Table = t.fill_missing("salary", cell_float(50000.0))

# Drop rows where critical columns have missing data
t_clean: Table = t.drop_missing(["salary", "department"], how="any")
```

### 5. Summary Statistics

```nox
# Direct column statistics
avg_salary: float = t.mean("salary")
med_salary: float = t.median("salary")
std_salary: float = t.std("salary", is_sample=True)
min_sal: Cell = t.min_val("salary")
max_sal: Cell = t.max_val("salary")
```

### 6. GroupBy & Multi-Metric Aggregations

```nox
from atlas import group_by, GroupedTable, agg_count, agg_mean, agg_median, agg_std

grouped: GroupedTable = group_by(t, ["region", "department"])
report: Table = grouped.aggregate([
    agg_count("total_employees"),
    agg_mean("salary", "avg_salary"),
    agg_median("salary", "median_salary"),
    agg_std("salary", "salary_dispersion")
])
```

### 7. JSON ↔ CSV Conversion

```nox
from atlas import table_to_json, table_from_json, csv_to_json, json_to_csv

# Convert Table to JSON array of objects
json_str: str = table_to_json(t)

# Direct string conversions
json_output: str = csv_to_json(raw_csv_text)
csv_output: str = json_to_csv(raw_json_text)
```

---

## Performance Benchmark

Measured on 10,000 synthetic records with `noxc 2.0.86`:

| Task | Execution Time |
| :--- | :--- |
| Generate 10,000 rows | **2 ms** |
| Filter 10,000 rows (predicate) | **1 ms** |
| Mean, Median, StdDev (Welford) | **2 ms** |
| GroupBy 4 categories + 4 aggregations | **3 ms** |
| Multi-key Stable Merge Sort | **11 ms** |

Run benchmarks yourself:
```bash
noxc run benchmarks/benchmark_table.nox
```

---

## Running Tests

Atlas includes a full automated test suite:

```bash
noxc test
```

All 14 test suites run and verify:
- Typed cell conversions and comparisons (`tests/values_test.nox`)
- Schema uniqueness and projection invariants (`tests/schema_test.nox`)
- Table shape checks and bounds (`tests/table_test.nox`)
- RFC 4180 quoting, ragged row checks, formula injection (`tests/csv_test.nox`)
- JSON array-of-objects round-trips (`tests/json_test.nox`)
- Declarative filter evaluation (`tests/filter_test.nox`)
- Stable multi-key merge sort (`tests/sort_test.nox`)
- Welford's algorithm and numerical stability (`tests/stats_test.nox`)
- Multi-column group-by engine (`tests/groupby_test.nox`)
- Missing value cleaning and policies (`tests/missing_test.nox`)
- Relational inner and left joins with collision suffixes (`tests/join_test.nox`)
- Vertical and horizontal table concatenation (`tests/concat_test.nox`)
- Overwrite safety and filesystem I/O (`tests/io_test.nox`)
- End-to-end integration and regression pipelines (`tests/regression_test.nox`)

---

## Security

- **Formula Injection Mitigation**: Pass `safe_spreadsheet=True` to `to_csv()` or `write_csv_file()` to escape prefixes (`=`, `+`, `-`, `@`).
- **File Overwrite Protection**: `write_csv_file()` and `write_json_file()` require explicit `overwrite=True` to overwrite existing files.

---

## License

MIT License. See [LICENSE](LICENSE) for details.
