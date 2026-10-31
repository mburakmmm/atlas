# Atlas Architecture

Atlas is a lightweight, pure Nox-native tabular data analysis and statistics library designed to make data cleaning, transformation, and numerical aggregation fast, predictable, and memory-safe.

## Architectural Layers

```text
User Application / CLI / Scripts
               |
        Atlas Public Facade (`atlas.nox`)
               |
  +------------+------------+-------------+
  |            |            |             |
Table        Schema       Expr          Stats
(`table.nox`) (`schema.nox`) (`expr.nox`) (`stats.nox`)
  |            |            |             |
  +------------+------------+-------------+
               |
          Tagged Value / Cell Representation (`values.nox`)
               |
  +------------+------------+
  |                         |
CSV Adapter (`csv_adapter.nox`) JSON Adapter (`json_adapter.nox`)
  |                         |
`nox.csv`                 `nox.json`
  \                         /
          `nox.fs` (`io.nox`, opt-in)
```

## Module Separation of Concerns

- **`values.nox`**: Defines `Cell` (tagged union for int, float, str, bool, null), comparison logic, and explicit missing value semantics.
- **`schema.nox`**: Enforces column name uniqueness, typing, and immutability contracts across transformations.
- **`expr.nox`**: Declarative filter expressions (comparisons, strings, logical combinators, membership) evaluating row by row without code injection.
- **`sort.nox`**: Multi-column stable merge sort guaranteeing $O(N \log N)$ worst-case performance and tie-stability.
- **`stats.nox`**: Numerically stable aggregations implementing Welford's algorithm for sample and population variance/stddev.
- **`table.nox`**: 2D in-memory row storage and table transformation operations (`select`, `drop`, `rename`, `head`, `tail`, `slice`, `filter`, `sort_by`, `fill_missing`, `drop_missing`).
- **`groupby.nox`**: Group-by engine computing multi-column aggregations preserving deterministic group order.
- **`csv_adapter.nox`**: Integrates with `nox.csv` for RFC 4180 parsing, automated type inference, formula injection mitigation, and CSV text export.
- **`json_adapter.nox`**: Integrates with `nox.json` and `JsonWriter` for array-of-objects round-tripping.
- **`io.nox`**: Filesystem convenience functions with explicit overwrite protection.
- **`atlas.nox`**: Top-level facade unifying all components under a single importable namespace.
