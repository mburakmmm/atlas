# Table Concatenation (`concat_rows` and `concat_cols`)

Atlas supports both vertical (row-wise) and horizontal (column-wise) concatenation.

## 1. Vertical Row Concatenation (`concat_rows`)

```nox
concat_rows(tables: list[Table]) -> Table
```

### Semantics
- Combines multiple tables with compatible schemas vertically.
- **Column Alignment**: Columns are automatically aligned by name, even if individual tables have columns in differing orders.
- **Missing Columns**: If a column in the target schema is absent in one of the subsequent tables, it is populated with `Cell(TYPE_NULL)` (provided the target column is nullable). If a non-nullable column is absent, raises `ShapeError`.

## 2. Horizontal Column Concatenation (`concat_cols`)

```nox
concat_cols(tables: list[Table], suffix_collisions: bool = True) -> Table
```

### Semantics
- Combines columns of multiple tables side-by-side.
- **Row Count Invariant**: Every table in the list must possess the exact same row count; otherwise, raises `ShapeError`.
- **Collision Disambiguation**: If columns have identical names, `suffix_collisions=True` appends `_2`, `_3`, etc. If `suffix_collisions=False`, duplicate names trigger `SchemaError`.
