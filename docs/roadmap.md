# Atlas Roadmap

## Version 0.1.0 (Current Release)
- RFC 4180 CSV parsing and dumping via `nox.csv`
- Automated schema inference (int, float, bool, string)
- In-memory 2D typed table (`Table`, `Schema`, `Cell`)
- Column operations: selection, dropping, renaming
- Declarative row filtering (`Expr`) with comparisons and logical combinators
- Multi-column stable merge sort ($O(N \log N)$)
- Numerically stable summary statistics (mean, median, variance, stddev via Welford's algorithm)
- Multi-key group-by and aggregations
- Missing value management (`count_missing`, `fill_missing`, `drop_missing`)
- Bidirectional JSON ↔ Table ↔ CSV conversion via `JsonWriter`
- Filesystem I/O with overwrite protection and formula injection mitigation

## Version 0.2.0 (Planned)
- Inner and left table joins on matching key columns
- Table vertical concatenation (`concat_rows`) and horizontal concatenation (`concat_columns`)
- Custom delimiter support (semicolon `;`, tab `\t`)
- Quantiles & percentile estimation (linear interpolation)
- Optional pretty terminal table formatting adapter for Rune
