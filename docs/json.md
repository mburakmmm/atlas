# JSON ↔ Table ↔ CSV Conversion

Atlas provides direct bidirectional conversion between tabular data and JSON array-of-objects representations:

## 1. Supported Shape

The standard input format is a JSON array of flat objects:
```json
[
  {"id": 1, "name": "Alice", "score": 95.5, "active": true},
  {"id": 2, "name": "Bob", "score": 88.0, "active": false}
]
```

## 2. Ingestion (`table_from_json`)

- Discovers schema from union of all keys across objects in the array.
- Respects JSON types:
  - `null` -> `cell_null()`
  - `boolean` -> `cell_bool()`
  - `number` -> `cell_int()` if integer value, otherwise `cell_float()`
  - `string` -> `cell_str()`
- Rejects nested structures (nested objects or nested arrays) with `JsonImportError`.

## 3. Serialization (`table_to_json`)

Uses Nox stdlib `JsonWriter` to stream parts and build the JSON string in $O(N)$ time via a single buffer concatenation.
Missing values are written as JSON `null`.
