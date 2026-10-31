# AGENTS.md — ATLAS
## Lightweight tabular data processing and statistics for the Nox programming language

> **Binding engineering contract** for Cursor, Codex, Claude Code, and other coding agents working on Atlas.
>
> **Identity:** Atlas (`atlas`) is a **small-to-medium, Nox-native, Pandas-inspired table/data-analysis library**, centered on CSV, typed columns, row selection, filtering, sorting, grouping, summary statistics, missing values, and JSON ↔ CSV conversion. **It is not a complete Pandas clone**, a distributed analytics engine, a database ORM, or a machine-learning framework.
>
> **Source of truth:** https://github.com/mburakmmm/nox-lang and https://noxlang.com, plus the current Atlas repository. Verify every API against the installed Nox compiler before implementing it.

---

# 1. Mission and product principles

Atlas should make everyday data cleaning and analysis straightforward in Nox without dragging in heavyweight dependencies.

**Primary use cases**
- Read a CSV, inspect schema and row counts, clean missing values.
- Select/rename/drop columns; filter rows by conditions.
- Sort and group by one or more columns; compute aggregates.
- Compute mean, median, variance, standard deviation, min/max, sum and count.
- Convert tabular JSON arrays to/from CSV, with explicit schema policies.
- Export reproducible reports and data transformations.
- Process medium-sized in-memory datasets efficiently.

**Non-goals for v1.0**
- Full Pandas API parity.
- Excel, Parquet, Arrow, NumPy ABI, GPU, Spark, distributed data frames.
- SQL engine, ORM, graph engine, machine-learning framework.
- Interactive plotting and dashboard rendering.
- Arbitrary user-code expression evaluation.
- Automatic inference that silently corrupts types or precision.
- Implicit lazy execution engine or multi-threading by default.

**Core principles**
1. **Nox-native:** idiomatic static types and Nox stdlib.
2. **Reuse existing capabilities:** do not duplicate `nox.csv` and `nox.json` unnecessarily.
3. **Explicit data semantics:** column types, NULL/missing, sorting and grouping rules are documented.
4. **Correct before clever:** data integrity and numerical correctness outweigh micro-optimizations.
5. **Small modular API:** simple tables and transforms first.
6. **Predictable performance:** no hidden O(n²) behavior for ordinary operations.
7. **Determinism:** stable output ordering and reproducible aggregates where specified.
8. **Framework independence:** no mandatory Nyx, Aether, Aura, Noxen, Nova, Rune, Feather, or Chronos.
9. **Honest compatibility:** do not advertise unsupported Nox types, backends, platforms, encodings, or streaming.
10. **Test every public feature:** compile real Nox examples and run automated tests.

---

# 2. Mandatory Nox capability audit

Before any implementation task:

1. Inspect Atlas repository, tests, manifests, existing design decisions.
2. Read Nox docs/source for `nox.csv`, `nox.json`, `nox.fs`, `nox.strings`, `nox.math`, `nox.mathx`, `nox.test`, and any verified statistics module.
3. Inspect Nox language support for `list`, `dict`, records/classes, enums/unions, option/nullable types, generics, protocols, callbacks/closures, iterators, sorting, error handling, memory ownership.
4. Verify Nox package layout (`nox.json`, `nox.lock`) and compiler commands.
5. Run `noxc version`; record compiler, backend and OS.
6. Compile small proofs for typed column storage, nullable values, row selection and custom predicates.
7. Record actual available primitives and limitations in `docs/capabilities.md`.
8. If source docs are stale, trust compiling implementation and document discrepancies.
9. Do not assume Python syntax or APIs are supported merely because Nox resembles Python.

## 2.1 Verified baseline from NOX documentation (recheck at implementation time)

**`nox.csv`**
- `parse(text)` → `list[list[str]]`, RFC 4180-style quoted fields, embedded commas/quotes/newlines.
- `parse_dicts(text)` → header-based `list[dict[str,str]]`.
- `dump_row(fields)` and `dump(rows)` generate CSV text.
- Parsing and dumping operate on **text**, not files.
- Input supports `\n` and `\r\n`; output uses `\n`.
- **All parsed values are strings.**
- **Custom delimiters (semicolon/tab), BOM handling and direct file I/O are documented out of scope.**
- Do not invent `nox.csv.read`, `nox.csv.write`, custom delimiter arguments, or native streaming.

