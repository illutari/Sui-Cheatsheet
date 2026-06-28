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
7. [Publishing Packages (`sui client publish`)](#7-publishing-packages-sui-client-publish)
8. [Upgrading Packages (`sui client upgrade`)](#8-upgrading-packages-sui-client-upgrade)
9. [PTB Publish and Upgrade](#9-ptb-publish-and-upgrade)
10. [Quick Start Tips](#quick-start-tips)
11. [Official Resources](#official-resources)

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
<a name="6-programmable-transaction-blocks-sui-client-ptb"></a>

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
| `--publish <PATH>` | Publish a Move package (returns UpgradeCap as first result; must transfer or make immutable) | `--publish .` |
| `--upgrade <PATH>` | Upgrade an existing package (requires passing an UpgradeCap via prior move-call or assign) | `--upgrade .` |
| `--make-move-vec <TYPE> "[VALUES]"` | Create vector | `--make-move-vec <u64> "[1,2,3]"` |

**Publish & Upgrade in PTB**:

- `--publish <PATH>` compiles the package at the path and executes a publish. The result (UpgradeCap) is available as the first command result — use `--assign upgrade_cap` immediately after and transfer it (or call `sui::package::make_immutable` on it).
- `--upgrade <PATH>` performs an upgrade. You must supply the `UpgradeCap` (via a previous command result or as an input object) in the same PTB.

See dedicated sections below for the standalone `sui client publish` / `upgrade` commands (preferred for simple cases) and expanded PTB examples.

**Example PTB**:
```bash
sui client ptb \
  --split-coins gas "[1000000000]" \
  --assign new_coin \
  --transfer-objects "[new_coin]" @0x... \
  --gas-budget 50000000
  ```

## 7. Publishing Packages (`sui client publish`)
<a name="7-publishing-packages-sui-client-publish"></a>

The primary way to publish a new Move package.

```bash
sui client publish [package_path] [OPTIONS]
```

- `package_path` defaults to `.` (current directory, which must contain a `Move.toml`).
- Automatically builds the package (supports most `sui move build` flags such as `--test`, `--doc`, `--force`, `--no-lint`, etc.).
- Gas budget is optional — the CLI performs a dry-run to estimate when omitted (since Sui v1.24.1).
- On success, the package object is created (immutable) and an `UpgradeCap` is transferred to the sender (or the address specified via `--sender`).
- Updates (or creates) `Published.toml` with the on-chain address, version, `upgrade-capability`, toolchain info, etc. **Commit `Published.toml`** to source control.
- For local/ephemeral networks use `sui client test-publish` (writes to `Pub.<env>.toml` which should **not** be committed).

**Key flags** (common to publish/upgrade):

| Flag | Purpose | Example |
|------|---------|---------|
| `--gas-budget <MIST>` | Max gas (omit for auto-estimate) | `--gas-budget 100000000` |
| `--dry-run` | Simulate only | `--dry-run` |
| `--json` | Machine readable output (pipe to `jq`) | `sui client publish --json 2>/dev/null \| jq ...` |
| `--with-unpublished-dependencies` | Also publish unpinned transitive deps | `--with-unpublished-dependencies` |
| `--skip-dependency-verification` | Skip on-chain source verification of deps | `--skip-dependency-verification` |
| `--sender <ADDR>` | Override sender | `--sender 0x...` |
| `--gas-sponsor <ADDR>` | Sponsor the gas | `--gas-sponsor 0x...` |

**Example**:
```bash
cd my_package
sui client publish
# or with explicit gas + json for scripting
sui client publish . --gas-budget 200000000 --json
```

After publishing, inspect the **Object Changes** section for:
- The new package ID (immutable object)
- The `0x2::package::UpgradeCap` ID (owned by you — save it for upgrades)

See also `sui client test-publish --help` for local dev.

## 8. Upgrading Packages (`sui client upgrade`)
<a name="8-upgrading-packages-sui-client-upgrade"></a>

Upgrades an existing package while preserving its on-chain identity (original ID).

```bash
sui client upgrade [package_path] --upgrade-capability <UPGRADE_CAP_ID> [OPTIONS]
```

**Requirements**:
- The package must be layout-compatible (same public function signatures & struct layouts; new items and impl changes are allowed).
- You must own / control the `UpgradeCap` from the original (or previous) publish.
- `init` functions do **not** re-run on upgrade.
- Old package versions remain on chain forever.

**Typical upgrade steps**:
1. Make code changes (bump any internal `VERSION` constants if you use versioned migration patterns).
2. Run the upgrade command, passing the cap.
3. (If using shared objects + version guards) call your `migrate(...)` entry function in a follow-up tx (protected by AdminCap).

**Example**:
```bash
# After saving the UpgradeCap ID from the initial publish output
sui client upgrade . --upgrade-capability 0x34f7cf31a0a12f81252ab947cb51146bc8138fa5adb3f1fe38e244734319d73c
```

Useful flags (in addition to the shared ones above):
- `-c, --upgrade-capability <ID>` **(required)**
- `--skip-verify-compatibility` — Skip local compatibility check before submitting.

After a successful upgrade, `Published.toml` is updated with the new `version` and the same `upgrade-capability` (the cap object itself is mutated).

For advanced policies (timelocks, admin-only, make_immutable, etc.) see the custom upgrade policy guide and call `sui::package::make_immutable` on the cap (or your policy object).

## 9. PTB Publish and Upgrade (`--publish` / `--upgrade`)
<a name="9-ptb-publish-and-upgrade"></a>

While the standalone `sui client publish` / `upgrade` are simplest, PTBs give full control (e.g. publish + immediately call a function from the new package, or atomic publish+other steps).

**Publish example (standard pattern)**:
```bash
sui client ptb \
  --move-call sui::tx_context::sender \
  --assign sender \
  --publish "." \
  --assign upgrade_cap \
  --transfer-objects "[upgrade_cap]" sender \
  --gas-budget 100000000
```

This acquires the sender, publishes, binds the resulting UpgradeCap, and transfers it back to the sender in one PTB.

**Upgrade with PTB** (conceptual; supply the cap):
```bash
sui client ptb \
  --assign cap @0xUPGRADE_CAP_ID \
  --upgrade "." \
  --gas-budget 50000000
```
(Exact PTB syntax for passing the cap to `--upgrade` may require an authorizing move call in more complex policies; the standalone `sui client upgrade` is usually preferred.)

See the official PTB docs for current syntax and `--preview` / `--dry-run` during development.

## 10. Quick Start Tips

```
# Initial setup
sui client

# Common workflow
sui client faucet
sui client gas
sui move new my_project
cd my_project
sui move build
sui client publish .
# or for more control:
# sui client ptb --publish . --gas-budget 1000000000
```

## 11. Official Resources:

- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet
- Sui Client PTB: https://docs.sui.io/references/cli/ptb
- Move Package Management: https://docs.sui.io/develop/manage-packages/move-package-management
- Upgrading Packages: https://docs.sui.io/develop/publish-upgrade-packages/upgrade
- Full Sui CLI Docs: https://docs.sui.io/references/cli

For the most up-to-date list, run `sui client --help` or `sui client ptb --help`.

---

*Previous Page: [`Sui CLI Basic Summary`](../cli_summary.md)*