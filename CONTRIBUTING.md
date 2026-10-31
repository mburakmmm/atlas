# Contributing to Atlas

Thank you for contributing to Atlas!

## Development Guidelines

1. **Pure Nox**: All library logic must be written in standard, compiling Nox code without mandatory C or foreign ABI dependencies.
2. **AGENTS.md Contract**: Read and strictly adhere to `AGENTS.md`.
3. **Automated Testing**: Every new feature or bugfix must be covered by automated tests in `tests/` and pass with `noxc test`.
4. **Code Formatting**: Format all `.nox` files using `noxc fmt <file.nox>`.
5. **Numerical Stability**: When implementing statistical algorithms, prefer stable algorithms (such as Welford's algorithm) over naive formulas.