**`nox.json`**
- `parse(text)` returns `JsonValue`.
- `JsonValue` has kind-checking/accessor functions for null, boolean, number, string, array and object.
- JSON numeric accessor returns **float**, so integers above 2^53 can lose precision.
- `JsonWriter` supports incremental text construction but does not validate balanced nesting.
- Do not assume a JSON object has a direct `dict` conversion API.
- Avoid silent loss of integer precision in JSON round trips.

**`nox.fs`**
- `read_to_string(path)` reads an **entire file into memory**.
- `write_string(path, content)` **truncates/overwrites** the destination.
- Operations are synchronous; file paths are **not automatically sandboxed or canonicalized**.
- Do not promise chunked streaming or atomic writes without verified facilities.

**`nox.strings`**
- `byte_len(s)` measures UTF-8 bytes, `len(s)` counts code points.
- ASCII-only case conversions in `nox.strings` are not Unicode case folding.
- Never conflate bytes, code points and display width.

**`nox.math` / `nox.mathx`**
- `nox.math` uses libc `libm`, hosted only.
- `nox.mathx` is recommended for freestanding-capable pure math where supported.
- Floating point follows IEEE-754 behavior; `NaN`/`Inf` must be handled deliberately.

**Unverified until tested:** native streaming file readers, columnar SIMD, parallel loops, custom CSV delimiters, lazy evaluation, zero-copy slices, Arrow buffers, vectorized kernels, `Decimal`, arbitrary-precision integers, Unicode-aware case folding, stable sorting guarantees, runtime reflection. Never claim these without evidence.

---

# 3. Binding engineering rules

1. Write actual, compiling **Nox**, not Python disguised as Nox.
2. Never modify the NOX compiler, Zig runtime or stdlib without explicit permission.
3. No mandatory native C/C++/Rust/Go dependencies.
4. No mandatory dependence on Nyx, Aether, Aura, Noxen, Feather or other frameworks.
5. No magic reflection-based schema discovery unless compiler support is proven.
6. Keep data types explicit and prevent silent truncation or precision loss.
7. Missing/NULL is not the same as empty string, zero, `False`, or NaN.
8. Column names must be unique or duplicates must trigger an explicit documented policy.
9. Table operations must preserve row/column alignment.
10. A failed transformation must not leave partially mutated data unless clearly documented.
11. No hidden global table registry or global mutable parsing configuration.
12. No arbitrary code evaluation in filters, JSON fields or expressions.
13. No implicit network access or database queries.
14. No unsafe automatic file overwrite by default.
15. No fabricated streaming or big-data capability.
16. Document time/memory complexity for expensive operations.
17. Keep public APIs stable and documented.
18. Every bug fix adds a regression test.
19. Never claim cross-platform support without tests.
20. Never report test success unless tests were actually run.

---

# 4. Architecture

```text
Application / CLI / optional framework integration
                      |
                 Atlas public API
                      |
       +--------------+---------------+
       |              |               |
   atlas.table     atlas.expr       atlas.stats
   atlas.schema    atlas.filter     atlas.groupby
   atlas.column    atlas.sort       atlas.missing
       |              |               |
       +--------------+---------------+
                      |
                atlas.values
                      |
        +-------------+--------------+
        |                            |
    atlas.csv                     atlas.json
        |                            |
     nox.csv                     nox.json
        \                            /
                nox.fs (opt-in)
```

**Module boundaries**
- `values` defines cell values, types, NULL semantics and conversions.
- `schema` defines column names/types and validates table invariants.
- `table` owns row/column storage and basic selection.
- `expr` contains pure filter expressions and predicates.
- `groupby` and `stats` depend on table/values, not I/O.
- `csv` and `json` are adapters; core transformations never depend on filesystem.
- `io` file convenience functions use `nox.fs` only when requested.
- Optional SQL/Feather integration is separate, never in the core.

## Suggested repository layout (adjust to actual Nox module rules)

