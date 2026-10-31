# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |

## Reporting a Vulnerability

If you discover a security issue or vulnerability in Atlas, please report it responsibly by contacting the maintainers directly or opening a private advisory.

## Security Practices
- Always enable `safe_spreadsheet=True` when generating CSV files that might be opened in spreadsheet programs by untrusted users.
- Use explicit `overwrite=True` flags for intentional file modifications.
