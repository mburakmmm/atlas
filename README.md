# Atlas 🧭

**Atlas** is a lightweight, pure-Nox tabular data analysis and statistics library designed to make data cleaning, transformation, and analytical aggregation fast, predictable, and memory-safe.

Inspired by Pandas but crafted natively for the **Nox** static type system, Atlas avoids bloated dependencies and delivers native $C$-like performance on medium-sized datasets.

[![Nox Compiler](https://img.shields.io/badge/noxc-2.0.86-blue.svg)](https://github.com/mburakmmm/nox-lang)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Tests](https://img.shields.io/badge/tests-12%20passed-success.svg)](tests/)

---

## Features

- 📑 **RFC 4180 CSV I/O**: Robust parsing of commas, quotes, and newlines via `nox.csv` with automatic schema inference.
- 🔍 **Declarative Filtering**: Type-safe expression trees (`Expr`) with comparisons, string matches, set membership, and logical operators (`AND`/`OR`/`NOT`).
- 📊 **Summary Statistics**: Mean, median, sample & population variance/stddev implemented via numerically stable **Welford's algorithm**.
- 🗂️ **Grouping & Aggregations**: Deterministic multi-key group-by with count, sum, mean, median, min, and max aggregations.
- 🔀 **Stable Multi-key Sorting**: Guaranteed $O(N \log N)$ stable merge sort with per-key ascending/descending flags and null positioning.
- 🧹 **Missing Value Management**: Explicit `Cell(TYPE_NULL)` tracking, inspection (`count_missing`), replacement (`fill_missing`), and row dropping (`drop_missing`).
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

All 12 test suites run and verify:
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
- Overwrite safety and filesystem I/O (`tests/io_test.nox`)
- End-to-end integration and regression pipelines (`tests/regression_test.nox`)

---

## Security

- **Formula Injection Mitigation**: Pass `safe_spreadsheet=True` to `to_csv()` or `write_csv_file()` to escape prefixes (`=`, `+`, `-`, `@`).
- **File Overwrite Protection**: `write_csv_file()` and `write_json_file()` require explicit `overwrite=True` to overwrite existing files.

---

## License

MIT License. See [LICENSE](LICENSE) for details.
