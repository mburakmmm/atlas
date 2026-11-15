# Compatibility & System Requirements

## 1. Compiler & Platform Requirements

- **Nox Compiler**: `noxc >= 2.1.186` (verified on `noxc 2.1.186`).
- **Operating Systems**: macOS (Darwin x86_64 / arm64), Linux (x86_64 / aarch64), Windows (x86_64).
- **Backends**: LLVM (`--backend llvm`) and QBE (`--backend qbe`), verified continuously in CI.
- **Profiles**:
  - **Core (`atlas.nox`)**: 100% pure-Zig/Nox stdlib (`nox.csv`, `nox.json`, `nox.mathx`, `nox.decimal`, `nox.time`, `nox.timezone`, `nox.regex`, `nox.strings`, `nox.collections`, `nox.fs`, `nox.path`, `nox.gzip`, `nox.zstd.decompress`, `nox.zip`, `nox.tar`, `nox.thread`, `nox.async`). Never links `noxclibs.a`.
  - **Desktop Turbo (`atlas_fast.nox`, Opt-in)**: Uses `noxclibs` (`nox.yyjson`, `nox.xxhash`, `nox.pcre2`, `nox.lz4`, `nox.zstd.compress`, `nox.brotli`) on supported desktop targets, with runtime `*_is_supported()` capability probes.

## 2. Dependencies

Atlas has **zero external third-party package dependencies** (`"requires": []` in `nox.json`). It relies exclusively on the verified Nox standard library.
