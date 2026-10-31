# Group-By Operations & Aggregations

## 1. Grouping Model

`group_by(table: Table, keys: list[str]) -> GroupedTable` partitions a table into distinct groups based on one or more key columns.

### Semantics
- Group ordering is **deterministic** and follows the order of first occurrence.
- Missing values in group keys form valid distinct groups.
- Source table rows are never mutated.

## 2. Aggregations

The `.aggregate(list[AggregateDef])` method accepts aggregate definitions:
- `agg_count(output_col)`: Total rows in group.
- `agg_count_non_missing(target_col, output_col)`: Count of non-null observations in column.
- `agg_sum(target_col, output_col)`: Sum of numeric values.
- `agg_mean(target_col, output_col)`: Arithmetic mean.
- `agg_median(target_col, output_col)`: Median (middle value or average of two middle values).
- `agg_std(target_col, output_col, is_sample=True)`: Standard deviation via Welford's algorithm.
- `agg_variance(target_col, output_col, is_sample=True)`: Variance via Welford's algorithm.
- `agg_min(target_col, output_col)`: Minimum value.
- `agg_max(target_col, output_col)`: Maximum value.
