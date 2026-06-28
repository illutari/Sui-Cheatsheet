# Sui Replay CLI Cheatsheet

The `sui replay` command replays one or more transactions locally against a network (or GraphQL endpoint). It verifies that the locally computed effects match the on-chain effects and can generate detailed execution traces for the Move debugger and trace analysis tools (e.g., gas profiling).

Replaying is extremely useful for:
- Debugging failed or suspicious transactions
- Generating traces for the Move Trace Debugger VSCode extension
- Analyzing gas usage (`sui analyze-trace`)
- Batch-replaying many transactions from a file

**Note:** The older `sui client replay-*` commands are deprecated in favor of the top-level `sui replay`.

## 1. Basic Usage

| Command | Description | Example |
|---------|-------------|---------|
| `sui replay --digest <DIGEST>` | Replay a single transaction by its digest | `sui replay --digest 5sbk5UYpQv1i84zkSBSWmowAmPTxw71U7Bb1gbiW5w4y` |
| `sui replay --digests-path <FILE>` | Replay multiple transactions listed one per line in a text file | `sui replay --digests-path tx_digests.txt` |

## 2. Common Options

| Flag | Description | Example |
|------|-------------|---------|
| `-d, --digest <DIGEST>` | Transaction digest to replay | `--digest <TX_DIGEST>` |
| `--digests-path <PATH>` | Path to file containing one digest per line (batch mode) | `--digests-path ./digests.txt` |
| `-o, --output-dir <DIR>` | Directory for replay artifacts (default: `./.replay/<digest>`) | `--output-dir ./replays` |
| `--trace` | Generate execution trace (saved under output dir; required for Move debugger) | `--trace` |
| `-e, --show-effects <true\|false>` | Whether to print full transaction effects (default true) | `--show-effects false` |
| `--overwrite` | Overwrite existing replay artifacts instead of erroring | `--overwrite` |
| `--terminate-early` | In batch mode, stop on the first error instead of continuing | `--terminate-early` |
| `-n, --node <NETWORK\|URL>` | Network alias (`mainnet`, `testnet`) or full GraphQL URL to replay against. Defaults to active wallet env | `--node mainnet`<br>`--node https://my-graphql.example.com` |
| `--client.env <ENV>` | Explicit Sui environment from config | `--client.env testnet` |

## 3. Output and Artifacts

Successful replay prints a rich summary:

- Transaction Effects (status, mutated/shared/deleted objects, gas cost, dependencies)
- Gas Report table
- Per-object storage rebate table

Example (abbreviated):

```
Successfully replayed transaction

╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
│ ... (digest, status, mutated objects, gas cost, etc.) ...                                         │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯

Transaction Gas Report ...
```

When `--trace` is used, a trace file is written that can be opened with the official Move debugger or fed to `sui analyze-trace gas-profile`.

By default the tool will **not** overwrite a previous `.replay/<digest>` directory. Add `--overwrite` to force regeneration.

## 4. Configuration via `replay.toml`

Create `~/.sui/sui_config/replay.toml` (or alongside your client config) to set defaults:

```toml
[flags]
show-effects = false
overwrite = true
```

Long flag names are used (without the leading `--`).

## 5. Common Workflows

```bash
# Replay one transaction from mainnet (using GraphQL under the hood)
sui replay --digest 5sbk5UYpQv1i84zkSBSWmowAmPTxw71U7Bb1gbiW5w4y --node mainnet

# Replay + generate trace for debugger
sui replay --digest <DIGEST> --trace --overwrite

# Batch replay a list of digests, stop at first failure
sui replay --digests-path failed_txs.txt --terminate-early --node testnet

# Analyze the trace for gas (after a --trace run)
sui analyze-trace -p .replay/<digest>/trace gas-profile
```

## 6. Quick Reference

```bash
# Single tx with trace
sui replay -d <DIGEST> --trace --overwrite -n mainnet

# Using active env
sui replay --digest <DIGEST>

# See everything available
sui replay --help
```

## Official Resources:

- Sui Replay CLI: https://docs.sui.io/references/cli/replay
- Move Debugger: https://docs.sui.io/references/ide/debugger
- Trace Analysis: https://docs.sui.io/references/cli/trace-analysis
- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet

---

*Previous Page: [`Sui CLI Basic Summary`](../cli_summary.md)*
