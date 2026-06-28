# Sui CLI Basic Summary

The Sui CLI (`sui`) is the primary command-line interface for interacting with the Sui blockchain. It supports account management, transaction execution, Move development, local networks, and more.

## Main Top-Level Commands

| Command              | Purpose |
|----------------------|---------|
| [`sui client`](sui_client.md) | Core operations: addresses, gas, transactions, faucets, and network management |
| [`sui client ptb`](sui_client.md#6-programmable-transaction-blocks-sui-client-ptb) | Build and execute Programmable Transaction Blocks (PTBs) |
| [`sui move`](sui_move.md) | Move language tools: create, build, test, and manage packages |
| [`sui keytool`](sui_keytool.md) | Key generation, import, conversion, and management |
| `sui start`          | Start a local Sui development network |
| `sui genesis`        | Initialize a new Sui network |
| `sui replay`         | Replay and debug transactions |

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

### Development & Transactions
- `sui move new <name>` — Create a new Move package
- `sui move build` — Build a Move package
- `sui move test` — Run unit tests
- `sui client publish .` — Publish a Move package
- `sui client ptb ...` — Construct and execute complex transactions

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