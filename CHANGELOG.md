# Changelog

All notable changes to the Atlas library are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.1] - 2026-11-01

### Fixed & Improved
- **Hash GroupBy Collision Bucket Management (`groupby.nox`)**:
  - Transitioned `group_map` from single index mapping to collision bucket indexing (`dict[str, list[int]]`).
  - Groups sharing the same hash signature are preserved in candidate lists and differentiated via exact `equals()` checks; new groups update the bucket, guaranteeing identical rows never split across multiple groups.
- **Mixed 64-bit Integer / Float Numeric Precision (`values.nox`)**:
  - Implemented exact mixed numeric equality and ordering checks in `Cell.equals()` and `Cell.compare()`.
  - Non-whole floats and numbers above $2^{53}$ are compared without false equality or IEEE-754 precision degradation.
- **Length-Prefixed Hash Key Serialization (`join.nox`, `groupby.nox`)**:
  - Length-prefixed encoding (`"S" + len + ":" + s` and `[len]part`) prevents cross-column hash signature collisions when string values contain separator tokens.
- **SQL Dialect Validation & Primary Key Verification (`sql_adapter.nox`)**:
  - `quote_identifier`, `type_to_sql_type`, `table_to_create_table_sql`, and `table_to_insert_sql` validate dialects and raise `SchemaError` on unsupported inputs.
  - `table_to_create_table_sql` validates `primary_key_col` existence in table schema and raises `ColumnNotFoundError` on missing columns.
- **Strict RFC 4180 CSV Mode (`delimited.nox`, `io.nox`)**:
  - Added `strict: bool = False` mode to `parse_delimited`, `read_delimited`, `read_tsv`, and `read_delimited_file`.
  - In strict mode, unescaped quotes in unquoted fields or trailing characters after closing quotes raise `CsvImportError`.
- **CI Test Reliability**:
  - Decoupled wall-clock assertions (`assert duration < X`) from unit tests (`tests/hash_join_test.nox`, `tests/hash_groupby_test.nox`) into dedicated benchmark suites to prevent flaky CI failures under variable server loads.
- **Test Suite Expansion**:
  - Added `tests/groupby_collision_test.nox`, `tests/strict_csv_test.nox`, and `tests/sql_validation_test.nox`. Total test suites: 25 (100% passing).

## [0.6.0] - 2026-11-01

### Added
- **O(N + M) Hash Join Engine (`join.nox`)**:
  - Replaced nested-loop join with high-performance Hash Join index build and probe phases.
  - Type-aware normalized hash key generator preserving integer-float numeric equivalence and null key isolation.
- **O(N) Hash GroupBy Engine (`groupby.nox`)**:
  - Replaced $O(G)$ linear group scans with $O(1)$ dictionary hash table lookup.
  - Successfully groups 10,000 unique groups in 3 ms.
- **SQL Identifier Quoting & PostgreSQL Dialect (`sql_adapter.nox`)**:
  - Added `quote_identifier(name, dialect)` preventing SQL Identifier Injection across table and column names.
  - Added PostgreSQL `$1, $2, ...` positional placeholder generation to `table_to_insert_sql`.
- **64-bit Integer Precision & Data Integrity (`values.nox`)**:
  - Fixed `as_float()` to compute `float(self.i)` dynamically instead of duplicating float storage.
  - Direct 64-bit integer equality and comparisons in `equals()` and `compare()` preventing IEEE-754 double precision loss above $2^{53}$.
- **Strict Validation & Explicit Type Rules (`validation.nox`)**:
  - `rule_range`, `rule_min`, `rule_max` now strictly validate numeric types and flag non-numeric strings as `ValidationError`.
  - Added `rule_type(col_name, expected_type)` for explicit column data type enforcement.
- **CSV / Delimited Engine Hardening (`delimited.nox`)**:
  - Constrained quoted field entry strictly to field boundaries (`len(cur_field) == 0`).
  - Added substring multi-character delimiter detection in `dump_delimited` ensuring round-trip data safety for custom delimiters like `||`.
- **Benchmarks & Tests**:
  - Added `benchmarks/benchmark_hash.nox` (high cardinality and hash join verification).
  - Added `tests/hash_join_test.nox`, `tests/hash_groupby_test.nox`, `tests/values_precision_test.nox` bringing total test suites to 22.

## [0.5.0] - 2026-11-01

### Added
- **API Consolidation & Large-Scale Validation**:
  - Full API reference consolidated and verified in `docs/api.md` covering all 16 modules.
  - Large-scale 100,000 record benchmark suite (`benchmarks/benchmark_large.nox`): 100k rows ingested in 11 ms, filtered in 7 ms, mean computed in 4 ms, and 5-category groupby in 18 ms.
  - Enhanced CLI interactive demo in `main.nox` with pretty grid formatting and comprehensive summary steps.
  - 19 automated test suites passing with 100% success rate under `noxc test`.
  - Zero mandatory C/C++ or foreign framework runtime dependencies.

## [0.4.0] - 2026-11-01

### Added
- **Relational SQL & Feather Database Adapter (`sql_adapter.nox`)**:
  - `table_to_create_table_sql(table, table_name, dialect, primary_key_col)` generating clean DDL for SQLite and PostgreSQL.
  - `table_to_insert_sql(table, table_name)` generating injection-safe parameterized INSERT queries with `?` markers.
  - `table_from_records(col_names, col_types, rows)` building tables directly from database result rowsets.
  - `cell_from_db_value(kind, i, f, s, b, is_null)` providing Feather and generic DB value conversion.
- **Noxen-Compatible Tabular Validation Engine (`validation.nox`)**:
  - `TableValidator` configurable with column-specific rules.
  - Rules: `rule_non_empty`, `rule_range`, `rule_min`, `rule_max`, `rule_length`, `rule_allowed_values`, and `rule_email`.
  - Detailed `ValidationReport` with row/column error indexing and formatted summaries.
  - Automatic row partitioning: `validator.filter_valid(table)` and `validator.filter_invalid(table)`.
- **Tests**: `tests/sql_adapter_test.nox` and `tests/validation_test.nox` bringing test suite to 19 suites passing.
- **Documentation & Examples**: `docs/integrations.md` and `examples/sql_and_validation.nox`.

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
