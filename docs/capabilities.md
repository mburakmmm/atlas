# Nox Standard Library Capabilities & Verification Audit

Audit performed against `noxc 2.0.86` on macOS (darwin).

## 1. Verified Stdlib Primitives

### `nox.csv`
- `parse(text: str) -> list[list[str]]`: RFC 4180 parsing supporting commas, quotes, escaped quotes (`""`), and CRLF/LF line endings. Operates strictly in-memory on strings.
- `dump(rows: list[list[str]]) -> str`: Generates standard CSV text with quotes applied where required.
- **Limitation**: Custom delimiters (semicolon, tab) and streaming file reads are out of scope for stdlib. Atlas respects this by providing UTF-8 in-memory text parsing.

### `nox.json`
- `parse(text: str) -> JsonValue`: Full recursive JSON parsing.
- Accessors: `is_null`, `is_bool`, `is_number`, `is_string`, `is_array`, `is_object`, `as_bool`, `as_number`, `as_string`, `array_len`, `array_get`, `object_len`, `object_key`, `object_value`.
- `JsonWriter`: Incremental memory-buffered JSON construction with $O(N)$ concatenation via `nox.strings.join`.

### `nox.mathx`
- Pure math functions: `sqrt`, `pow`, `floor`, `ceil`, `min`, `max`, `abs`. Fully qualified calls (`nox.mathx.sqrt`).

### `nox.fs`
- `read_to_string(path: str) -> str`: Reads entire file into memory.
- `write_string(path: str, content: str) -> None`: Overwrites destination.
- `exists(path: str) -> bool`: Checks path existence.
- **Limitation**: `nox.fs` does not perform path sandboxing. Atlas checks `exists` before writing unless `overwrite=True` is explicitly passed.

### `nox.test`
- Assertion framework: `assert_eq_int`, `assert_eq_str`, `assert_eq_float`, `assert_true`.

## 2. Language Primitives Verified

- Tagged records and custom classes with typed constructors and methods.
- Optional / nullable types: `T | None` with explicit narrowing `if val != None:`.
- Slices: `list` indexing and slices, string methods (`startswith`, `contains`, `replace`).
- Native `list.sort()` for primitive lists.
- Integer division: `//`.
- Exception hierarchy deriving from `Exception`.

## 3. Streaming CSV & Memory Bounds Verified

- `nox.fs.open_read(path)` and `reader.read_chunk(bytes)`: Incremental file streaming without loading the entire document into RAM.
- `StreamingCsvReader`: Persistent RFC 4180 state machine processing chunks in $O(N)$ linear time without re-scanning buffers.
- **Realistic Streaming Memory Bound**: Memory consumption during streaming is bounded by $O(B + R \times S + L)$, where:
  - $B$ is `chunk_bytes` (the read buffer size),
  - $R$ is `chunk_rows` (the maximum yielded rows per chunk),
  - $S$ is average row size,
  - $L \le \text{max\_record\_bytes}$ is the maximum allowable single record length (default 10 MB, strictly measured in UTF-8 bytes via `nox.strings.byte_len`).
  - Parsing pauses immediately when $R$ rows are accumulated, avoiding unbounded `pending_rows` buffering even when large read chunks are provided.
- **Streaming Schema Policy**:
  - *Inference (Default)*: When `explicit_schema` is omitted, the table schema is inferred from the first chunk's data rows using `infer_schema`. Subsequent chunks are strictly validated against this schema (`ShapeError` on column count mismatch, `MissingValueError` on non-nullable missing cells, and strict cell parsing).
  - *Explicit Schema*: For datasets where later rows may contain wider types (e.g. strings in predominantly numeric columns), passing an `explicit_schema` enforces predefined types and nullability across all chunks.
- Online Welford / Chan algorithm: Numerically stable running mean, variance, and standard deviation with $O(1)$ memory usage; raises `AggregationError` when observations are insufficient.
- `try...finally`: Guaranteed file descriptor closure and resource cleanup.
