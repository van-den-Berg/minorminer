Guidance for AI coding agents working on minorminer

This repository contains a C++/Python hybrid research/archive implementation
of heuristic minor-embedding tools (Cython extensions + header-only C++).
Keep edits small and backwards-compatible: prefer adapters, tests, and
lightweight helpers rather than big refactors.


Quick orientation
- Primary Python entrypoint: `minorminer.find_embedding(...)` (wrapper in
  `minorminer/minorminer.py`) — inspect this file to learn the public API and
  the important parameters (timeout, threads, initial_chains, etc.).
- Cython extensions are declared in `setup.py`: `minorminer._minorminer`,
  `minorminer.busclique`, `minorminer.subgraph` (the latter pulls sources from
  `external/glasgow-subgraph-solver/gss`). Changes that affect performance or
  algorithms usually require editing `.pyx`/`.pxi` files and rebuilding the
  extension.
- C++ header-only usage: the `include/` directory exposes the C++ API. The
  top-level `CMakeLists.txt` creates an INTERFACE target and offers two
  options: `MINORMINER_BUILD_TESTS` and `MINORMINER_BUILD_EXAMPLES` (see
  `CMakeLists.txt`).

Build & test shortcuts (copyable)
- Python (from repo root):
  - Ensure submodules: `git submodule init && git submodule update`
  - Install deps: `pip install -r requirements.txt`
  - Build/install: `python setup.py install` (requires Cython, C++ toolchain)
  - Run tests: `pip install -r tests/requirements.txt && pytest .`
- C++ (header-only / examples):
  - `mkdir build && cd build && cmake .. && make`
  - Enable tests: `cmake -DMINORMINER_BUILD_TESTS=ON ..`

Packaging notes
- `pyproject.toml` contains `cibuildwheel` config used to build wheels; the
  project requires Cython at build time. `minorminer.__version__` is read by
  setuptools (see `pyproject.toml` and `minorminer/__init__.py`).

Repository-specific patterns & gotchas
- Platform-specific compile flags live in `setup.py` (see variables like
  `extra_compile_args`, `extra_compile_args_glasgow`). When editing native
  build behavior prefer adding small, well-scoped flags or helper functions
  rather than large global changes.
- The `subgraph` extension bundles third-party sources from
  `external/glasgow-subgraph-solver/gss`. If altering those files, prefer
  contributing small patches and keep the build invocation identical to avoid
  breaking ABI assumptions.
- There are no `.cu` / CUDA sources in the main tree. If GPU experiments are
  needed, add them under an `experimental/` path and keep CPU fallbacks.

Where to look for examples & tests
- Python example scripts: `examples/*.py` (e.g., `examples/fourcolor.py`).
- C++ usage example: `examples/example.cpp` and CMake example target.
- Tests are under `tests/` (both Python and C++ tests). Test-related
  requirements are in `tests/requirements.txt`.

Editing workflow recommendations for AI agents
- Small API changes: add a thin adapter in Python (e.g., in
  `minorminer/minorminer.py`) that translates new inputs to the existing
  Cython call. Provide unit tests in `tests/` demonstrating the new behavior.
- Native/Cython edits: update `.pyx`/`.pxi` files and then rebuild (run
  `python setup.py build_ext --inplace` or `python setup.py install`).
- Avoid touching `external/` sources unless necessary. If you do, keep the
  changes minimal and document why they're needed in a short note in the
  commit message.

If anything here is unclear or you'd like more detail (for example, a
step-by-step dev loop for faster Cython rebuilds, or pointers to the
most-used algorithmic routines in `include/find_embedding`), tell me which
section to expand and I'll iterate. 
