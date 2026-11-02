# Contributing to Atlas 🧭

Thank you for your interest in contributing to **Atlas**! Atlas is a pure-Nox tabular data processing and statistics library designed to make data cleaning, transformation, and analytical aggregation fast, predictable, and memory-safe.

---

## 📜 Guiding Engineering Contract (`AGENTS.md`)

All contributions to Atlas are governed by the binding engineering rules set forth in [`AGENTS.md`](AGENTS.md). Before writing or modifying any code, review these core principles:

1. **Pure Nox**: All library logic must be written in standard, compiling Nox code without mandatory C or foreign ABI dependencies.
2. **Deterministic & Cross-Platform**: Code must run identically on macOS, Linux, and Windows across both LLVM and pure-math backends.
3. **Explicit Data Semantics**: Missing/NULL values (`Cell(TYPE_NULL)`) are distinct from empty strings, zeros, `False`, or `NaN`.
4. **Checked Arithmetic**: 64-bit integer calculations must be bounded using `checked_add_int64`, `checked_sub_int64`, `checked_mul_int64`, and `checked_div_int64` (`ShapeError` on overflow).
5. **Numerical Stability**: Statistical operations must prioritize numerical accuracy (e.g. Welford's algorithm for variance/stddev, Neumaier compensated summation for floating-point sums).
6. **Cross-Platform File I/O**: Never use Unix-specific absolute paths (`/tmp/...`) in tests or library code. Always use relative paths and ensure file handles are closed in `finally:` blocks.

---

## 🛠️ Development Setup

### 1. Prerequisites
- **Nox Compiler**: Ensure `noxc` is installed and available in your `PATH`.
  - **Linux / macOS**:
    ```bash
    curl -fsSL https://noxlang.com/install.sh | bash
    ```
  - **Windows**:
    ```powershell
    irm https://noxlang.com/install.ps1 | iex
    ```
- Verify toolchain:
  ```bash
  noxc version
  ```

### 2. Clone the Repository
```bash
git clone https://github.com/mburakmmm/atlas.git
cd atlas
```

### 3. Verify Local Build & Tests
Run the complete automated test suite (all 39 test suites):
```bash
noxc test
```

Typecheck the public library facade and entrypoint:
```bash
noxc check atlas.nox
noxc check main.nox
```

Run the window function performance benchmark:
```bash
noxc run benchmarks/benchmark_window.nox
```

---

## 🧪 Testing Guidelines

1. **Test Every Public Feature**: No feature or bug fix may be merged without comprehensive unit tests in `tests/`.
2. **Regression Tests**: Every bug report must include a regression test that fails before the fix and passes after.
3. **Reference Verification**: When introducing analytical transforms or aggregates, add ground-truth verification comparing outputs against Pandas 2.x, Polars 1.x, or SQL standards in `tests/reference_comparison_test.nox`.
4. **Cross-Platform Safety**: Always wrap temporary file operations in `try...finally: if nox.fs.exists(path): nox.fs.remove_file(path)` and close stream readers before deletion.

---

## 📝 Commit Conventions & Pull Request Workflow

We follow standard [Conventional Commits](https://www.conventionalcommits.org/):

- `feat(...)`: New analytical feature or transform (e.g. `feat(window): add rolling median`)
- `fix(...)`: Bug fix or numerical correction (e.g. `fix(io): cross-platform relative path support on Windows`)
- `perf(...)`: Performance optimization or benchmark expansion
- `test(...)`: Adding or updating test suites
- `docs(...)`: Documentation updates in `docs/` or `README.md`
- `chore(...)`: Tooling, CI, or dependency updates

### Pull Request Checklist
- [ ] Code compiles and typechecks with `noxc check atlas.nox`.
- [ ] All 39+ test suites pass with `noxc test`.
- [ ] Cross-platform CI passes on Ubuntu, macOS, and Windows.
- [ ] `CHANGELOG.md` updated following the [Keep a Changelog](https://keepachangelog.com/) format.
- [ ] Relevant documentation in `docs/` and `README.md` updated.

---

## 🤝 Code of Conduct

All contributors are expected to uphold the [Code of Conduct](CODE_OF_CONDUCT.md). Please report unacceptable behavior to [burakmemis@gmail.com](mailto:burakmemis@gmail.com).
