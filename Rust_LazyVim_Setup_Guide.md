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
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

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

## 10. Quick Reference Cheat Sheet

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
