# Security Policy

## Supported Versions

Atlas follows semantic versioning and regularly updates security guarantees across active minor releases.

| Version | Supported          |
| ------- | ------------------ |
| 0.9.x   | :white_check_mark: |
| 0.8.x   | :white_check_mark: |
| < 0.8.0 | :x:                |

## Reporting a Vulnerability

If you discover a security vulnerability or critical integrity flaw within Atlas, please do not open a public issue. Instead, report it responsibly via:

- **Email**: Send detailed vulnerability reports to [burakmemis@gmail.com](mailto:burakmemis@gmail.com).
- **GitHub Private Advisory**: Open a private vulnerability report via the **Security** tab of the Atlas repository.

Please include:
1. Detailed description of the vulnerability.
2. Minimal reproducible example or PoC in pure Nox.
3. Affected versions and target platforms (macOS, Linux, Windows).
4. Potential security impact (e.g. arbitrary file overwrite, formula injection, unbounded memory allocation).

Maintainers will acknowledge receipt within **48 hours** and provide regular progress updates toward a patched release.

## Security Architecture & Best Practices

1. **Spreadsheet Formula Injection Prevention (`safe_spreadsheet=True`)**:
   - When generating CSV/TSV output intended to be opened in desktop spreadsheet software (Excel, LibreOffice, Google Sheets), values starting with `=`, `+`, `-`, or `@` can trigger remote code execution or formula evaluation.
   - Always export with `to_csv(table, safe_spreadsheet=True)` or `write_csv_file(table, path, safe_spreadsheet=True)`, which prefixes dangerous formula symbols with a single quote (`'`).

2. **Filesystem Overwrite Protection**:
   - `write_csv_file`, `write_json_file`, and `write_tsv_file` check `nox.fs.exists(path)` before modifying any file.
   - Unless `overwrite=True` is explicitly passed by the caller, Atlas raises an `AtlasError` preventing accidental data destruction.

3. **Checked Arithmetic & Numeric Overflow**:
   - To guard against silent integer wraparound vulnerabilities, Atlas uses `checked_*_int64` throughout summation and aggregation pathways, enforcing 64-bit bounds and raising actionable `ShapeError` exceptions.

4. **Resource Bounds & DoS Prevention**:
   - `StreamingCsvReader` enforces `max_record_bytes` (default 10 MB, configurable via `CsvOptions`) measured in UTF-8 bytes to prevent unbounded heap allocation from malformed, unclosed quotes or hostile CSV streams.
