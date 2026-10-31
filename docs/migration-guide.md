# Atlas Migration Guide

## Upgrading to Atlas 0.1.0

Atlas 0.1.0 is the initial public release of Atlas for the Nox programming language.

### Key API Conventions
- `read_csv(text)` takes CSV text, not a file path. To read directly from a file on disk, use `read_csv_file(path)`.
- `filter(predicate)` accepts strongly-typed declarative `Expr` expressions (e.g. `col_gt_int("age", 25)` or `expr_and(...)`).
- Statistical operations on single columns (`mean`, `median`, `std`, `variance`) are available directly as methods on `Table` (e.g. `table.mean("salary")`).
- Multi-column sorting (`table.sort_by(["col1", "col2"])`) is stable and guarantees preservation of original relative order on ties.