```text
atlas/
  AGENTS.md
  README.md
  LICENSE
  CHANGELOG.md
  CONTRIBUTING.md
  nox.json
  nox.lock                 # only if needed
  docs/
    architecture.md
    capabilities.md
    api.md
    data-model.md
    csv.md
    json.md
    filtering.md
    grouping.md
    statistics.md
    missing-values.md
    performance.md
    security.md
    compatibility.md
    roadmap.md
    migration-guide.md
  atlas/
    values.nox
    schema.nox
    column.nox
    table.nox
    expr.nox
    filter.nox
    sort.nox
    groupby.nox
    stats.nox
    missing.nox
    csv.nox
    json.nox
    io.nox
    errors.nox
  tests/
    values_test.nox
    schema_test.nox
    table_test.nox
    csv_test.nox
    json_test.nox
    filter_test.nox
    sort_test.nox
    groupby_test.nox
    stats_test.nox
    missing_test.nox
    regression_test.nox
  examples/
    quickstart.nox
    csv_cleaning.nox
    group_summary.nox
    json_conversion.nox
  benchmarks/
  .github/workflows/
```

This tree is a design suggestion, not proof of valid import syntax. Do not create empty placeholder files.

---

# 5. Data model and table invariants

## 5.1 Recommended representation

Choose after measuring and verifying Nox types:

**MVP option A: row-oriented**
- `list[list[Cell]]` with explicit schema.
- Simple CSV import and row filtering.
- Less efficient for repeated column aggregates.

**MVP option B: column-oriented**
- ordered list of typed columns with equal lengths;
- efficient column selection and numerical aggregation;
- harder to represent mixed types if Nox union types are limited.

**Recommendation:** start with explicit schema + a simple, testable row representation, then benchmark before adopting columnar storage. Do not expose storage internals as the public API.

## 5.2 Cell types

Target logical kinds:
- string
- integer
- float
- boolean
- missing/null

Optional later:
- datetime/date
- decimal
- binary
- categorical
- nested JSON

Implementation must match verified Nox type system; use a tagged value design if necessary. Do not invent sum types or generics.

## 5.3 Schema

- Ordered column names and logical types.
- Explicit nullable flag per column.
- Unique column names by default.
- Validate row length and cell type against schema.
- Define type promotion only through explicit, documented rules.
- No implicit schema mutation when adding rows.
- Stable schema order in CSV/JSON output.
- Clear behavior for empty tables with schema vs without schema.
- Explicit rename and duplicate-name handling.
- Column index bounds checked.
- Preserve schema across filter, sort and select.

## 5.4 Data ownership

- Document whether tables own their data, copy it, or expose views.
- MVP should prefer clear copy semantics over unsafe shared mutable slices.
- Define `clone`, `select`, `filter`, `sort` mutation behavior explicitly.
- Prefer immutable-by-default transformation APIs; use explicit `in_place` only if supported and safe.
- Do not return references that outlive their storage.
- Never allow one column to have a different row count from others.
- Row IDs/indexes are logical positions, not automatically unique database keys.

---

# 6. Public API proposal

All signatures here are **conceptual, not verified Nox code**.

```text
atlas.read_csv(path, options) -> Table
atlas.parse_csv(text, options) -> Table
atlas.table(schema, rows) -> Table
table.select(["name", "age"]) -> Table
table.filter(expression) -> Table
table.sort_by(["age"], ascending=true) -> Table
table.group_by(["department"]).aggregate({...}) -> Table
table.fill_missing("age", 0) -> Table
table.drop_missing(["age"]) -> Table
table.to_csv() -> string
table.to_json() -> string
atlas.mean(table, "age") -> float/optional
```

**Do not publish as executable Nox until syntax, callback support, constructors, methods and type returns are compiled.**

API design guidelines:
- Favor a small number of obvious methods.
- Use explicit column names or validated indexes.
- Avoid ambiguous positional boolean arguments.
- Keep parsing options distinct from table transforms.
- Return structured errors rather than silently dropping malformed data.
- Expose an opt-in `try_*` or result style only if idiomatic Nox supports it.
- Do not require Python-style lambda callbacks; provide typed expression predicates if callbacks are difficult.
- Avoid pandas-specific naming that suggests unsupported pandas semantics.
- Public API must remain independent of storage layout.

---

# 7. CSV reading and writing

## 7.1 Reuse `nox.csv`

- Use `nox.csv.parse` and `nox.csv.dump` for default comma-delimited text.
- File adapters may call `nox.fs.read_to_string` and `nox.fs.write_string`.
- Do not reimplement RFC 4180 parsing just to add a thin wrapper.
- Custom delimiters, BOM and streaming require separate parser design and tests.
- Do not pretend stdlib accepts a delimiter argument.

