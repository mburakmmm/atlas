# Atlas Roadmap & Release History

## Version 1.0.0 (Current Production Release)
- **Complete Typed Column Model (`values.nox`, `schema.nox`, `temporal.nox`)**:
  - `TYPE_NULL`, `TYPE_INT` (exact 64-bit checked arithmetic), `TYPE_FLOAT` (IEEE-754 + Neumaier compensated summation), `TYPE_STR`, `TYPE_BOOL`, `TYPE_DECIMAL` (exact base-10 via `nox.decimal`), and `TYPE_DATETIME` / `TYPE_DATE` (`nox.time.DateTime` + `nox.timezone`).
  - `enum CellKind` and `enum ExprOp` with exhaustive `match / case` helpers and $O(1)$ `Schema.index_of` / `Set[str]`-backed `drop_columns`.
- **Relational, Analytical & Reshaping Engine (`table.nox`, `join.nox`, `groupby.nox`, `window.nox`, `reshape.nox`)**:
  - Inner, Left, Right, and Full Outer Joins with hash-probe execution.
  - Multi-key `GroupBy`, single-pass lazy `WindowContext` (`row_number`, `rank`, `dense_rank`, `lag`, `lead`, `cumulative_sum`, `moving_average`), `pivot_table`, and `melt`.
  - Shared-nothing deterministic `parallel=True` execution across `Table.filter`, `GroupedTable.aggregate`, `WindowContext`, and `read_csv_files_parallel`.
- **Portable & Desktop Turbo I/O (`csv_adapter.nox`, `delimited.nox`, `json_adapter.nox`, `archive.nox`, `atlas_fast.nox`)**:
  - Zero-copy slice-based delimited parser/dumper, native JSON tape `int64` / `Decimal` preservation, `@dataclass @derive("json", "order", "copy")` row interop (`table_to_json_rows`, `table_from_json_rows`).
  - Path-confined (`nox.path.safe_join`) and crash-safe atomic (`nox.fs.write_string_atomic`) file I/O.
  - Portable `.gz`, `.zst` (decompress), `.zip`, and `.tar.gz` archives (`archive.nox`).
  - Opt-in desktop turbo module (`atlas_fast.nox`) wrapping `nox.yyjson` (strict/relaxed/JSON5), `nox.xxhash` (`xxh3_64`/`xxh3_128_hex` + fast dedup), `nox.pcre2` (JIT regex filter/replace), `nox.lz4`, `nox.zstd.compress`, and `nox.brotli`.
- **Verification & Quality Gates**:
  - 47 automated test suites (unit, regression, differential, high-cardinality, Pandas/Polars ground-truth reference, LLVM/QBE cross-backend matrix, and `nox.property` shrinking property-based tests).

## Post-1.0 Future Directions
- Optional columnar arena projections for numeric-only matrix kernels.
- Extended time-series resampling and rolling time-window joins.
