# Nox Standard Library Capabilities & Verification Audit

Audit performed against `noxc 2.0.86` on macOS (darwin).

## 1. Verified Stdlib Primitives

### `nox.csv`
- `parse(text: str) -> list[list[str]]`: RFC 4180 parsing supporting commas, quotes, escaped quotes (`""`), and CRLF/LF line endings. Operates strictly in-memory on strings.
- `dump(rows: list[list[str]]) -> str`: Generates standard CSV text with quotes applied where required.
- **Limitation**: Custom delimiters (semicolon, tab) and streaming file reads are out of scope for stdlib. Atlas respects this by providing UTF-8 in-memory text parsing.

### `nox.json`
- `parse(text: str) -> JsonValue`: Full recursive JSON parsing.
- Accessors: `is_null`, `is_bool`, `is_number`, `is_string`, `is_array`, `is_object`, `as_bool`, `as_number`, `as_string`, `array_len`, `array_get`, `object_len`, `object_key`, `object_value`.
- `JsonWriter`: Incremental memory-buffered JSON construction with $O(N)$ concatenation via `nox.strings.join`.

### `nox.mathx`
- Pure math functions: `sqrt`, `pow`, `floor`, `ceil`, `min`, `max`, `abs`. Fully qualified calls (`nox.mathx.sqrt`).

### `nox.fs`
- `read_to_string(path: str) -> str`: Reads entire file into memory.
- `write_string(path: str, content: str) -> None`: Overwrites destination.
- `exists(path: str) -> bool`: Checks path existence.
- **Limitation**: `nox.fs` does not perform path sandboxing. Atlas checks `exists` before writing unless `overwrite=True` is explicitly passed.

### `nox.test`
- Assertion framework: `assert_eq_int`, `assert_eq_str`, `assert_eq_float`, `assert_true`.

## 2. Language Primitives Verified

- Tagged records and custom classes with typed constructors and methods.
- Optional / nullable types: `T | None` with explicit narrowing `if val != None:`.
- Slices: `list` indexing and slices, string methods (`startswith`, `contains`, `replace`).
- Native `list.sort()` for primitive lists.
- Integer division: `//`.
- Exception hierarchy deriving from `Exception`.

## 3. Streaming CSV & Memory Bounds Verified

- `nox.fs.open_read(path)` and `reader.read_chunk(bytes)`: Incremental file streaming without loading the entire document into RAM.
- `StreamingCsvReader`: Persistent RFC 4180 state machine processing chunks in $O(N)$ linear time without re-scanning buffers.
- **Realistic Streaming Memory Bound**: Memory consumption during streaming is bounded by $O(B + R \times S + L)$, where:
  - $B$ is `chunk_bytes` (the read buffer size),
  - $R$ is `chunk_rows` (the maximum yielded rows per chunk),
  - $S$ is average row size,
  - $L \le \text{max\_record\_bytes}$ is the maximum allowable single record length (default 10 MB, strictly measured in UTF-8 bytes via `nox.strings.byte_len`).
  - Parsing pauses immediately when $R$ rows are accumulated, avoiding unbounded `pending_rows` buffering even when large read chunks are provided.
- **Streaming Schema Policy**:
  - *Inference (Default)*: When `explicit_schema` is omitted, the table schema is inferred from the first chunk's data rows using `infer_schema`. Subsequent chunks are strictly validated against this schema (`ShapeError` on column count mismatch, `MissingValueError` on non-nullable missing cells, and strict cell parsing).
  - *Explicit Schema*: For datasets where later rows may contain wider types (e.g. strings in predominantly numeric columns), passing an `explicit_schema` enforces predefined types and nullability across all chunks. If an empty file (0 bytes) is read with `explicit_schema`, the specified column definitions and types are strictly preserved.
  - *Parameter Validation*: `StreamingCsvReader` strictly validates `chunk_rows > 0`, `chunk_bytes > 0`, `max_record_bytes > 0`, and `len(delimiter) > 0` (raising `CsvImportError("delimiter cannot be empty")`).
- Online Welford / Chan algorithm: Numerically stable running mean, variance, and standard deviation with $O(1)$ memory usage; raises `AggregationError` when observations are insufficient.
- `try...finally`: Guaranteed file descriptor closure and resource cleanup.

## 4. Streaming Transform Pipeline & Online GroupBy Engine

Atlas v0.8.0 provides a fluent, chainable streaming query engine that processes arbitrary-sized CSV datasets with bounded memory:

```text
CSV File -> Streaming Reader -> Filter -> Select -> Online Hash GroupBy -> Aggregate Table
```

- **Chunk-Level Declarative Filtering (`filter`)**:
  - Declarative AST expressions (`col_gt`, `col_eq`, `expr_and`, etc.) are evaluated directly on chunks as they are read.
  - Non-matching rows are discarded immediately, preventing working-set accumulation.
  - Automatically navigates consecutive non-matching chunks when filtering sparse datasets.
