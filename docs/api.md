# Atlas Public API Reference

## 1. Table Creation and I/O

```nox
# In-memory CSV parsing with automated type inference
read_csv(text: str, has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table

# Export Table to CSV string
to_csv(table: Table, include_header: bool = True, safe_spreadsheet: bool = False) -> str

# JSON conversions
table_from_json(json_text: str, explicit_schema: Schema | None = None) -> Table
table_to_json(table: Table) -> str
csv_to_json(csv_text: str, has_header: bool = True) -> str
json_to_csv(json_text: str, include_header: bool = True, safe_spreadsheet: bool = False) -> str

# Filesystem operations (with overwrite protection)
read_csv_file(path: str, has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table
write_csv_file(table: Table, path: str, include_header: bool = True, safe_spreadsheet: bool = False, overwrite: bool = False) -> None
read_json_file(path: str, explicit_schema: Schema | None = None) -> Table
write_json_file(table: Table, path: str, overwrite: bool = False) -> None
```

## 2. Table Methods

```nox
table.row_count() -> int
table.column_count() -> int
table.column_names() -> list[str]
table.cell(row_idx: int, col_idx: int) -> Cell
table.cell_by_name(row_idx: int, col_name: str) -> Cell
table.row(row_idx: int) -> list[Cell]
table.column_cells(col_name: str) -> list[Cell]

# Transformations (immutable, returns new Table)
table.select(col_names: list[str]) -> Table
table.drop(col_names: list[str]) -> Table
table.rename(old_name: str, new_name: str) -> Table
table.filter(predicate: Expr) -> Table
table.sort_by(col_names: list[str], ascending: bool = True, nulls_first: bool = False) -> Table
table.sort_by_keys(col_names: list[str], ascending_flags: list[bool], nulls_first: bool = False) -> Table
table.head(n: int = 5) -> Table
table.tail(n: int = 5) -> Table
table.slice(start: int, end: int) -> Table
table.clone() -> Table

# Missing values
table.count_missing(col_name: str) -> int
table.fill_missing(col_name: str, fill_val: Cell) -> Table
table.drop_missing(col_names: list[str], how: str = "any") -> Table  # how: "any" | "all"

# Summary statistics
table.mean(col_name: str) -> float
table.median(col_name: str) -> float
table.std(col_name: str, is_sample: bool = True) -> float
table.variance(col_name: str, is_sample: bool = True) -> float
table.sum_val(col_name: str) -> float
table.min_val(col_name: str) -> Cell
table.max_val(col_name: str) -> Cell
table.count_val(col_name: str) -> int

# Display
table.print_preview(max_rows: int = 10) -> None
table.print_grid(max_rows: int = 10, max_col_width: int = 30) -> None
table.to_grid_string(max_rows: int = 10, max_col_width: int = 30) -> str

# Quantiles & Outlier Detection
table.quantile(col_name: str, q: float) -> float
table.iqr(col_name: str) -> float
table.filter_outliers_iqr(col_name: str, multiplier: float = 1.5, keep_missing: bool = True) -> Table
```

## 3. Filter Expressions

```nox
col_eq(col: str, val: Cell) -> Expr
col_eq_int(col: str, i: int) -> Expr
col_eq_str(col: str, s: str) -> Expr
col_eq_bool(col: str, b: bool) -> Expr

col_neq(col: str, val: Cell) -> Expr
col_neq_int(col: str, i: int) -> Expr
col_neq_str(col: str, s: str) -> Expr

col_gt(col: str, val: Cell) -> Expr
col_gt_int(col: str, i: int) -> Expr
col_gt_float(col: str, f: float) -> Expr

col_gte(col: str, val: Cell) -> Expr
col_gte_int(col: str, i: int) -> Expr
col_gte_float(col: str, f: float) -> Expr

col_lt(col: str, val: Cell) -> Expr
col_lt_int(col: str, i: int) -> Expr
col_lt_float(col: str, f: float) -> Expr

col_lte(col: str, val: Cell) -> Expr
col_lte_int(col: str, i: int) -> Expr
col_lte_float(col: str, f: float) -> Expr

col_is_missing(col: str) -> Expr
col_not_missing(col: str) -> Expr

col_contains(col: str, sub: str) -> Expr
col_starts_with(col: str, prefix: str) -> Expr
col_in(col: str, values: list[Cell]) -> Expr

expr_and(left: Expr, right: Expr) -> Expr
expr_or(left: Expr, right: Expr) -> Expr
expr_not(inner: Expr) -> Expr
```

## 4. GroupBy & Aggregations

```nox
group_by(table: Table, keys: list[str]) -> GroupedTable

grouped.aggregate(aggregations: list[AggregateDef]) -> Table
grouped.count(output_col: str = "count") -> Table

# Aggregate definitions
agg_count(output_col: str = "count") -> AggregateDef
agg_sum(target_col: str, output_col: str) -> AggregateDef
agg_mean(target_col: str, output_col: str) -> AggregateDef
agg_median(target_col: str, output_col: str) -> AggregateDef
agg_std(target_col: str, output_col: str, is_sample: bool = True) -> AggregateDef
agg_variance(target_col: str, output_col: str, is_sample: bool = True) -> AggregateDef
agg_min(target_col: str, output_col: str) -> AggregateDef
agg_max(target_col: str, output_col: str) -> AggregateDef
agg_count_non_missing(target_col: str, output_col: str) -> AggregateDef
```

