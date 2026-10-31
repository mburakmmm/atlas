# Data Model & Invariants

## 1. Cell Value Model

Each table cell is an instance of `Cell`:
- `kind`:
  - `0`: `TYPE_NULL` (missing observation)
  - `1`: `TYPE_INT` (64-bit signed integer)
  - `2`: `TYPE_FLOAT` (64-bit IEEE-754 float)
  - `3`: `TYPE_STR` (UTF-8 string)
  - `4`: `TYPE_BOOL` (boolean)

### Semantics
- A missing cell is **not** equivalent to empty string `""`, zero `0`, or `false`.
- Missing cells have `is_missing() == True`.
- Numeric comparison between `int` and `float` cells is value-based (`cell_int(10).equals(cell_float(10.0)) == True`).

## 2. Schema

`Schema` holds an ordered list of `ColumnDef`:
- `name: str`: Non-empty, unique across the schema.
- `data_type: int`: Logical expected type.
- `nullable: bool`: Whether missing cells are permitted.

### Invariants
1. **Row Length Invariant**: Every row in `Table.rows` has length strictly equal to `schema.column_count()`.
2. **Nullability Invariant**: A non-nullable column (`nullable == False`) rejects missing cells at row ingestion time, raising `MissingValueError`.
3. **Ordering**: Schema column order is preserved across filtering, sorting, and serialization.
