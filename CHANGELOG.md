# Changelog

All notable changes to the Atlas library are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