## 7.2 Header and row policies

Define:
- `has_header` true/false;
- supplied column names when no header;
- duplicate headers: reject by default;
- blank header: reject or deterministic generated name;
- missing cells in a row: error, pad with missing, or configured policy;
- extra cells: error by default;
- blank lines: documented behavior;
- whitespace trimming: opt-in, never silently strip meaningful text;
- embedded newlines, commas and quotes: preserve correctly;
- line endings: accept CRLF/LF according to stdlib;
- encoding: UTF-8 text unless verified otherwise.

## 7.3 Type inference

**Default MVP:** strings unless schema or inference is explicitly requested.

When inference is enabled:
- deterministic sampling or full-scan policy;
- explicit type precedence and promotion;
- strict parsing for int/float/bool;
- locale-independent numeric formats;
- distinguish empty string from missing according to configured sentinels;
- reject inconsistent data in strict mode;
- configurable permissive mode with error counts;
- avoid silently turning IDs with leading zeros into integers;
- avoid silent precision loss for long integers and decimals;
- do not interpret arbitrary strings as dates by default.

## 7.4 CSV writing and safety

- Preserve schema order and field content.
- Define missing-value serialization (empty field or configured token).
- Round-trip quoting and embedded newlines.
- Document float formatting and precision limitations.
- **Spreadsheet formula injection:** when CSV may be opened in Excel/LibreOffice, values beginning with `=`, `+`, `-`, `@`, or certain control prefixes can be interpreted as formulas. Provide an explicit spreadsheet-safe export mode, with documented transformation and reversibility limitations.
- Never silently modify raw CSV output for machine-to-machine use unless safe mode is selected.
- Avoid overwriting existing files by default; if the Nox filesystem API cannot support exclusive create, check and document race limitations rather than claiming atomic safety.
- No streaming claims while reading entire text into memory.

---

# 8. JSON ↔ table ↔ CSV conversion

Supported JSON input shapes for MVP:
- array of objects;
- optionally array of arrays with explicit schema.

Rules:
- Preserve JSON object key order only when documented and useful; schema order is authoritative.
- Determine schema from explicit declaration or deterministic union of keys.
- Missing key ≠ explicit JSON `null`.
- Nested arrays/objects: reject, stringify as JSON, or flatten only under explicit option.
- Do not silently flatten nested objects.
- JSON numbers are floats in documented `nox.json`; large integers above 2^53 may lose precision. Reject or document unsafe conversion.
- Reject unsupported NaN/Infinity in JSON output.
- Distinguish JSON boolean from string `"true"`.
- Empty array needs explicit schema or produces a documented empty table.
- Consistent JSON output for missing cells (null vs omit key) must be configurable.
- CSV has no native types; JSON→CSV→JSON cannot guarantee original types without a schema.
- Add round-trip tests and property tests for supported types.
- Use `JsonWriter` correctly with balanced containers and key/value order.
- No arbitrary JSONPath engine in MVP.

---

# 9. Column selection and table operations

## 9.1 Select/drop/rename

- Select columns in caller-specified order.
- Reject unknown column names by default.
- Duplicate selected columns require an explicit policy.
- Dropping columns must preserve row count.
- Renaming must prevent name collisions.
- Zero-column tables need explicit supported/unsupported semantics.

## 9.2 Filter

MVP operators:
- equals / not equals;
- greater/less comparisons;
- AND / OR / NOT with explicit precedence;
- is missing / is not missing;
- contains / starts-with for strings if verified;
- membership (`IN`) with defined empty-set behavior.

Rules:
- Comparisons of incompatible types must error or follow documented policy.
- Missing values require explicit three-valued or two-valued logic. Do not accidentally equate missing with false.
- No user-controlled expression evaluation or code injection.
- Callback predicates only if real Nox function types permit.
- Preserve input row order.
- Deterministic filtering with no side effects.
- Keep expression depth bounded for untrusted inputs.

## 9.3 Sorting

