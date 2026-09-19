# Quick install (Ubuntu / Debian / WSL)

```bash
sudo apt update
sudo apt install -y git curl build-essential clang libclang-dev pkg-config pkgconf libssl-dev libpcsclite-dev

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"

rustc --version
cargo --version
```

Keep `libpcsclite-dev`. The LEZ wallet build depends on it.
