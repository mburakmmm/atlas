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