- Stable sort is strongly preferred; implement/test stability if stdlib sort does not guarantee it.
- Multi-key lexicographic ordering.
- Per-key ascending/descending if feasible.
- Explicit missing-first or missing-last.
- Deterministic tie behavior.
- Numeric sort must be numeric, not lexicographic.
- Define NaN ordering.
- Case-sensitive string sorting by default; locale collation is optional and requires verified Unicode support.
- Avoid relying on hash-map iteration order.

## 9.4 Joins and concatenation (post-MVP)

- Optional inner/left joins with explicit keys.
- Define duplicate-key fanout and output column collision rules.
- Never silently drop unmatched rows when left join is requested.
- Detect join explosion risks and provide safeguards.
- Row concatenation requires schema compatibility and clear type promotion.
- Column concatenation requires equal row counts or explicit index alignment.
- No relational query optimizer in initial release.

---

# 10. Group-by and aggregation

## 10.1 Grouping semantics

- One or multiple grouping keys.
- Stable group order: first occurrence or sorted, documented.
- Define missing-key grouping policy.
- Equality and hashing semantics consistent with cell types.
- Distinguish integer `1`, float `1.0`, string `"1"` unless explicit coercion.
- NaN key handling explicit.
- Preserve deterministic output column names.

## 10.2 Aggregations

MVP:
- `count_rows`
- `count_non_missing`
- `sum`
- `min`
- `max`
- `mean`

Later:
- median
- variance/stddev
- distinct count
- first/last
- quantiles
- multiple aggregates per column
- weighted mean

Rules:
- Aggregate numeric columns only where required.
- Missing values excluded or propagated according to explicit policy.
- Empty groups and all-missing groups have defined outputs.
- Count rows is not the same as count non-missing values.
- Prevent integer overflow in sums; use explicit strategy.
- Do not silently coerce numeric strings.
- Grouping must not mutate source data.
- Test duplicate keys, missing keys, one-row groups, empty inputs, large cardinality.

---

# 11. Statistics: numerical contracts

## 11.1 Definitions

- Mean = sum(x)/n for non-missing numeric observations.
- Median = middle ordered value or mean of two middle values; define behavior for empty input.
- Population variance uses denominator `n`.
- Sample variance uses denominator `n-1`; **default must be documented**.
- Standard deviation = sqrt(variance).
- Min/max and count have explicit empty-input behavior.
- Percentiles/quantiles are deferred until interpolation convention is specified.

## 11.2 Numerical stability

- Avoid naive summation for large magnitude cancellation when feasible; consider compensated summation.
- Prefer Welford's online algorithm for variance/stddev rather than unstable `E[x²] - E[x]²`.
- Handle n=0 and sample n<2 explicitly.
- Avoid integer overflow before conversion.
- Define NaN and ±Infinity policy (reject, propagate or omit; never silently choose).
- No silent decimal/money precision claims.
- Test constant vectors, negative values, huge/small magnitudes, mixed signs, one item, all missing.
- Compare against known reference calculations with documented tolerances.
- Do not claim bit-identical results across LLVM/QBE/libm implementations without tests.

## 11.3 Median and quantiles

- Median sorting may be O(n log n); document it.
- Avoid modifying the input column when calculating median.
- Handle even-length median without overflow from `(a+b)/2` when possible.
- Quantiles must specify interpolation method and boundaries before implementation.

---

# 12. Missing data semantics

**Foundational rule:** `Missing`, empty string, zero, false, JSON null, CSV empty field, and float NaN are not automatically equivalent.

Recommended design:
- explicit missing marker or nullable/tagged cell type;
- `is_missing`;
- `fill_missing`;
- `drop_missing`;
- `count_missing`;
- optional `replace_missing`/forward-fill later.

Rules:
- CSV blank-to-missing conversion is configurable.
- JSON `null` maps to missing by documented default.
- JSON absent key is tracked as missing, optionally distinguishable if required.
- `fill_missing` must validate replacement type.
- `drop_missing` supports selected columns and `any`/`all` policy.
- No implicit forward-fill across groups.
- Missing values do not become zeros in statistics.
- Distinguish `NaN` handling from missing; optional `nan_is_missing` flag.
- Empty datasets and all-missing columns have deterministic behavior.
- Export semantics documented for CSV and JSON.

---

# 13. Error model

Suggested conceptual categories (verify Nox exception design):
- `AtlasError`
- `SchemaError`
- `ColumnNotFoundError`
- `TypeConversionError`
- `MissingValueError`
- `CsvImportError`
- `JsonImportError`
- `ShapeError`
- `AggregationError`
- `SortError`
- `ResourceLimitError`
- `UnsupportedFeatureError`

