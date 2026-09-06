# Agent Guidelines for HIPIFY

HIPIFY is AMD's source-to-source translator that converts CUDA code into portable HIP C++. It consists of:

- `hipify-clang` — clang-based tool for precise, syntax-driven translation.
- `hipify-perl` — fallback Perl script for regex-based translation.
- `bin/` — helper scripts for in-place conversion and codebase analysis.
- `docs/` — Sphinx/ROCm-flavored documentation.

## Project Layout

```
src/               # hipify-clang C++ source
bin/               # utility shell/perl scripts
packaging/         # packaging metadata
docs/              # Sphinx documentation
CMakeLists.txt     # main build file
```

## Critical Rules

1. **Preserve test files.** Any change that touches `tests/` must include a lit/FileCheck regression test under `tests/unit/` or `tests/functional/`.
2. **Match existing style.** C++ follows LLVM/Clang coding conventions (camelCase, no tabs, 80-column soft limit).
3. **Document user-facing behavior.** If a new CUDA API is now supported, update `docs/reference/supported_apis.md` or regenerate docs with `hipify-clang --md --doc-format=full --doc-roc=joint`.

## Build Commands

Prerequisites: LLVM + Clang built from source, CUDA toolkit 7.0+, CMake 3.16.8+.

```bash
# Configure (point to LLVM/Clang install)
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=$LLVM_DIST       -DCMAKE_BUILD_TYPE=Release       -DHIPIFY_CLANG_TESTS=ON       ..

# Build
make -j$(nproc)

# Install
make install
```

## Test Commands

```bash
cd build

# Run lit/FileCheck tests (requires HIPIFY_CLANG_TESTS=ON and lit installed)
ninja test
# or
llvm-lit -v test

# Run only hipify-clang functional tests
llvm-lit -v tests/functional

# Validate a single CUDA file translation
./hipify-clang --cuda-path=/usr/local/cuda     ../tests/functional/unit_tests/synthetic_driver_functions.cu     -- -I/usr/local/cuda/include
```

## Lint / Format

```bash
# Markdown lint (matches CI)
markdownlint --config .mdlrc '**/*.md' --ignore README.md

# C++ formatting (project uses LLVM style)
clang-format -i src/*.cpp src/*.h
```

## Common Operations

```bash
# Translate a single file
./hipify-clang input.cu --cuda-path=/usr/local/cuda -o output.hip.cpp

# In-place conversion of a whole tree
../bin/hipconvertinplace.sh /path/to/cuda/sources

# Regenerate supported-API docs
./hipify-clang --md --doc-format=full --doc-roc=joint
```

## Gotchas

- `hipify-clang` links against the installed LLVM/Clang headers; always build LLVM first.
- The `--cuda-path` argument must point to a valid CUDA toolkit; missing headers cause silent or misleading parse errors.
- Regenerated docs (`docs/reference/supported_apis.md`) are committed; remember to include them in the same PR when support tables change.
- Perl fallback (`hipify-perl`) is maintained for environments without clang; keep regex maps in sync with C++ mappings when adding APIs.
