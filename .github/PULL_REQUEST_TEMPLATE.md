## Summary
<!-- Briefly explain the motivation, context, and problem solved by this pull request. -->

## Changes Introduced
<!-- Bulleted list of architectural, API, or operational changes introduced. -->
- 

## Verification & Test Plan
<!-- Describe the tests and verification steps executed. All tests must pass cleanly. -->
- [ ] `noxc check atlas.nox main.nox` (all core modules typecheck cleanly)
- [ ] `noxc test` (all 39+ automated test suites pass)
- [ ] `noxc run benchmarks/benchmark_window.nox` (performance benchmarks meet SLAs)
- [ ] Added regression or feature unit tests in `tests/` for all new behavior

## Engineering Standards Checklist
- [ ] Adheres strictly to `AGENTS.md` engineering principles (pure Nox, no unverified C/C++ dependencies).
- [ ] 64-bit integer arithmetic protected with `checked_*_int64` against overflow and underflow (`ShapeError`).
- [ ] Floating-point operations preserve IEEE 754 precision without catastrophic cancellation.
- [ ] File I/O operations are strictly cross-platform (no hardcoded `/tmp`, relative test paths, file handles closed in `finally:` blocks).
- [ ] Updated `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/) format.
- [ ] Documentation updated in `docs/` (`capabilities.md`, `api.md`, `README.md`) if public API changed.