## 5. Relational Joins & Concatenation

```nox
# Inner Join
join_inner(left: Table, right: Table, on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table
join_inner_keys(left: Table, right: Table, left_on: list[str], right_on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table

# Left Outer Join
join_left(left: Table, right: Table, on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table
join_left_keys(left: Table, right: Table, left_on: list[str], right_on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table

# Vertical Concatenation (Row-wise)
concat_rows(tables: list[Table]) -> Table

# Horizontal Concatenation (Column-wise)
concat_cols(tables: list[Table], suffix_collisions: bool = True) -> Table
```

## 6. Custom Delimiters & TSV

```nox
# In-memory custom delimiters
parse_delimited(text: str, delimiter: str = ",") -> list[list[str]]
dump_delimited(rows: list[list[str]], delimiter: str = ",") -> str
read_delimited(text: str, delimiter: str = ",", has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table
to_delimited(table: Table, delimiter: str = ",", include_header: bool = True, safe_spreadsheet: bool = False) -> str

# TSV convenience functions
read_tsv(text: str, has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table
to_tsv(table: Table, include_header: bool = True, safe_spreadsheet: bool = False) -> str

# Filesystem operations
read_delimited_file(path: str, delimiter: str = ",", has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table
write_delimited_file(table: Table, path: str, delimiter: str = ",", include_header: bool = True, safe_spreadsheet: bool = False, overwrite: bool = False) -> None
read_tsv_file(path: str, has_header: bool = True, explicit_schema: Schema | None = None, blank_is_missing: bool = True) -> Table
write_tsv_file(table: Table, path: str, include_header: bool = True, safe_spreadsheet: bool = False, overwrite: bool = False) -> None
```

## 7. Relational SQL & Feather Bridge

```nox
# DDL & INSERT code generation
table_to_create_table_sql(table: Table, table_name: str, dialect: str = "sqlite", primary_key_col: str = "") -> str
table_to_insert_sql(table: Table, table_name: str) -> str

# Record and DB Value mapping
table_from_records(col_names: list[str], col_types: list[int], rows: list[list[Cell]]) -> Table
cell_from_db_value(kind: int, i: int, f: float, s: str, b: bool, is_null: bool) -> Cell
```

## 8. Tabular Data Validation

```nox
validator = TableValidator()
validator.add_rule(rule: ColumnRule) -> TableValidator

# Rules
rule_non_empty(col_name: str) -> ColumnRule
rule_range(col_name: str, min_val: float, max_val: float) -> ColumnRule
rule_min(col_name: str, min_val: float) -> ColumnRule
rule_max(col_name: str, max_val: float) -> ColumnRule
rule_length(col_name: str, min_len: int, max_len: int) -> ColumnRule
rule_allowed_values(col_name: str, allowed: list[str]) -> ColumnRule
rule_email(col_name: str) -> ColumnRule

# Execution & Filtering
report: ValidationReport = validator.validate(table: Table)
report.is_valid() -> bool
report.error_count() -> int
report.errors_for_row(row_idx: int) -> list[ValidationError]
report.errors_for_column(col_name: str) -> list[ValidationError]
report.summary() -> str

clean_table: Table = validator.filter_valid(table: Table)
quarantine_table: Table = validator.filter_invalid(table: Table)
```

## 9. Low-Memory Streaming CSV & Online Aggregations

```nox
# Streaming Reader with bounded memory and state machine
reader: StreamingCsvReader = open_csv_stream(path: str, chunk_rows: int = 1000, options: CsvOptions | None = None, explicit_schema: Schema | None = None)

while reader.has_next():
    chunk: Table = reader.next_chunk()
reader.close()

# Online Aggregations (O(1) memory, numerically stable Welford/Chan algorithm)
stream_count(path: str, options: CsvOptions | None = None) -> int
stream_sum(path: str, col_name: str, options: CsvOptions | None = None) -> float
stream_mean(path: str, col_name: str, options: CsvOptions | None = None) -> float
stream_variance(path: str, col_name: str, options: CsvOptions | None = None, is_sample: bool = True) -> float
stream_std(path: str, col_name: str, options: CsvOptions | None = None, is_sample: bool = True) -> float

# Configurable options
opts = csv_options(delimiter=",", has_header=True, blank_is_missing=True, strict=False, safe_spreadsheet=False, max_record_bytes=10485760)
```

### Memory Complexity & Schema Policy

- **Streaming Memory Bound**: Bounded by $O(B + R \times S + L)$, where $B$ is `chunk_bytes`, $R$ is `chunk_rows`, $S$ is average row size, and $L \le \text{max\_record\_bytes}$ (default 10 MB in true UTF-8 bytes). Parsing pauses immediately upon accumulating $R$ rows.
- **Schema Policy**: When `explicit_schema` is omitted, the schema is inferred from the first chunk's data rows. Subsequent rows are strictly validated against this schema (`ShapeError` on column count mismatch, `MissingValueError` on non-nullable nulls). Provide `explicit_schema` when types must be enforced upfront.