- **Chunk-Level Column Projection (`select`)**:
  - Drops unselected columns at the chunk boundary, dramatically compressing row memory width.
- **Online Hash GroupBy Engine (`StreamingGroupedTable`)**:
  - Bounded Memory: $O(G \times K + B + R \times S)$, where $G$ is the number of distinct groups, $K$ is key column count, $B$ is read chunk bytes, $R$ is rows per chunk, and $S$ is average row size.
  - Instead of accumulating raw rows into memory buckets, maintains compact `SingleAggState` per unique group.
  - Computes counts, non-missing counts, sums, online means, Welford variances/stddevs, and extrema simultaneously in a single pass.
  - Complete collision bucket resolution with exact cell equality verification.
- **Materialization & Direct Aggregation**:
  - `.to_table()` drains filtered & selected stream rows into an in-memory typed `Table`.
  - Direct metrics: `.count()`, `.sum(col)`, `.mean(col)`, `.variance(col)`, `.std(col)`.

---

## 8. Window Functions & Analytical Calculations (`window.nox`) (v0.9.0, refined v0.9.1, v0.9.2, v0.9.3)

- **Fluent WindowContext Pipeline with Lazy Column Buffering**:
  - `WindowContext` can be instantiated once via `table.window(partition_by=..., order_by=..., ascending=...)` and chained with `.with_row_number()`, `.with_rank()`, `.with_dense_rank()`, `.with_lag()`, `.with_lead()`, `.with_cumulative_sum()`, `.with_moving_average()`, and finalized with `.to_table()`.
  - Hash-partitioning and $O(N \log N)$ stable merge-sorting are executed strictly **once** across chained window operations.
  - Computed window columns are buffered lazily in `pending_columns` and materialized into the final schema and rows in a single pass ($O(N)$ allocation instead of $K \times O(N \times C)$). Empirically verified at **~2.89x median faster** than independent unpipelined window operations across 10 iterations over 100,000 rows (352 ms control group median vs 122 ms pipeline median). Verified 100% cell-by-cell differential equivalence at the end of the benchmark.
  - Option A Independent Pipeline Contract: All operations in a `WindowContext` pipeline evaluate independently against the base table columns for optimal performance. Referencing a pending pipeline column before `.to_table()` raises an actionable `SchemaError` guiding the caller to materialize with `.to_table()` before chaining dependent computations.
  - Strict order key validation: `with_rank` and `with_dense_rank` require an active context ordering and validate that `order_col` matches the context's primary order key (`SortError`). Multi-column orderings evaluate ties across all specified order columns according to standard SQL semantics.
- **Non-Destructive Row Architecture**:
  - Evaluates analytical window calculations partitioned by keys and ordered by specified columns, appending computed result columns without reordering or altering the input table's original row sequence.
- **Supported Window Operations**:
  - `row_number(t, output_col, partition_by, order_by, ascending)`: Consecutive 1-based row index per partition.
  - `rank(t, order_col, output_col, partition_by, ascending)`: Standard ranking with gap skipping on tied values (e.g. 1, 2, 2, 4).
  - `dense_rank(t, order_col, output_col, partition_by, ascending)`: Dense ranking without gaps on tied values (e.g. 1, 2, 2, 3).
  - `lag(t, target_col, output_col, offset, partition_by, order_by, ascending, default_val)`: Look-behind access to preceding rows.
  - `lead(t, target_col, output_col, offset, partition_by, order_by, ascending, default_val)`: Look-ahead access to succeeding rows.
  - `cumulative_sum(t, target_col, output_col, partition_by, order_by, ascending, skip_null=True, null_policy="skip")`: Running cumulative sums for integers and floats. Supports `null_policy="skip"` (retains running sum), `"null_row"` (null only on null row), and `"propagate"` (nulls cascade to end of partition). When `null_policy != "skip"`, `null_policy` takes explicit precedence over `skip_null`. Includes proactive 64-bit integer overflow protection (`ShapeError`).
  - `moving_average(t, target_col, window_size, output_col, partition_by, order_by, ascending, min_periods)`: Rolling sliding-window means based on valid numeric observation count. Validates `min_periods <= window_size`.

---

## 9. Table Reshaping: Melt & Pivot (`reshape.nox`) (v0.9.0, refined v0.9.1, v0.9.2, v0.9.3)

- **Wide to Long Unpivoting (`melt`)**:
  - Converts tabular data from wide to long format across specified identifier and value columns.
  - Automatic type promotion across heterogeneous value columns (int + float -> float, mixed -> string).
- **Long to Wide Pivoting (`pivot`)**:
  - Reshapes long format data into wide format with unique index and column combinations.
  - Strict duplicate checking: raises `ShapeError` when duplicate `(index, column)` pairs are detected.
  - Collision Safeguards: raises `SchemaError` if a pivot column value matches `index_col`, is empty, or if distinct cell values produce colliding column string representations (e.g. `1` vs `"1"`).
