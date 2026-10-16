# chess-engine-c++

A **UCI chess engine** in C++, based on [Stockfish](https://github.com/official-stockfish/Stockfish). It analyzes positions, searches deeply with alpha–beta and NNUE evaluation, and speaks the Universal Chess Interface (UCI) protocol for use with external GUIs.

## Overview

This repository contains the engine source, build system, CI workflows, and test harness inherited from the Stockfish project. The binary is intended for analysis and bot integration—not as a standalone chess application. You run the engine from the command line or connect it to a UCI-compatible GUI (Arena, Cutechess, Lichess bots, etc.).

The evaluation uses embedded NNUE networks; build scripts can fetch network files when needed. Platform-specific slices (x86 AVX tiers, ARM, universal binaries) are supported via `src/Makefile` and helper scripts under `scripts/`.

## Features

- Full UCI command loop (`position`, `go`, `setoption`, `bench`, `perft`, and related commands)
- NNUE-based evaluation with tunable search options (threads, hash, Syzygy tablebases, Chess960)
- Multi-threaded search with NUMA-aware thread placement
- Syzygy endgame tablebase probing (when `.rtbw` / `.rtbz` files are configured)
- Built-in benchmark and perft utilities for correctness and performance checks
- Shell and Python tests under `tests/` (signature, perft, reproducible search, instrumented runs)
- GitHub Actions workflows for multi-platform compilation and sanitizers

## Tech stack

| Area | Technology |
|------|------------|
| Language | C++17 (GCC 9.3+, Clang 11+, or MSVC as supported by Makefile) |
| Build | GNU Make (`src/Makefile`) |
| Tests | Python 3, Bash, Expect (for some shell tests) |
| CI | GitHub Actions |
| Upstream | Stockfish engine architecture and GPL-licensed code |

## Prerequisites

- **C++ toolchain**: GCC, Clang, or MSVC matching Stockfish requirements (see `src/types.h` for minimum compiler versions)
- **Make**: GNU Make on Linux/macOS; MSYS2/MinGW or Visual Studio build paths on Windows (see Makefile help)
- **Python 3**: For `tests/testing.py` and `tests/instrumented.py` (optional `requests` for Syzygy fixture download)
- **Expect** (optional): For `tests/perft.sh` and `tests/reprosearch.sh` on Unix-like systems

## Getting started

```bash
git clone https://github.com/vonmiller1/chess-engine-c-.git
cd chess-engine-c-
```

No root-level dependency manifest is required; the engine builds from `src/` with Make.

To download default NNUE weights before or after building (from the repo root):

```bash
sh scripts/net.sh
```

## Running locally

Build the engine (from `src/`):

```bash
cd src
make help          # list targets and architecture flags
make -j build      # native optimized binary (name varies by platform, e.g. stockfish)
```

Run in UCI mode (stdin/stdout):

```bash
./stockfish
```

Example interaction:

```
uci
isready
position startpos
go depth 10
quit
```

One-shot CLI commands are also supported, e.g. `./stockfish bench` or `./stockfish compiler`.

There is no HTTP server or default port; communication is process stdin/stdout only.

## Testing

Tests are **not** run as part of the optimization workflow documented elsewhere; when you choose to run them locally:

```bash
cd src && make -j build
cd ../tests
python testing.py          # framework entry (see file for test classes)
python instrumented.py --help
./signature.sh             # requires built stockfish in PATH or EXE env
./perft.sh                 # requires expect
./reprosearch.sh           # requires expect
```

Platform validation scripts live under `scripts/` (e.g. `check_universal.sh` for SDE-based universal binary checks).

## Project structure

| Path | Purpose |
|------|---------|
| `src/` | Engine source, `Makefile`, NNUE and Syzygy code |
| `src/universal/` | Multi-architecture universal binary entry points |
| `scripts/` | NNUE download, native ARCH detection, universal-binary verification |
| `tests/` | Python test framework and shell regression scripts |
| `.github/workflows/` | CI: tests, sanitizers, releases, cross-compilation |

## Configuration

Engine options are set at runtime via UCI `setoption` (documented by the `uci` command). Common options include `Threads`, `Hash`, `SyzygyPath`, `UCI_Chess960`, and `Debug Log File`. Do not commit secrets or personal paths into the repository; point `SyzygyPath` at your local tablebase directory.

Build-time knobs are passed through `make` variables (see `make help` in `src/`): `ARCH`, `COMP`, `CXXFLAGS`, etc.

## Deployment

CI workflows under `.github/workflows/` build and test on push; release-oriented workflows mirror upstream Stockfish automation where applicable. For personal forks, push to your GitHub remote and rely on Actions or copy the artifact from workflow runs. This project does not ship a container image in-tree.

## Contributing

1. Fork [vonmiller1/chess-engine-c-](https://github.com/vonmiller1/chess-engine-c-).
2. Create a feature branch from `main`.
3. Keep commits focused; match existing C++ and Makefile style.
4. Open a pull request against `main`.

Engine move-quality debates belong in community channels, not issue trackers (see issue templates).

## License

Stockfish and this derivative are **free software** under the **GNU General Public License v3** (or later). See upstream [Stockfish licensing](https://github.com/official-stockfish/Stockfish/blob/master/Copying.txt) and `src/incbin/UNLICENCE` for bundled third-party components. When distributing binaries, comply with GPL source-offer requirements.

## Upstream

Functional changes that belong in the main project should eventually be proposed to [official-stockfish/Stockfish](https://github.com/official-stockfish/Stockfish). This fork tracks that codebase for learning, CI, and customization.
