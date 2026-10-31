# Missing Value Semantics & Operations

## 1. Missing Value Contract

In Atlas, missing data is explicitly tracked via `Cell(TYPE_NULL)`:
- `cell.is_missing()` returns `True`.
- It is distinct from `""` (empty string), `0` (zero), `false` (boolean false), or `NaN` (float NaN).
- Missing values do not participate as zeroes in numeric calculations (`sum`, `mean`, `variance` ignore missing values).

## 2. Operations

### `table.count_missing(col_name: str) -> int`
Counts missing cells in the specified column.

### `table.fill_missing(col_name: str, fill_val: Cell) -> Table`
Returns a new table where missing cells in `col_name` are replaced with `fill_val`.

### `table.drop_missing(col_names: list[str], how: str = "any") -> Table`
Drops rows where missing cells occur in the target columns:
- `how="any"` (default): Drops row if any of the specified columns is missing.
- `how="all"`: Drops row only if all of the specified columns are missing.
