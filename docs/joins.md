# Relational Table Joins (`join_inner` and `join_left`)

Atlas provides relational inner and left-outer join operations between tables based on matching key columns.

## 1. Signatures

```nox
# Inner Join on shared column names
join_inner(left: Table, right: Table, on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table

# Inner Join on differing column names
join_inner_keys(left: Table, right: Table, left_on: list[str], right_on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table

# Left Outer Join on shared column names
join_left(left: Table, right: Table, on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table

# Left Outer Join on differing column names
join_left_keys(left: Table, right: Table, left_on: list[str], right_on: list[str], suffix_left: str = "_left", suffix_right: str = "_right") -> Table
```

## 2. Invariants & Rules

1. **Duplicate Key Matching (Fanout)**: If a join key appears $M$ times in the left table and $N$ times in the right table, $M \times N$ joined rows are produced.
2. **Column Name Collisions**: Non-key columns that share the same name in both tables are disambiguated by appending `suffix_left` (default `_left`) and `suffix_right` (default `_right`).
3. **Shared Key Deduplication**: When joining on identically named keys (`on=["id"]`), the key column appears once in the resulting table.
4. **Left Join Preservation**: In `join_left`, every left row is preserved. If no match is found in the right table, all columns contributed by the right table are padded with `Cell(TYPE_NULL)`.
5. **Null Key Isolation**: Missing / null values in join keys do **not** match other null keys, preventing accidental cross-joins of records with unassigned identifiers.