- **Aggregation Pivoting (`pivot_table`)**:
  - Handles duplicate `(index, column)` pairs with configurable aggregation functions:
    - `"mean"`, `"sum"`, `"count"`, `"min"`, `"max"`, `"first"`, `"last"`.
  - Enforces identical collision safeguards as `pivot`.
  - Integer sum aggregations include 64-bit signed integer overflow and underflow detection (`ShapeError`).
  - Missing combinations are consistently populated with `cell_null()`.

---

## 10. Advanced Relational Joins (`join.nox`) (v0.9.0)

- **Full Outer Join (`join_full`, `join_full_keys`)**:
  - Retains all rows from both left and right tables.
  - Left-unmatched rows are padded with nulls on the right; right-unmatched rows are padded with nulls on the left while preserving the right join key.
  - Automatically promotes joined columns to `nullable=True` to maintain schema soundness.
- **Cartesian Product Cross Join (`join_cross`)**:
  - Generates full $M \times N$ Cartesian product.
  - Handles identical column names between tables with customizable suffixes (`_left` and `_right`).

---

## 11. Unified 64-bit Integer Checked Arithmetic (`values.nox`, `atlas.nox`) (v0.9.4, v0.9.5)

- **`checked_add_int64(a: int, b: int, context_msg: str = "") -> int`**:
  - Proactive bounds checking against signed 64-bit limits:
    - $\text{INT64\_MAX} = 9223372036854775807$
    - $\text{INT64\_MIN} = -9223372036854775807 - 1$
  - Prevents silent CPU integer wraparound by raising an explicit, descriptive `ShapeError`.
- **`checked_sub_int64(a: int, b: int, context_msg: str = "") -> int`**:
  - Prevents underflow and overflow in 64-bit signed subtraction, raising `ShapeError`.
- **`checked_mul_int64(a: int, b: int, context_msg: str = "") -> int`**:
  - Prevents boundary overflow in multiplication with full sign checking, raising `ShapeError`.
- **`checked_div_int64(a: int, b: int, context_msg: str = "") -> int`**:
  - Validates divisor $\ne 0$ and guards against two's complement boundary overflow ($\text{INT64\_MIN} // -1$), raising `ShapeError`.
- **Unified Integer Aggregation Contract**:
  - `groupby.agg_sum`: When the target column is `TYPE_INT`, exact integer arithmetic is preserved without 53-bit float precision loss; overflows raise `ShapeError`.
  - `StreamingCsvReader.sum`: When summing a `TYPE_INT` column, values across chunks are checked and accumulated with `checked_add_int64`.
  - `StreamingGroupedTable.aggregate([agg_sum(...)])`: Computes integer totals with `SingleAggState.sum_val_i`, preserving `TYPE_INT` output and enforcing integer overflow bounds.
  - `table.Table.sum_val`: Uses `stat_sum_int` when summing integer columns to prevent integer overflow.
  - `reshape.pivot_table(sum)`: Preserves signed 64-bit integer limits using `checked_add_int64`.
  - `window.cumulative_sum`: Running integer accumulation uses `checked_add_int64`.

---

## 12. Neumaier Compensated Summation (`stats.nox`) (v0.9.5)

- **Algorithm & Numerical Stability**:
  - `stat_sum` and `stat_mean` use Neumaier's enhanced compensated summation algorithm.
  - While naive float accumulation suffers from catastrophic cancellation when small floats are added to large sums, Neumaier maintains a running compensation term $c$ for low-order bits:
    $$\text{sum} + \text{correction}$$
  - Accurately computes $[10^{16}, 1.0, -10^{16}] = 1.0$ (naive summation yields $0.0$).
  - Zero performance degradation on ordinary datasets while guaranteeing IEEE 754 precision preservation across high-dynamic-range floats.

---

## 13. Cross-Backend Compatibility & Ground-Truth Reference Verification (v0.9.5)

- **Backend Matrix Verification (`tests/backend_matrix_test.nox`)**:
  - Systematic cross-backend test suite ensuring portable execution across LLVM, C, and native backends:
    1. INT64 extreme boundaries and two's complement representation.
    2. IEEE 754 float behavior, NaN non-reflexivity (`nan != nan`), Infinity propagation.
    3. Collision-free type-distinct hash keys across string, int, float, bool, and null.
    4. Stable sort tie preservation across primary keys.
    5. Multi-byte UTF-8 character integrity across Turkish characters (`ı, ğ, ü, ş, ö, ç`), CJK characters, and emojis.
    6. Complete `AtlasError` exception class hierarchy.
- **Ground-Truth Analytical Equivalence (`tests/reference_comparison_test.nox`)**:
  - Outputs from multi-key GroupBy, window functions, pivot tables, melt unrolling, and full outer joins are verified directly against analytical reference values precomputed in Pandas 2.x and Polars 1.x.
- **High Cardinality Scaling (`tests/high_cardinality_test.nox`)**:
  - Hash GroupBy scales seamlessly to 50,000 unique groups across 100,000 rows in ~37 ms.
