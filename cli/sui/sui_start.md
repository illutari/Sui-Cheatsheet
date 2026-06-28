# Sui Start CLI Cheatsheet

The `sui start` command starts a local Sui development network (often called "localnet"). It is the recommended way to run a private Sui network on your machine for development and testing.

**Run from any directory.** The command supports two primary modes:
- **Ephemeral** (`--force-regenesis`): Fresh genesis every run; state is not saved.
- **Persisted**: Uses a genesis created by `sui genesis`; state is kept between restarts.

**Note:** Indexer, Consistent Store, and GraphQL features require a running PostgreSQL database. Basic fullnode + faucet do not.

## 1. Basic Usage

| Command | Description | Example |
|---------|-------------|---------|
| `sui start` | Start local network (auto-creates genesis on first run if none exists; persists state) | `sui start` |
| `sui start --force-regenesis` | Start a fresh ephemeral network every time (recommended for clean dev loops) | `sui start --force-regenesis` |
| `sui start --with-faucet` | Start network + local faucet (default: 0.0.0.0:9123) | `sui start --with-faucet` |

## 2. Common Options

| Flag | Description | Example |
|------|-------------|---------|
| `--force-regenesis` | Force new genesis + no state persistence between runs | `sui start --force-regenesis` |
| `--with-faucet[=<HOST:PORT>]` | Start faucet service. Default `0.0.0.0:9123`. Use `=port` or `=host:port` syntax | `sui start --with-faucet=9123`<br>`sui start --with-faucet=0.0.0.0:9123` |
| `--with-indexer[=<DB_URL>]` | Start indexer. Omitting value uses temp DB in config dir; provide Postgres URL for external | `sui start --with-indexer`<br>`sui start --with-indexer=postgres://user:pass@localhost:5432/sui` |
| `--with-graphql[=<HOST:PORT>]` | Start GraphQL server (default 0.0.0.0:9125). Auto-enables indexer + consistent store | `sui start --with-graphql` |
| `--with-consistent-store[=<HOST:PORT>]` | Start Consistent Store (default 0.0.0.0:9124). Auto-enabled by GraphQL | `sui start --with-consistent-store` |
| `--fullnode-rpc-port <PORT>` | Fullnode RPC listen port (default: 9000) | `sui start --fullnode-rpc-port 9000` |
| `--epoch-duration-ms <MS>` | Epoch length in milliseconds (only with `--force-regenesis` or no prior genesis; default 60s when force) | `sui start --force-regenesis --epoch-duration-ms 30000` |
| `--committee-size <N>` | Number of validators (only affects new genesis) | `sui start --committee-size 1` |
| `--network.config <DIR>` | Use a custom config directory (generated via `sui genesis --write-config`) | `sui start --network.config ~/.sui/localnet` |
| `--data-ingestion-dir <DIR>` | Dump executed checkpoints as files (useful for data pipelines) | `sui start --data-ingestion-dir ./checkpoints` |
| `--no-full-node` | Run without a fullnode (validators only mode) | `sui start --no-full-node` |

**Protocol overrides** (advanced): Set `SUI_PROTOCOL_CONFIG_OVERRIDE_ENABLE=1` + `SUI_PROTOCOL_CONFIG_OVERRIDE_<param>=value` env vars.

## 3. Persisted vs Ephemeral Networks

- **Ephemeral (default dev workflow)**:
  ```bash
  RUST_LOG="off,sui_node=info" sui start --with-faucet --force-regenesis
  ```
  - No history preserved across restarts.
  - Use for rapid iteration / clean slate testing.
  - Each run gets new addresses / objects.

- **Persisted**:
  1. Generate genesis once (see [`sui genesis`](sui_genesis.md)):
     ```bash
     sui genesis --force --with-faucet --working-dir ~/.sui/my-localnet
     ```
  2. Start using the config:
     ```bash
     sui start --network.config ~/.sui/my-localnet
     ```
  - State, history, and objects survive restarts.
  - Do **not** pass `--force-regenesis`.

## 4. Connecting the Sui CLI to Your Local Network

```bash
# Add and switch to local env (RPC default 127.0.0.1:9000)
sui client new-env --alias local --rpc http://127.0.0.1:9000
sui client switch --env local

# Verify
sui client active-env
sui client active-address

# Get test SUI (wait ~60s for coins to arrive)
sui client faucet
sui client gas
```

## 5. Quick Examples

```bash
# Clean ephemeral devnet with faucet (most common)
sui start --force-regenesis --with-faucet

# With full dev stack (indexer + GraphQL explorer)
sui start --force-regenesis --with-faucet --with-indexer --with-graphql

# Custom port + longer epochs
sui start --force-regenesis --with-faucet --fullnode-rpc-port 9001 --epoch-duration-ms 120000

# Persisted custom network
sui genesis -f --with-faucet --working-dir /tmp/sui-local
sui start --network.config /tmp/sui-local
```

After starting, monitor via any Sui explorer by pointing it at `http://127.0.0.1:9000` (or your custom RPC port).

## 6. Troubleshooting

- **Port in use** (9000 etc.): Kill process or change port with `--fullnode-rpc-port`.
- **Faucet not working**: Ensure you waited after `sui client faucet`; check logs.
- **Postgres for indexer**: Install `libpq` / Postgres and use a valid `--with-indexer=...` URL.
- **TMPDIR issues**: If `/tmp` is tmpfs, set `TMPDIR=./local-tmp sui start ...`

For full current options always run:

```bash
sui start --help
```

## Official Resources:

- Connecting to a Local Network: https://docs.sui.io/getting-started/onboarding/local-network
- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet
- Full Sui CLI reference: https://docs.sui.io/references/cli

---

*Previous Page: [`Sui CLI Basic Summary`](../cli_summary.md)*
