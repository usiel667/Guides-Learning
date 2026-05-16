# Rust Development Setup with LazyVim

## Table of Contents

1. [System Requirements](#1-system-requirements)
2. [Install Rust (rustup)](#2-install-rust-rustup)
3. [Essential Rust Tools](#3-essential-rust-tools)
4. [LazyVim Prerequisites](#4-lazyvim-prerequisites)
5. [LazyVim Rust Plugin Setup](#5-lazyvim-rust-plugin-setup)
6. [LSP & Autocompletion](#6-lsp--autocompletion)
7. [Debugging Setup](#7-debugging-setup)
8. [Verify Everything Works](#8-verify-everything-works)
9. [Useful Keybindings](#9-useful-keybindings)
10. [Common Crates to Know](#10-common-crates-to-know)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. System Requirements

Before installing anything, make sure your system has these basics:

| Dependency | Purpose | Check with |
|---|---|---|
| `curl` or `wget` | Downloading installers | `curl --version` |
| `git` | Version control, crate fetching | `git --version` |
| `gcc` or `clang` | C linker (Rust needs one) | `gcc --version` |
| `pkg-config` | Linking system libraries | `pkg-config --version` |
| `unzip` | Unpacking tools | `unzip --help` |

**Linux (Ubuntu/Debian):**
```bash
sudo apt update
sudo apt install -y curl git gcc pkg-config unzip build-essential
```

**Linux (Arch/Manjaro):**
```bash
sudo pacman -S curl git gcc pkg-config unzip base-devel
```

**macOS:**
```bash
xcode-select --install   # installs clang + git
brew install pkg-config  # if you use Homebrew
```

---

## 2. Install Rust (rustup)

`rustup` is the official Rust toolchain installer. It manages Rust versions and components.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Follow the prompts (default installation is fine). Then reload your shell:

```bash
source "$HOME/.cargo/env"
```

**Verify:**
```bash
rustc --version    # e.g. rustc 1.78.0
cargo --version    # e.g. cargo 1.78.0
```

### Rust Toolchain Components

After installing, add the components LazyVim's tooling depends on:

```bash
rustup component add rust-analyzer   # language server (LSP)
rustup component add rustfmt         # auto-formatter
rustup component add clippy          # linter
```

### Keeping Rust Updated

```bash
rustup update        # update to latest stable
rustup show          # see installed toolchains
```

---

## 3. Essential Rust Tools

Install these CLI tools with `cargo install`. They improve the development workflow significantly.

| Tool | Install command | What it does |
|---|---|---|
| `cargo-watch` | `cargo install cargo-watch` | Re-runs cargo commands on file save |
| `cargo-edit` | `cargo install cargo-edit` | `cargo add`/`cargo rm` for dependencies |
| `cargo-nextest` | `cargo install cargo-nextest` | Faster test runner |
| `cargo-expand` | `cargo install cargo-expand` | Shows macro expansions |
| `cargo-audit` | `cargo install cargo-audit` | Checks for security vulnerabilities |

**Install all at once:**
```bash
cargo install cargo-watch cargo-edit cargo-nextest cargo-expand cargo-audit
```

> These install into `~/.cargo/bin/`, which is already on your PATH after `rustup` setup.

---

## 4. LazyVim Prerequisites

LazyVim itself requires a few system tools to work well with Rust.

### Neovim

You need **Neovim 0.9+** (0.10+ recommended):

```bash
# Ubuntu/Debian — use the official PPA for a recent version
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt update
sudo apt install neovim

# Arch
sudo pacman -S neovim

# macOS
brew install neovim
```

**Verify:** `nvim --version`

### Nerd Font

LazyVim displays icons. Install any Nerd Font (e.g. JetBrainsMono Nerd Font) and set it in your terminal emulator. Download from: https://www.nerdfonts.com/font-downloads

### ripgrep (for fuzzy search)

```bash
# Ubuntu/Debian
sudo apt install ripgrep

# Arch
sudo pacman -S ripgrep

# macOS
brew install ripgrep
```

### fd (faster file finder)

```bash
# Ubuntu/Debian
sudo apt install fd-find
ln -s $(which fdfind) ~/.local/bin/fd   # create alias if needed

# Arch
sudo pacman -S fd

# macOS
brew install fd
```

### Node.js (for some LSP tools)

Some Mason-installed tools need Node:

```bash
# Ubuntu/Debian (via NodeSource for a current version)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Arch
sudo pacman -S nodejs npm

# macOS
brew install node
```

---

## 5. LazyVim Rust Plugin Setup

LazyVim has an official **Rust extra** that wires everything up automatically.

### Enable the Rust Extra

In your LazyVim config (`~/.config/nvim/lua/plugins/`), create or edit a file (e.g. `rust.lua`):

```lua
-- ~/.config/nvim/lua/plugins/rust.lua
return {
  { import = "lazyvim.plugins.extras.lang.rust" },
}
```

This single import adds:
- `rustaceanvim` — the main Rust plugin (replaces `rust-tools`)
- `crates.nvim` — inline crate version info and updates in `Cargo.toml`
- Treesitter grammar for Rust
- DAP (debug adapter) config for Rust via `codelldb`

### What Gets Installed Automatically

When you open Neovim after adding the extra, **Mason** (LazyVim's tool installer) will prompt to install:

| Mason package | Role |
|---|---|
| `rust-analyzer` | Language server (LSP) |
| `codelldb` | Debug adapter for `lldb` |
| `taplo` | LSP + formatter for `Cargo.toml` |

You can also install them manually:
```
:MasonInstall rust-analyzer codelldb taplo
```

### crates.nvim (optional standalone config)

If you want extra crate management features, you can configure `crates.nvim` further:

```lua
-- already included via the rust extra, but you can override options:
{
  "saecki/crates.nvim",
  opts = {
    completion = {
      crates = { enabled = true },
    },
  },
},
```

---

## 6. LSP & Autocompletion

LazyVim uses `nvim-lspconfig` + `mason-lspconfig` under the hood. With the Rust extra enabled, `rust-analyzer` is configured automatically via `rustaceanvim` (it does NOT go through `lspconfig` — this is intentional).

### rust-analyzer Settings

You can customize `rust-analyzer` behaviour inside `rustaceanvim`:

```lua
-- ~/.config/nvim/lua/plugins/rust.lua
return {
  { import = "lazyvim.plugins.extras.lang.rust" },
  {
    "mrcjkb/rustaceanvim",
    opts = {
      server = {
        settings = {
          ["rust-analyzer"] = {
            checkOnSave = { command = "clippy" },  -- use clippy instead of check
            cargo = { allFeatures = true },
            inlayHints = { lifetimeElisionHints = { enable = "always" } },
          },
        },
      },
    },
  },
}
```

### Autocompletion

LazyVim uses `nvim-cmp` by default (or `blink.cmp` if you enable that extra). Both work out of the box with `rust-analyzer`. No extra setup needed.

---

## 7. Debugging Setup

The Rust extra installs `codelldb` via Mason and configures `nvim-dap` automatically.

### Install codelldb

If it wasn't auto-installed:
```
:MasonInstall codelldb
```

### Basic Debug Usage

Open a Rust project and use these commands (LazyVim default DAP keymaps under `<leader>d`):

| Action | Key |
|---|---|
| Set breakpoint | `<leader>db` |
| Start / continue | `<leader>dc` |
| Step over | `<leader>do` |
| Step into | `<leader>di` |
| Step out | `<leader>dO` |
| Terminate | `<leader>dq` |

To debug a specific binary, use the `rustaceanvim` command which auto-detects targets:
```
:RustLsp debuggables
```

---

## 8. Verify Everything Works

### Step 1 — Create a test project

```bash
cargo new hello_rust
cd hello_rust
nvim src/main.rs
```

### Step 2 — Check LSP is running

Inside Neovim:
```
:LspInfo
```
You should see `rust-analyzer` listed as active for the current buffer.

### Step 3 — Check Treesitter

```
:TSInstall rust     # install if not already
:checkhealth nvim-treesitter
```

### Step 4 — Full health check

```
:checkhealth
```

Look for any `ERROR` lines and resolve them. `WARNING` lines are usually fine to skip.

### Step 5 — Run and watch

```bash
cargo watch -x run       # re-runs on every file save
cargo watch -x test      # re-runs tests on every file save
```

---

## 9. Useful Keybindings

These are available in Rust files via `rustaceanvim` (all under normal mode):

| Key | Action |
|---|---|
| `<leader>ca` | Code actions (including Rust-specific ones) |
| `K` | Hover docs |
| `gd` | Go to definition |
| `gr` | Find references |
| `<leader>cr` | Rename symbol |
| `<leader>cf` | Format file (`rustfmt`) |
| `<leader>rr` | `:RustLsp runnables` — pick a target to run |
| `<leader>rd` | `:RustLsp debuggables` — pick a target to debug |
| `<leader>re` | `:RustLsp expandMacro` — expand macro under cursor |
| `<leader>rp` | `:RustLsp parentModule` — jump to parent module |

> You can customise these in your `which-key` config if the defaults clash.

---

## 10. Common Crates to Know

Crates are Rust's equivalent of packages/libraries. Unlike the tools in section 3, **crates are not installed globally** — you add them per project via `cargo add` and they get listed in your `Cargo.toml`.

### Tokio — Async Runtime

Tokio is the most widely used async runtime in Rust. You need it any time you write `async`/`await` code.

```bash
cargo add tokio --features full
```

This adds the following to your `Cargo.toml`:
```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

**When you need it:** web servers, HTTP clients, database queries, networking, anything involving concurrent I/O.

**When you don't need it:** learning Rust basics (ownership, borrowing, structs, traits, iterators). Tokio is an intermediate-to-advanced topic — get comfortable with the language first.

Basic usage example:
```rust
#[tokio::main]
async fn main() {
    println!("Running on Tokio!");
}
```

---

### Other Frequently Used Crates

These are not required for setup but are worth knowing as you start building projects:

| Crate | `cargo add` command | What it does |
|---|---|---|
| `serde` | `cargo add serde --features derive` | Serialize/deserialize data (JSON, TOML, etc.) |
| `serde_json` | `cargo add serde_json` | JSON parsing and generation |
| `reqwest` | `cargo add reqwest --features json` | HTTP client (needs Tokio) |
| `axum` | `cargo add axum` | Web framework built on Tokio |
| `sqlx` | `cargo add sqlx --features runtime-tokio,sqlite` | Async SQL (Postgres, MySQL, SQLite) |
| `clap` | `cargo add clap --features derive` | CLI argument parsing |
| `anyhow` | `cargo add anyhow` | Simplified error handling |
| `thiserror` | `cargo add thiserror` | Custom error types |
| `tracing` | `cargo add tracing` | Structured logging and diagnostics |
| `rayon` | `cargo add rayon` | Easy data parallelism (no async needed) |

### How to Add a Crate to a Project

```bash
# Navigate to your project first
cd my_project

# Add a crate (requires cargo-edit from section 3)
cargo add tokio --features full

# Remove a crate
cargo rm some_crate

# Search crates.io for a package
cargo search keyword
```

You can also browse crates at [crates.io](https://crates.io) and [docs.rs](https://docs.rs) for documentation.

### crates.nvim — Crate Management Inside LazyVim

The Rust extra you installed includes `crates.nvim`, which shows crate version info inline in `Cargo.toml` and lets you update them without leaving Neovim:

| Key (in Cargo.toml) | Action |
|---|---|
| `<leader>cu` | Update crate under cursor |
| `<leader>ca` | Update all crates |
| `<leader>cv` | Show available versions |
| `<leader>cd` | Open docs.rs for crate |
| `K` | Show crate popup with version info |

---

## 11. Quick Reference Cheat Sheet

### System installs
```bash
# Rust toolchain
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add rust-analyzer rustfmt clippy

# Cargo tools
cargo install cargo-watch cargo-edit cargo-nextest cargo-expand cargo-audit

# Neovim dependencies
sudo apt install ripgrep fd-find nodejs   # adjust for your distro
```

### LazyVim config (one file)
```lua
-- ~/.config/nvim/lua/plugins/rust.lua
return {
  { import = "lazyvim.plugins.extras.lang.rust" },
}
```

### Mason packages (auto-installed, or manual)
```
:MasonInstall rust-analyzer codelldb taplo
```

### Common cargo commands
```bash
cargo new my_project       # create new project
cargo build                # compile
cargo run                  # compile and run
cargo test                 # run tests
cargo clippy               # lint
cargo fmt                  # format
cargo doc --open           # build and open docs
cargo audit                # check for vulnerabilities
```

---

**What to learn next:** Start with [The Rust Book](https://doc.rust-lang.org/book/) (free online). Once you're comfortable with ownership and borrowing, explore [Rustlings](https://github.com/rust-lang/rustlings) for hands-on exercises.