Rules:
- Include safe column name, row index, expected/actual type when useful.
- Do not dump entire datasets into error messages.
- Preserve original `CsvError`, `JsonError`, `FsError` context where supported.
- Do not return empty tables on parse failures.
- Never silently skip invalid rows unless a documented permissive mode is explicitly enabled.
- Errors should be actionable and deterministic.

---

# 14. Resource limits and memory safety

The documented `nox.csv.parse` and `nox.fs.read_to_string` operate on complete in-memory content.

Therefore:
- **Atlas MVP is in-memory.** Never advertise true streaming or out-of-core processing.
- Document approximate memory amplification from raw text → parsed strings → typed cells.
- Provide configurable row, column, field-length and input-size limits when feasible.
- Reject excessively wide/ragged inputs rather than crashing.
- Avoid unnecessary full-table copies.
- Make mutating vs non-mutating operations explicit.
- Bound expression depth and nested JSON flattening.
- Benchmark sorting/grouping on 1k, 10k, 100k rows, subject to test machine limits.
- Do not claim gigabyte-scale capability without evidence.
- Avoid quadratic string concatenation in CSV/JSON export; reuse efficient stdlib writers.
- Do not introduce unsafe pointer tricks or native shims for speed without approval.

---

# 15. Security and privacy

- CSV formula injection protections as an explicit export mode.
- Treat CSV and JSON input as untrusted data; enforce limits.
- Avoid path traversal when file paths are supplied by untrusted callers; `nox.fs` does not sandbox.
- Never automatically overwrite existing data files.
- No code execution from filters, computed columns or JSON.
- No shell execution for parsing/conversion.
- No automatic telemetry or network upload.
- Do not log sensitive dataset contents by default.
- Avoid exposing entire row values in exceptions.
- Clearly document encoding and invalid-text behavior.
- Escape strings properly in JSON output.
- Do not assume data is safe for HTML/SQL/shell contexts merely because it came from Atlas.
- Optional SQL/Feather integration must parameterize SQL and live outside core.

---

# 16. Optional integrations

Atlas must compile and work independently.

- **Feather:** optional adapter to convert query results to tables and vice versa; never make DB drivers mandatory.
- **Noxen:** optional schema/validation adapter; do not duplicate the validation framework.
- **Rune:** optional pretty terminal table printing; core table formatting should not depend on Rune.
- **Nyx/Aether:** optional CSV/JSON API response examples; no web imports in core.
- **Aura:** optional data-grid adapter; no UI imports in core.
- **Chronos:** optional scheduled ETL example; no scheduler dependency.
- **Nova:** optional HTTP-fetched CSV example; no HTTP dependency.

Any adapter needs its own tests and clear compatibility/version constraints.

---

# 17. Public API governance

Every Atlas release must define a deliberate public API.

**Artifacts:**
- `docs/api.md` — official public API reference.
- `docs/compatibility.md` — Nox version/backend/OS support.
- `docs/migration-guide.md` — breaking changes.
- `CHANGELOG.md` — release history.
- `docs/capabilities.md` — verified stdlib and language features.

**Stability levels:**
- Stable — backward compatible within major version.
- Experimental — may change with release notes.
- Deprecated — migration path and removal policy.
- Internal — no compatibility guarantee.

**Versioning:** SemVer. Document breaking changes even before 1.0. Add compile-time API contract tests using old examples. Avoid exporting internal storage representation as public types unless essential.

---

# 18. Testing requirements

Use verified Nox tooling:
- `noxc version`
- `noxc check <file.nox>`
- `noxc run <file.nox>`
- `noxc build <file.nox>`
- `noxc test`
- `noxc fmt <file.nox>` (**modifies source in place**)

## 18.1 CSV tests

- Empty input, header-only, one row, many rows.
- Quoted commas, escaped quotes, multiline quoted fields.
- LF and CRLF.
- Ragged rows, duplicate headers, empty headers.
- Missing/extra fields and strict/permissive policies.
- Type inference and explicit schema.
- Leading-zero identifiers, invalid numbers.
- Empty vs missing vs whitespace.
- CSV parse/dump round trips.
- Spreadsheet formula injection safe-export mode.
- File overwrite protections.

