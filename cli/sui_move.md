# Sui Move CLI Cheatsheet

The `sui move` command provides tools for creating, building, testing, and managing Move packages on Sui. Run commands from the root of your Move package directory (containing `Move.toml`).

**Latest reference**: [Sui Move CLI Documentation](https://docs.sui.io/references/cli/move)

### Table of Contents

- [Sui Move CLI Cheatsheet](#sui-move-cli-cheatsheet)
  - [1. Basic Commands](#1-basic-commands)
  - [2. Project Creation and Management](#2-project-creation-and-management)
  - [3. Building and Compilation](#3-building-and-compilation)
  - [4. Testing and Coverage](#4-testing-and-coverage)
  - [5. Inspection and Utilities](#5-inspection-and-utilities)
  - [6. Common Global Options](#6-common-global-options)
  - [Useful Tips](#useful-tips)
  - [Official Resources](#official-resources)

## 1. Basic Commands

| Command                        | Description                                      | Example |
|--------------------------------|--------------------------------------------------|---------|
| `sui move --version`           | Show Sui CLI version                             | `sui move --version` |
| `sui move --help`              | Display general help and available subcommands   | `sui move --help` |
| `sui move <subcommand> --help` | Show help for a specific subcommand              | `sui move build --help` |

## 2. Project Creation and Management

| Command                              | Description                                      | Example |
|--------------------------------------|--------------------------------------------------|---------|
| `sui move new <name>`                | Create a new Move package                        | `sui move new my_project` |
| `sui move update-deps`               | Re-pin dependencies in `Move.toml` and `Move.lock` | `sui move update-deps` |
| `sui move migrate`                   | Migrate package to Move 2024 edition             | `sui move migrate` |
| `sui move migrate <path>`            | Migrate package at specific path                 | `sui move migrate ./my_project` |

## 3. Building and Compilation

| Command                              | Description                                      | Example |
|--------------------------------------|--------------------------------------------------|---------|
| `sui move build`                     | Build the Move package                           | `sui move build` |
| `sui move build --path <dir>`        | Build package at specified path                  | `sui move build --path ./my_project` |
| `sui move build --force`             | Force full recompilation                         | `sui move build --force` |
| `sui move build --dump-bytecode-as-base64` | Dump compiled bytecode as base64 (for publishing) | `sui move build --dump-bytecode-as-base64` |
| `sui move build --disassemble`       | Generate disassembly output                      | `sui move build --disassemble` |

**Common build options**: `--test`, `--doc`, `--quiet`, `--no-lint`, `--warnings-are-errors`

## 4. Testing and Coverage

| Command                              | Description                                      | Example |
|--------------------------------------|--------------------------------------------------|---------|
| `sui move test`                      | Run unit tests                                   | `sui move test` |
| `sui move test --coverage`           | Run tests with coverage instrumentation         | `sui move test --coverage` |
| `sui move test --trace`              | Generate execution trace for debugger            | `sui move test --trace` |
| `sui move coverage summary`          | Show test coverage summary                       | `sui move coverage summary` |
| `sui move coverage summary --test`   | Detailed coverage report                         | `sui move coverage summary --test` |

## 5. Inspection and Utilities

| Command                              | Description                                      | Example |
|--------------------------------------|--------------------------------------------------|---------|
| `sui move summary`                   | Generate serialized summary of the package (functions, structs, etc.) | `sui move summary` |
| `sui move disassemble`               | Disassemble bytecode                             | `sui move disassemble` |
| `sui move coverage`                  | Inspect test coverage (after running with `--coverage`) | `sui move coverage` |

## 6. Common Global Options

- `-p, --path <PACKAGE_PATH>` — Specify package directory
- `--test` — Compile in test mode (include `tests/` directory)
- `--doc` — Generate documentation
- `--force` — Force rebuild
- `--quiet` (`-q`) — Reduce output
- `--lint` / `--no-lint` — Control linters
- `--default-move-edition 2024` — Set Move edition

## Useful Tips

- **Project Structure** (created by `new`):
    ```
    Move.toml
    sources/
    tests/
    ```
- Run `sui move build` before testing or publishing.
- Use `--dump-bytecode-as-base64` when preparing for `sui client publish`.
- Publishing example:
    ```bash
    sui client publish --gas-budget 100000000
    ```
- Dependencies are managed via Move.toml and automatically cached in ~/.move.
- For full options on any command: sui move <command> --help

## Official Resources:

- Sui Move CLI: https://docs.sui.io/references/cli/move
- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet
- Move on Sui: https://docs.sui.io/develop/write-move

For the most up-to-date information, run sui move --help locally.

---

*Previous Page: [`Sui CLI Basic Summary`](cli_summary.md)*