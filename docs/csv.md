# CSV Reading, Type Inference & Writing

## 1. RFC 4180 Parsing via `nox.csv`

Atlas uses the Nox standard library `nox.csv` parser for RFC 4180 parsing:
- Embedded commas within quotes (`"New York, NY"`) are handled correctly.
- Escaped quotes (`"Quote ""test"" here"`) parse to `Quote "test" here`.
- Multi-line quoted fields are preserved.
- Both LF (`\n`) and CRLF (`\r\n`) line endings are supported.

## 2. Type Inference

When `explicit_schema` is omitted, Atlas scans columns across all data rows:
1. If all non-empty values are valid integers: inferred as `TYPE_INT`.
2. Else if all non-empty values are valid floats: inferred as `TYPE_FLOAT`.
3. Else if all non-empty values are valid booleans (`true`/`false`): inferred as `TYPE_BOOL`.
4. Otherwise: inferred as `TYPE_STR`.

Missing value tokens (`""`, `"NA"`, `"null"`, `"NULL"`, `"NaN"`) parse to `Cell(TYPE_NULL)`.

## 3. Spreadsheet Formula Injection Mitigation

When CSV files are opened in Microsoft Excel, LibreOffice, or Google Sheets, cells starting with `=`, `+`, `-`, or `@` can trigger formula execution.

When `safe_spreadsheet=True` is passed to `to_csv()` or `write_csv_file()`:
- Cells whose string representation starts with `=`, `+`, `-`, or `@` are automatically prefixed with a single apostrophe (`'`).
- For machine-to-machine exchange, leave `safe_spreadsheet=False` (default) to avoid altering raw strings.
