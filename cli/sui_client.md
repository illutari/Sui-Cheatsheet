# Sui Client CLI Cheatsheet

The `sui client` command is the primary interface for interacting with the Sui blockchain — managing addresses, gas, networks, and executing transactions.

**Run commands from any directory.** Most commands support `--help` for detailed options.

### Table of Contents

1. [Addresses and Aliases](#1-addresses-and-aliases)
2. [Faucet and Gas](#2-faucet-and-gas)
3. [Network Management](#3-network-management)
4. [Object and Transaction Inspection](#4-object-and-transaction-inspection)
5. [Executing Transactions (Legacy `call`)](#5-executing-transactions-legacy-call)
6. [Programmable Transaction Blocks (`sui client ptb`)](#6-programmable-transaction-blocks-sui-client-ptb)
7. [Quick Start Tips](#quick-start-tips)
8. [Official Resources](#official-resources)

## 1. Addresses and Aliases

| Command | Description | Example |
|---------|-------------|---------|
| `sui client active-address` | Show the currently active address | `sui client active-address` |
| `sui client addresses` | List all addresses, aliases, and active address | `sui client addresses` |
| `sui client new-address <scheme>` | Create a new address (`ed25519`, `secp256k1`, or `secp256r1`) | `sui client new-address ed25519` |
| `sui client new-address <scheme> <alias>` | Create address with custom alias | `sui client new-address ed25519 my_wallet` |
| `sui client switch --address <ADDRESS>` | Switch active address (supports alias) | `sui client switch --address 0x...` |

## 2. Faucet and Gas

| Command | Description | Example |
|---------|-------------|---------|
| `sui client faucet` | Request test SUI for active address | `sui client faucet` |
| `sui client faucet --address <ADDRESS>` | Request SUI for specific address | `sui client faucet --address 0x...` |
| `sui client faucet --url <URL>` | Use custom faucet | `sui client faucet --url https://faucet.example.com` |
| `sui client gas` | List gas coins for active address | `sui client gas` |
| `sui client gas <ADDRESS>` | List gas coins for specific address | `sui client gas 0x...` |

## 3. Network Management

| Command | Description | Example |
|---------|-------------|---------|
| `sui client active-env` | Show current network | `sui client active-env` |
| `sui client envs` | List all configured environments | `sui client envs` |
| `sui client new-env --rpc <URL> --alias <NAME>` | Add a new network | `sui client new-env --rpc https://fullnode.testnet.sui.io --alias testnet` |
| `sui client switch --env <ALIAS>` | Switch network | `sui client switch --env testnet` |

## 4. Object and Transaction Inspection

| Command | Description | Example |
|---------|-------------|---------|
| `sui client object --id <OBJECT_ID>` | View object details | `sui client object --id 0x...` |
| `sui client objects` | List objects owned by active address | `sui client objects` |
| `sui client tx --digest <TX_DIGEST>` | View transaction details | `sui client tx --digest ...` |

## 5. Executing Transactions (Legacy `call`)

| Command | Description | Example |
|---------|-------------|---------|
| `sui client call ...` | Call a Move entry function | `sui client call --package 0x... --module m --function f --args ... --gas-budget 100000000` |

**Recommended**: Use `sui client ptb` for modern Programmable Transaction Blocks (see below).

## 6. Programmable Transaction Blocks (`sui client ptb`)

The modern way to build complex transactions.

**Common Flags**:
- `--gas-budget <MIST>` — Set gas budget
- `--dry-run` — Simulate without executing
- `--preview` — Show PTB commands without executing
- `--summary` — Show short summary only
- `--json` — Output in JSON format

**Key PTB Commands**:

| PTB Command | Description | Example |
|-------------|-------------|---------|
| `--move-call <PKG::MOD::FN> <TYPE_ARGS> <ARGS>` | Call Move function | `--move-call sui::coin::mint_and_transfer "<SUI>" 1000 @recipient` |
| `--split-coins <COIN> "[AMOUNTS]"` | Split coin | `--split-coins gas "[100000000, 200000000]"` |
| `--merge-coins <INTO> "[COINS]"` | Merge coins | `--merge-coins @primary "[@coin1, @coin2]"` |
| `--transfer-objects "[OBJECTS]" <TO>` | Transfer objects | `--transfer-objects "[gas]" @recipient` |
| `--publish <PATH>` | Publish Move package | `--publish .` |
| `--upgrade <PATH>` | Upgrade package | `--upgrade .` |
| `--make-move-vec <TYPE> "[VALUES]"` | Create vector | `--make-move-vec <u64> "[1,2,3]"` |

**Example PTB**:
```bash
sui client ptb \
  --split-coins gas "[1000000000]" \
  --assign new_coin \
  --transfer-objects "[new_coin]" @0x... \
  --gas-budget 50000000
  ```

## 7. Quick Start Tips

```
# Initial setup
sui client

# Common workflow
sui client faucet
sui client gas
sui move new my_project
cd my_project
sui move build
sui client ptb --publish . --gas-budget 1000000000
```

## 8. Official Resources:

- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet
- Sui Client PTB: https://docs.sui.io/references/cli/ptb
- Full Sui CLI Docs: https://docs.sui.io/references/cli

For the most up-to-date list, run `sui client --help` or `sui client ptb --help`.

---

*Previous Page: [`Sui CLI Basic Summary`](cli_summary.md)*