## 18.2 JSON tests

- Array of objects, array of arrays with schema.
- Empty arrays, null values, missing keys.
- Boolean vs string, numeric vs string.
- Nested objects/arrays with explicit policy.
- JSON number precision >2^53.
- Invalid JSON and unsupported values.
- JSON↔table↔CSV type-loss documentation tests.
- Deterministic schema/field order.

## 18.3 Table operation tests

- Select/drop/rename.
- Unknown and duplicate columns.
- Row/column shape invariants.
- Filter precedence and missing comparisons.
- Stable sort and multi-key sorting.
- Numeric vs lexical sort.
- Group-by with duplicate and missing keys.
- Empty inputs, all-missing groups.
- No mutation of source by pure transforms.

## 18.4 Statistics tests

- Mean, sum, min, max, count.
- Median odd/even, empty and one-element.
- Population/sample variance and stddev.
- Negative and mixed magnitude values.
- Cancellation-sensitive datasets.
- NaN/Infinity and all-missing columns.
- Integer overflow and float tolerance.
- Group aggregates match standalone stats.

## 18.5 Compatibility tests

- LLVM/QBE backends where supported.
- Windows/macOS/Linux where claimed.
- Hosted/freestanding profiles only where capability requirements allow.
- Verify compiler and stdlib minimum version.
- Compile README examples in CI.
- No external DB or network required for core tests.

## 18.6 Test discipline

- Every bug fix adds a regression test.
- Never weaken or delete tests just to get green CI.
- Never claim tests ran unless actually executed.
- Use deterministic fixtures and explicit numeric tolerances.
- Snapshot tests must not normalize away real defects.
- Keep large-data performance tests separate from fast unit tests.

---

# 19. Benchmarks and performance budgets

Compare:
- `nox.csv.parse` vs Atlas CSV import overhead.
- Direct Nox loops vs Atlas filter/group/stat operations.
- Row-oriented vs column-oriented representation if considered.
- 1k / 10k / 100k rows for import, filter, sort, group-by and export.
- Memory use and allocation counts where measurement is available.
- Compiler version, OS, backend, optimization and dataset shape.

Rules:
- No fabricated benchmark numbers.
- No unsupported claims of "faster than Pandas".
- Track both throughput and correctness.
- Benchmark wide vs narrow tables, high vs low cardinality groups.
- Measure sorting O(n log n) and group-by behavior.
- Do not trade numeric correctness or safety for speed.

---

# 20. Documentation requirements

`README.md` must include:
- purpose, audience and non-goals;
- Nox minimum version and verified backends;
- installation via real `nox.json`;
- a **compiling** CSV import example;
- selection/filter/sort example;
- group-by/statistics example;
- missing-values example;
- JSON conversion example;
- CSV export and formula-injection caveat;
- schema/type inference rules;
- memory and precision limitations;
- public API support matrix;
- license and contribution guidance.

Additional docs:
- `data-model.md` — cells, schema, ownership, NULL/NaN.
- `csv.md` — header, quoting, parsing, inference, encoding, export.
- `json.md` — type mapping, precision, nested data.
- `statistics.md` — exact variance/stddev/median definitions.
- `performance.md` — complexity and benchmarks.
- `security.md` — untrusted data, file paths, formula injection.
- `compatibility.md` — platform/backend matrix.
- `roadmap.md` — verified release gates.

**Never put unverified Python-like pseudocode in executable code fences without a clear warning.**

---

# 21. Development roadmap and release gates

## Phase 0 — Discovery
- Audit Nox stdlib and type system.
- Verify CSV/JSON parsing and filesystem behavior.
- Choose table/cell representation.
- Compile a minimal schema + table example.
- Write capabilities and architecture docs.
**Gate:** no core design depends on fictional APIs.

## Phase 1 — Atlas 0.1: table + CSV
- Explicit schema, rows, cells and missing values.
- CSV parse/import/export using `nox.csv`.
- Basic select/drop/rename.
- Strict shape and type checks.
- SQLite/DB not required.
**Gate:** real CSV→table→CSV examples compile and pass tests.

## Phase 2 — Atlas 0.2: transformations
- Filtering expressions and comparisons.
- Stable sorting, multi-key sorting.
- Missing-value fill/drop/count.
- Deterministic API and tests.
**Gate:** type/null semantics are consistent across operations.

