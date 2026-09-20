# Logos LEZ Wallet and Custom Token Guide

Step by step tutorial for the Logos Execution Zone (LEZ) **public testnet**:

![Uploading photo_3_2026-09-20_11-03-35.jpg…]()


1. Install system dependencies (including Rust/Cargo)
2. Install the wallet CLI
3. Create a wallet and save the recovery phrase
4. Create a public account
5. Initialize the account
6. Claim test tokens from the Piñata faucet
7. Create a custom token
8. Send the custom token

Official docs:

- [What is Logos?](https://docs.logos.co/get-started/what-is-logos)
- [Run an LEZ wallet via the CLI](https://docs.logos.co/lez/get-started/run-lez-wallet-via-cli)
- [Create and transfer custom tokens](https://docs.logos.co/lez/transfer-tokens/create-and-transfer-custom-tokens-on-the-logos-execution-zone)

This guide was verified against the public testnet using wallet tag **`v0.2.4`**.  
Tag `v0.2.1` can fail health checks because local program IDs do not match the remote sequencer.

Testnet tokens have no monetary value.

---

## 0. Operating system

Commands below target **Ubuntu / Debian / WSL**.

---

## 1. System packages

Most machines are missing these. The wallet will not compile without them.

```bash
sudo apt update
sudo apt install -y \
  git \
  curl \
  build-essential \
  clang \
  libclang-dev \
  pkg-config \
  pkgconf \
  libssl-dev \
  libpcsclite-dev
```

`libpcsclite-dev` is required for Keycard / PC/SC support. Without it, `cargo install` usually dies looking for `libpcsclite`.

### Fedora

```bash
sudo dnf install git curl gcc glibc-devel clang clang-devel pkgconf-pkg-config openssl-devel llvm-libs pcsc-lite-devel
```

### macOS

```bash
xcode-select --install
brew install pkg-config openssl
```

---

## 2. Install Rust and Cargo

Most people do not have Cargo. Install it with rustup:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Accept the default (`1`). Then reopen the terminal, or:

```bash
source "$HOME/.cargo/env"
```

Check:

```bash
rustc --version
cargo --version
```

The wallet repository pins the toolchain in `rust-toolchain.toml`.

---

## 3. Clone and install the wallet

```bash
git clone https://github.com/logos-blockchain/logos-execution-zone.git
cd logos-execution-zone
git fetch --tags
git checkout v0.2.4
```

If you already have a local wallet and want a clean start:

```bash
mv ~/.lee/wallet ~/.lee/wallet.old 2>/dev/null || true
```

Install the CLI:

```bash
cargo install --path lez/wallet --force
```

The first build takes several minutes.

Point it at public testnet:

```bash
wallet change-network testnet
```

Health check (connection + built-in program IDs):

```bash
wallet check-health
```

Success looks like:

```text
✅All looks good!
```

If you see:

```text
Local ID for authenticated transfer program is different from remote
```

the binary does not match the live testnet. Rebuild from `v0.2.4`, or try `v0.2.5-rc3`.

---

## 4. Recovery phrase and password

On first run the wallet asks for a password and prints a **recovery phrase**.

- Remember the password.
- Write the 24 words on paper. The CLI will not show them again.
- Keys live in:

```text
~/.lee/wallet/storage.json
```

Restore (this overwrites current storage):

```bash
wallet restore-keys --depth 2
```

LEZ does not print a MetaMask-style `0x…` private key. Account addresses look like `Public/…` or `Private/…`.

---

## 5. Create and initialize an account

```bash
wallet account new public
```

Example output:

```text
Generated new account with account_id Public/<ACCOUNT_ID> at path /0
With pk <PUBLIC_KEY_HEX>
```

Save `<ACCOUNT_ID>`. Inspect it:

```bash
wallet account get --account-id Public/<ACCOUNT_ID>
```

A new account is `Uninitialized`. Initialize it so it can spend native tokens:

```bash
wallet auth-transfer init --account-id Public/<ACCOUNT_ID>
```

You should get a transaction hash and a block number. Then:

```bash
wallet account get --account-id Public/<ACCOUNT_ID>
```

Expected: owned by `authenticated transfer program`, `"balance":0`.

List accounts:

```bash
wallet account ls
wallet account ls -l
```

---

## 6. Claim test tokens (Piñata)

```bash
wallet pinata claim --to Public/<ACCOUNT_ID>
```

Then:

```bash
wallet account get --account-id Public/<ACCOUNT_ID>
```

On current testnet this is usually **150**.

Do **not** reuse this funded account as the token definition or token supply account. It is already owned by the transfer program.

---

## 7. Create a custom token

You need two **empty public** accounts. Creating them is local and free.

```bash
wallet account new public
wallet account new public
wallet account ls
```

Pick two `Public/…` addresses that are still `Uninitialized`:

- `DEFINITION` — the token type (similar to a mint)
- `SUPPLY` — receives the full supply

```bash
wallet token new \
  --name SAMIMI \
  --total-supply 1000 \
  --definition-account-id Public/<DEFINITION> \
  --supply-account-id Public/<SUPPLY>
```

Name and total supply cannot be changed after creation.

Check:

```bash
wallet account get --account-id Public/<DEFINITION>
wallet account get --account-id Public/<SUPPLY>
```

Successful output looks like:

```text
Definition account owned by token program
{"Fungible":{"name":"SAMIMI","total_supply":1000,"metadata_id":null}}

Holding account owned by token program
{"Fungible":{"definition_id":"<DEFINITION>","balance":1000}}
```

### Why two public accounts?

A private supply account requires local ZK proving. Without prover artifacts you get:

```text
Failed to prove program: No such file or directory (os error 2)
```

For a first token, use Public/Public only.

Do not paste an explorer address unless that account’s keys are in **your** `storage.json`.

---

## 8. Send the custom token

Type the command on **one line** with normal spaces. Copied invisible spaces produce:

```text
error: unexpected argument ' ' found
```

```bash
wallet token send --from Public/<SUPPLY> --to Public/<RECIPIENT> --amount 100
```

The recipient can be an empty public account. The token program will claim it for this token only.

If you see `Transaction hash is …` followed by `All pollers failed`, the transaction may still have landed. Do not resend until you check balances:

```bash
wallet account get --account-id Public/<SUPPLY>
wallet account get --account-id Public/<RECIPIENT>
```

Explorer:

```text
https://explorer.testnet.lez.logos.co/transaction/<TX_HASH>
```

---

## 9. Example from a real testnet run

These values are examples only. Yours will differ.

| Role | Value |
|---|---|
| Main account + faucet | `Public/34wC8gXPhYR9iKwDXmotpRibPs5GVfDEPk2gZKTMxnwa` — native balance 150 |
| SAMIMI definition | `Public/3RbTQAzvs9okNdaoE7GZkdTF4pM8B8qwjZB22pji1YyG` |
| SAMIMI supply | `Public/GPr6u7UgPCPQioyfL3z9jNVv9WPWko5noUC1tqdRGaju` — balance 1000 |
| Create-token tx | `738c1ba050275fc8e7dac5e6d53908036e71a1fa242e7295a75e8cd406ea4e24` — block 15572 |

---

## 10. Quick troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `libpcsclite` / PCSC not found | Missing system package | `sudo apt install -y libpcsclite-dev pkgconf` |
| `cargo: command not found` | Rust not installed | rustup (section 2) |
| Local ID … different from remote | Wallet tag ≠ live testnet | `git checkout v0.2.4` then `cargo install --path lez/wallet --force` |
| Account is Uninitialized | Skipped init | `wallet auth-transfer init` |
| Failed to prove program | Private account, no prover | Use public accounts |
| unexpected argument `' '` | Invisible whitespace | Type the command on one line |
| All pollers failed | Finality poll timed out | Check explorer and `account get` before sending again |

---

## 11. Helpful commands

```bash
wallet --help
wallet account --help
wallet token --help
wallet token send --help
wallet pinata --help
```

Testnet sequencer:

```text
https://testnet.lez.logos.co
```
