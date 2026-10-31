# Row Filtering & Predicates

## 1. Filter Expressions (`Expr`)

Atlas uses declarative expression trees to filter table rows without dynamic string evaluation or code injection vulnerabilities.

### Supported Operations:
- **Equality**: `col_eq(col, val)`, `col_eq_int`, `col_eq_str`, `col_eq_bool`
- **Inequality**: `col_neq`, `col_neq_int`, `col_neq_str`
- **Comparisons**: `col_gt`, `col_gte`, `col_lt`, `col_lte` (supporting int/float auto-promotion)
- **String predicates**: `col_contains(col, substring)`, `col_starts_with(col, prefix)`
- **Missing values**: `col_is_missing(col)`, `col_not_missing(col)`
- **Set membership**: `col_in(col, list_of_cells)`
- **Logical combinators**: `expr_and(e1, e2)`, `expr_or(e1, e2)`, `expr_not(e)`

## 2. Null Logic in Comparisons

In accordance with standard database semantics:
- Comparing a missing cell with any value via equality or inequality returns `False`.
- To test for missing values, explicitly use `col_is_missing()` or `col_not_missing()`.
