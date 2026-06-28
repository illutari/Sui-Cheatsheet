# Sui CLI Basic Summary

The Sui CLI (`sui`) is the primary command-line interface for interacting with the Sui blockchain. It supports account management, transaction execution, Move development, local networks, and more.

## Main Top-Level Commands

| Command              | Purpose |
|----------------------|---------|
| [`sui client`](sui/sui_client.md) | Core operations: addresses, gas, transactions, faucets, and network management |
| [`sui client ptb`](sui/sui_client.md#6-programmable-transaction-blocks-sui-client-ptb) | Build and execute Programmable Transaction Blocks (PTBs) |
| [`sui move`](sui/sui_move.md) | Move language tools: create, build, test, and manage packages.<br>See also the [Move language reference](../../move/move_summary.md) |
| [`sui keytool`](sui/sui_keytool.md) | Key generation, import, conversion, and management |
| [`sui start`](sui/sui_start.md) | Start a local Sui development network |
| [`sui genesis`](sui/sui_genesis.md) | Initialize a new Sui network (for persisted localnets) |
| [`sui replay`](sui/sui_replay.md) | Replay and debug transactions |

## Essential Everyday Commands

### Account & Address Management
- `sui client addresses` — List all addresses
- `sui client active-address` — Show the currently active address
- `sui client new-address <scheme>` — Create a new address (`ed25519`, `secp256k1`, or `secp256r1`)
- `sui client switch --address <ADDRESS>` — Switch active address

### Faucet & Gas
- `sui client faucet` — Request test SUI tokens
- `sui client gas` — List owned gas coins

### Network Management
- `sui client envs` — List configured networks
- `sui client active-env` — Show current network
- `sui client switch --env <NAME>` — Switch between networks (mainnet, testnet, devnet, local)

### Local Development
- [`sui start`](sui/sui_start.md) — Start local network (use `--force-regenesis --with-faucet` for fresh dev)
- [`sui genesis`](sui/sui_genesis.md) — Generate persisted genesis config
- [`sui replay`](sui/sui_replay.md) — Replay txs for debugging / traces

### Development & Transactions
- `sui move new <name>` — Create a new Move package
- `sui move build` — Build a Move package
- `sui move test` — Run unit tests
- [`sui client publish`](sui/sui_client.md#7-publishing-packages-sui-client-publish) — Publish a Move package (also see [`--publish` in PTB](sui/sui_client.md#9-ptb-publish-and-upgrade))
- [`sui client upgrade`](sui/sui_client.md#8-upgrading-packages-sui-client-upgrade) — Upgrade a published package
- `sui client ptb ...` — Construct and execute complex transactions (including `--publish` / `--upgrade`)

## Quick Reference

```bash
# Version and Help
sui --version
sui --help
sui client --help

# Common Workflow
sui client faucet
sui move new my_project
cd my_project
sui move build
sui client publish .
```