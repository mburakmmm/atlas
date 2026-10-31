# Changelog

All notable changes to the Atlas library are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.3.0] - 2026-11-01

### Added
- **Custom Delimiters Engine**: Generalized RFC 4180 parsing and serializing state machine supporting arbitrary delimiters (`,`, `;`, `\t`, `|`, etc.) in `delimited.nox`.
- **TSV Support**: Native tab-separated values reading and writing via `read_tsv`, `to_tsv`, `read_tsv_file`, and `write_tsv_file`.
- **Custom Delimited File Operations**: `read_delimited`, `to_delimited`, `read_delimited_file`, and `write_delimited_file`.
- **Quantiles & Percentiles**: `stat_quantile` and `table.quantile(col_name, q)` with continuous linear interpolation (NumPy / R Type-7 compatible).
- **Interquartile Range (IQR)**: `stat_iqr` and `table.iqr(col_name)`.
- **Outlier Detection & Filtering**: `table.filter_outliers_iqr(col_name, multiplier=1.5, keep_missing=True)` using Tukey's fences.
- **Pretty Grid Formatter**: Terminal ASCII grid formatter `table.to_grid_string()` and `table.print_grid()` with column auto-width calculation and cell truncation.
- **Tests**: `tests/tsv_test.nox`, `tests/quantile_test.nox`, and `tests/grid_test.nox` bringing the test suite to 17 suites passing.
- **Documentation & Examples**: `docs/delimiters.md`, `docs/quantiles.md`, and `examples/delimited_and_quantiles.nox`.

## [0.2.0] - 2026-11-01

### Added
- **Relational Joins**: `join_inner` and `join_inner_keys` for inner equi-joins with column collision disambiguation (`_left`, `_right`).
- **Left Outer Joins**: `join_left` and `join_left_keys` preserving all left rows and padding unmatched right fields with `Cell(TYPE_NULL)`.
- **Null Key Isolation**: Missing keys do not join against other missing keys.
- **Vertical Concatenation (`concat_rows`)**: Multi-table row union with automatic column alignment across varying schemas and null-padding.
- **Horizontal Concatenation (`concat_cols`)**: Multi-table column concatenation with strict row count invariant checks and automatic collision resolution.
- **Tests**: `tests/join_test.nox` and `tests/concat_test.nox` bringing the test suite to 14 suites passing.
- **Documentation & Examples**: `docs/joins.md`, `docs/concatenation.md`, and `examples/table_joins.nox`.

## [0.1.0] - 2026-11-01

### Added
- **Core Data Structures**: `Table`, `Schema`, `ColumnDef`, and `Cell` tagged record.
- **CSV Support**: RFC 4180 compliant CSV parser and serializer via `nox.csv` with automatic type inference (`TYPE_INT`, `TYPE_FLOAT`, `TYPE_BOOL`, `TYPE_STR`).
- **Table Operations**: `select`, `drop`, `rename`, `head`, `tail`, `slice`, and `clone`.
- **Filtering**: Type-safe AST-based predicate expressions (`col_eq`, `col_gt`, `col_lt`, `col_contains`, `col_starts_with`, `col_in`, `expr_and`, `expr_or`, `expr_not`).
- **Sorting**: Multi-column stable merge sort ($O(N \log N)$) with per-column ascending/descending flags and null positioning.
- **Statistics**: Numerically stable estimators implementing Welford's algorithm for sample/population variance and standard deviation, arithmetic mean, and odd/even median.
- **Grouping**: Multi-key `group_by` supporting count, sum, mean, median, stddev, min, and max aggregations.
- **Missing Data Handling**: Explicit `Cell(TYPE_NULL)`, `count_missing`, `fill_missing`, and `drop_missing` (with "any" and "all" policies).
- **JSON Support**: Bidirectional conversion between tables and JSON arrays of objects via `nox.json` and `JsonWriter`. Direct `csv_to_json` and `json_to_csv` helpers.
- **Filesystem I/O**: `read_csv_file`, `write_csv_file`, `read_json_file`, and `write_json_file` with overwrite safety.
- **Security**: Formula injection mitigation mode (`safe_spreadsheet=True`) and overwrite prevention.
- **Test Suite**: 12 comprehensive test suites passing under `noxc test`.
- **Documentation**: Full architecture, capabilities, data model, performance, and security documentation.
