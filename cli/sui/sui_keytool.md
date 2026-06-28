# Sui Keytool CLI Cheatsheet

The `sui keytool` command provides cryptographic utilities for managing keys, addresses, signatures, and MultiSig/zkLogin operations in Sui.

**Most commands interact with `~/.sui/sui_config/sui.keystore`.** Use `--help` on any subcommand for full options.

## 1. Core Key Management

| Command | Description | Example |
|---------|-------------|---------|
| `sui keytool list` | List all keys (address, public key, scheme) in the keystore | `sui keytool list` |
| `sui keytool generate <SCHEME>` | Generate a new keypair (`ed25519`, `secp256k1`, or `secp256r1`) and save to file | `sui keytool generate ed25519` |
| `sui keytool import <INPUT> <SCHEME>` | Import key using mnemonic phrase or Bech32 `suiprivkey` | `sui keytool import "mnemonic words..." ed25519` |
| `sui keytool export <ADDRESS-or-ALIAS>` | Export private key as Bech32 `suiprivkey` | `sui keytool export 0x...` |
| `sui keytool convert <PRIVATE_KEY>` | Convert legacy Hex/Base64 to new Bech32 format | `sui keytool convert 0x...` |
| `sui keytool update-alias <OLD_ALIAS> <NEW_ALIAS>` | Update or assign alias to an address | `sui keytool update-alias my_old_alias new_alias` |

## 2. Key Inspection

| Command | Description | Example |
|---------|-------------|---------|
| `sui keytool show <FILE>` | Show keypair details from a `.key` file | `sui keytool show 0x....key` |
| `sui keytool load-keypair <FILE>` | Load and display keypairs from file (SuiKeyPair or AuthorityKeyPair) | `sui keytool load-keypair authority.key` |
| `sui keytool unpack <BASE64_KEYPAIR>` | Unpack Base64 keypair to file | `sui keytool unpack <base64>` |

## 3. Signing and Verification

| Command | Description | Example |
|---------|-------------|---------|
| `sui keytool sign --address <ADDR> --tx-bytes <BASE64>` | Sign transaction bytes | `sui keytool sign --address 0x... --tx-bytes <base64>` |
| `sui keytool sign-kms` | Sign using AWS KMS | `sui keytool sign-kms --help` |
| `sui keytool decode-or-verify-tx` | Decode and optionally verify transaction | `sui keytool decode-or-verify-tx --tx-bytes <base64>` |

## 4. MultiSig Commands

| Command | Description | Example |
|---------|-------------|---------|
| `sui keytool multi-sig-address` | Create MultiSig address from public keys | `sui keytool multi-sig-address --pks <base64-pk1> <base64-pk2> --weights 1 1 --threshold 2` |
| `sui keytool multi-sig-combine-partial-sig` | Combine partial signatures into MultiSig | `sui keytool multi-sig-combine-partial-sig --sigs <base64-sig1> ... --threshold 2` |
| `sui keytool decode-multi-sig` | Decode and verify MultiSig signature | `sui keytool decode-multi-sig --sig <base64>` |

## 5. zkLogin Commands (Advanced)

| Command | Description | Example |
|---------|-------------|---------|
| `sui keytool zk-login` | Generate zkLogin signature (interactive) | `sui keytool zk-login --help` |
| `sui keytool zk-login-sig-verify` | Verify zkLogin signature | `sui keytool zk-login-sig-verify --sig <sig> --bytes <bytes>` |

## Common Options

- `--keystore-path <PATH>` — Custom keystore location
- `--json` — Output in JSON format
- `--alias <NAME>` — Use/set alias (with `import`/`generate`)
- `--help` — Detailed help for any command

## Quick Examples

```bash
# Generate and inspect new key
sui keytool generate ed25519
sui keytool list

# Import from mnemonic
sui keytool import "word1 word2 ..." ed25519 --alias my_wallet

# Export private key
sui keytool export my_wallet
```

## Official Resources:

- Sui Keytool Documentation: https://docs.sui.io/references/cli/keytool
- Sui CLI Cheatsheet: https://docs.sui.io/references/cli/cheatsheet

For the most up-to-date information, run `sui keytool --help`.

---

*Previous Page: [`Sui CLI Basic Summary`](../cli_summary.md)*