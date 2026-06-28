# Sui Genesis CLI Cheatsheet

The `sui genesis` command bootstraps and initializes a new Sui network by generating the genesis configuration, validator keys, the initial `genesis.blob`, and related files.

Use `sui genesis` when you want a **persisted local network** (state survives restarts). The resulting config directory can then be used with `sui start --network.config <DIR>`.

For one-off ephemeral development, `sui start --force-regenesis` will auto-generate a genesis internally — you usually do not need to call `sui genesis` directly.

## 1. Basic Usage

| Command | Description | Example |
|---------|-------------|---------|
| `sui genesis` | Generate default genesis in the working directory (or `~/.sui/sui_config`) | `sui genesis` |
| `sui genesis --force` | Overwrite any existing configuration in the target directory | `sui genesis --force` |
| `sui genesis --with-faucet` | Include a faucet configuration (so `sui start` can easily launch a faucet) | `sui genesis --with-faucet` |

## 2. Common Options

| Flag | Description | Example |
|------|-------------|---------|
| `-f, --force` | Force overwrite of existing config directory / keys | `sui genesis -f --with-faucet` |
| `--working-dir <DIR>` | Directory in which to write the genesis config and `genesis.blob` (defaults to Sui config dir) | `sui genesis --working-dir ~/.sui/my-localnet` |
| `--from-config <FILE>` | Use an existing genesis config file (instead of generating interactively) | `sui genesis --from-config my-genesis.yaml` |
| `--write-config <FILE>` | Build a genesis config, write it to the given path, and exit (without creating the full genesis) | `sui genesis --write-config genesis-config.yaml` |
| `--epoch-duration-ms <MS>` | Set initial epoch duration for the generated network | `sui genesis --epoch-duration-ms 60000` |
| `--committee-size <N>` | Number of validators to include in the genesis committee | `sui genesis --committee-size 4` |
| `--benchmark-ips <ADDR>...` | Generate a genesis configuration optimized for benchmarks using the provided IP list | `sui genesis --benchmark-ips 10.0.0.1 10.0.0.2` |
| `--with-faucet` | Create faucet configuration entry alongside the genesis (pairs well with `sui start --with-faucet`) | `sui genesis --force --with-faucet` |

## 3. Typical Workflow for a Persisted Local Network

```bash
# 1. Create a dedicated config dir + genesis with faucet support
sui genesis --force --with-faucet --working-dir ~/.sui/localnet

# 2. (Optional) Inspect the generated files
ls ~/.sui/localnet
# genesis.blob, fullnode.yaml, genesis-config.yaml, etc.

# 3. Start the network using the generated config (state will persist)
sui start --network.config ~/.sui/localnet --with-faucet

# 4. In another terminal, configure your client to talk to it
sui client new-env --alias local --rpc http://127.0.0.1:9000
sui client switch --env local
sui client faucet
sui client gas
```

## 4. Generating Config Only (Advanced)

Use `--write-config` when you want to customize the YAML before final genesis creation:

```bash
sui genesis --write-config /tmp/my-config.yaml --committee-size 1
# Edit /tmp/my-config.yaml as needed...
sui genesis --from-config /tmp/my-config.yaml --working-dir ~/.sui/custom
```

## 5. Quick Reference

```bash
# Fresh persisted localnet (common pattern)
sui genesis -f --with-faucet --working-dir /tmp/sui-local
sui start --network.config /tmp/sui-local

# Minimal single-validator genesis
sui genesis --force --committee-size 1 --epoch-duration-ms 30000
```

Always run `sui genesis --help` for the exact flags available in your CLI version.

## Official Resources:

- Connecting to a Local Network: https://docs.sui.io/getting-started/onboarding/local-network
- Genesis (operator concepts): https://docs.sui.io/operators/genesis
- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet

---

*Previous Page: [`Sui CLI Basic Summary`](../cli_summary.md)*