## Phase 3 — Atlas 0.3: statistics
- Count/sum/min/max/mean.
- Median, sample/population variance/stddev.
- Numerically stable algorithms.
- Empty/NaN policies and benchmarks.
**Gate:** results match trusted reference cases.

## Phase 4 — Atlas 0.4: grouping
- Group-by one/multiple keys.
- Aggregation framework.
- Deterministic output and missing-key policy.
- Grouped stats.
**Gate:** no incorrect grouping, hidden coercion or unstable ordering.

## Phase 5 — Atlas 0.5: JSON and interoperability
- JSON array-of-objects import/export.
- JSON↔CSV conversion.
- Precision/type-loss policies.
- Optional examples with Feather/Rune/Noxen.
**Gate:** all round-trip limitations documented and tested.

## Phase 6 — Atlas 0.6+: advanced features (only if justified)
- Limited joins and concatenation.
- Additional quantiles and custom aggregations.
- Optional semicolon/TSV parser.
- Optional streaming **only after real Nox I/O primitives are verified**.
**Gate:** new features do not compromise the lightweight core.

## Phase 7 — Atlas 1.0
- Stable public API and SemVer policy.
- Complete docs and examples.
- Cross-platform/backend tests for claimed support.
- Performance baseline and security review.
- Release only with explicit user authorization.
**Gate:** all advertised features work and limitations are honest.

---

# 22. Agent execution protocol

For each task:

1. Restate scope and acceptance criteria.
2. Read relevant Atlas files, tests and Nox docs/source.
3. Check if `nox.csv`/`nox.json` already provide the primitive.
4. Verify uncertain language features with minimal compiling examples.
5. Describe API shape, schema/null semantics and complexity.
6. Implement the smallest coherent vertical slice.
7. Add unit tests and regression tests.
8. Run formatter, compiler checks, tests and examples.
9. Review precision, missing data, input limits, export safety and file overwrite behavior.
10. Update docs, API reference, capability matrix and changelog.
11. Report exact commands executed, changed files, actual pass/fail results and limitations.

When blocked:
- do not invent Nox features or stdlib functions;
- provide a minimal failing reproduction;
- document the verified limitation;
- implement a reduced, correct fallback;
- request permission before compiler/runtime/native dependency changes.

---

# 23. Definition of Done

- [ ] Compiles with the verified Nox toolchain.
- [ ] Uses actual Nox syntax and stdlib APIs.
- [ ] Schema, row shape and column types remain valid.
- [ ] Missing/NULL/NaN semantics are explicit.
- [ ] CSV/JSON conversions preserve documented invariants.
- [ ] Numerical results match documented formulas/tolerances.
- [ ] Sorting/grouping/filtering are deterministic as promised.
- [ ] Untrusted input limits and export risks are addressed.
- [ ] No mandatory framework or native dependencies.
- [ ] Public API is documented and contract-tested.
- [ ] Examples compile and automated tests run.
- [ ] No unsupported streaming/precision/platform claims.
- [ ] No skipped/hidden failing tests.
- [ ] Final report lists verification and unresolved gaps.

---

# 24. First autonomous assignment

**Build a real, small Atlas MVP — not an imaginary Pandas clone.**

1. Audit the current Nox compiler and `nox.csv`, `nox.json`, `nox.fs`, `nox.mathx`, `nox.test`.
2. Record the version, actual APIs, supported types and capability limitations.
3. Choose and document explicit schema + cell storage.
4. Scaffold a standalone `atlas` package with no Nyx/Aether/Aura dependencies.
5. Implement CSV text import/export using verified `nox.csv` functions.
6. Implement schema validation and a simple missing-value model.
7. Implement select/filter/sort on a small typed table.
8. Implement count/sum/mean with defined empty/missing policies.
9. Add tests for quoted CSV, ragged rows, missing cells and numeric conversion.
10. Compile/run real Nox examples and tests.
11. Publish an evidence-based backlog for group-by, median/stddev, JSON conversion, joins and optional streaming.
12. Do not modify Nox compiler/stdlib or publish releases without user authorization.

**Quality bar:** a correct, predictable, testable Nox-native CSV/table toolkit is more valuable than a large API with silent type loss or unverified features.
