# macOS Development Environment

A personal, reproducible setup for my macOS terminal, shell, and CLI tooling.

**Install order:** Homebrew → font → Ghostty → zsh → Oh My Zsh → plugins → Starship → CLI tools

---

## Homebrew

Prerequisite for everything below.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to the PATH (Apple Silicon):

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

---

## Font — Maple Mono NF CN

The `NF CN` build bundles Nerd Font icons (so Starship and prompt glyphs render correctly) and Chinese glyphs with proper CJK width alignment.

```bash
brew install --cask font-maple-mono-nf-cn
```

---

## Ghostty

Terminal emulator. Reopen Ghostty after installing the font so `font-family` picks it up.

```bash
brew install --cask ghostty
```

Configuration lives at `~/.config/ghostty/config`. Comments must be on their own line — a trailing comment after a value is parsed as part of the value and rejected.

```ini
# =============================
#  Ghostty configuration
#  Comments must be on their own line
# =============================

# --- Typography ---
# Terminal font (Nerd Font + Chinese)
font-family = "Maple Mono NF CN"
# Font size
font-size = 18

# --- Window / Appearance ---
# Restore window and split layout on restart
window-save-state = always
# Follow the system light/dark theme
window-theme = auto
# Dim unfocused splits slightly to highlight the active one
unfocused-split-opacity = 0.9

# --- Mouse / Clipboard ---
# Hide the mouse cursor while typing
mouse-hide-while-typing = true
# Copy to the clipboard on selection
copy-on-select = clipboard

# --- Compatibility / Performance ---
# Remote SSH: servers often lack the xterm-ghostty terminfo; xterm-256color is universal
term = xterm-256color
# Scrollback buffer, 25 MB
scrollback-limit = 25000000
```

Reload the config in Ghostty with `cmd+shift+,`.

---

## Shell — zsh + Oh My Zsh + Starship

zsh with plugins for the interactive experience, Starship for the prompt: cross-shell, single config file, trivial to reproduce.

### zsh

macOS ships zsh by default (Catalina and later), so there is usually nothing to install.

```bash
zsh --version
echo $SHELL            # expect /bin/zsh; otherwise run: chsh -s /bin/zsh
```

For a newer build (optional): `brew install zsh && chsh -s /opt/homebrew/bin/zsh`

### Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

The installer rewrites `.zshrc` and backs up the previous one to `.zshrc.pre-oh-my-zsh`. On a machine with an existing config, back up `.zshrc` first.

### Plugins

```bash
ZSH_CUSTOM=${ZSH_CUSTOM:-~/.oh-my-zsh/custom}
git clone https://github.com/zsh-users/zsh-autosuggestions     $ZSH_CUSTOM/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-completions          $ZSH_CUSTOM/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-syntax-highlighting  $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
```

Enable them in `.zshrc`:

```zsh
plugins=(git zsh-autosuggestions zsh-completions zsh-syntax-highlighting)
```

### Starship

```bash
brew install starship
# Catppuccin Powerline preset, matches the Ghostty Catppuccin Mocha theme
starship preset catppuccin-powerline -o ~/.config/starship.toml
```

In `.zshrc`:

```zsh
# Leave the Oh My Zsh theme empty; Starship owns the prompt
ZSH_THEME=""
# ... after Oh My Zsh is sourced
eval "$(starship init zsh)"
```

Swap presets with `starship preset <name> -o ~/.config/starship.toml` (`starship preset --list` for the full set). Popular choices: gruvbox-rainbow, catppuccin-powerline, tokyo-night, pastel-powerline. Every preset needs a Nerd Font, already covered by Maple Mono NF CN.

---

## CLI Tools

Core utilities installed on every machine. Language toolchains, cloud-native, and load-testing tools are installed on demand and not listed here.

```bash
brew install zoxide fzf yazi tmux lazygit gh tree wget helix glow
```

| Tool | Purpose | Notes |
|---|---|---|
| `zoxide` | Smart directory jumping | Replaces `cd` (see shell integration) |
| `fzf` | Fuzzy finder | Needs shell integration |
| `yazi` | Terminal file manager | Optional previews: `poppler ffmpegthumbnailer fd ripgrep` |
| `tmux` | Session multiplexing and persistence | Mainly for remote SSH — survives disconnects, which Ghostty's local splits cannot |
| `lazygit` | Git TUI | |
| `gh` | GitHub CLI | Authenticate once: `gh auth login` |
| `tree` | Directory tree | |
| `wget` | Downloader | macOS ships only `curl` |
| `helix` | Modal editor | Command is `hx` |
| `glow` | Markdown renderer | `glow -p file.md` to page; headings use color and weight, not larger fonts (terminal cells are fixed-size) |

### Shell integration

Add to `.zshrc`:

```zsh
# zoxide takes over cd
eval "$(zoxide init zsh --cmd cd)"
# fzf keybindings and completion (Ctrl-R history, Ctrl-T files)
eval "$(fzf --zsh)"
```

yazi, tmux, and lazygit work out of the box with no custom config. For yazi's "cd to the last directory on quit", add the `y` wrapper function from the yazi docs.

### Helix

Helix carries custom config that should be copied along.

`~/.config/helix/config.toml`

```toml
theme = "onedark"
```

`~/.config/helix/languages.toml` — format Go files with goimports on save:

```toml
[[language]]
name = "go"
formatter = { command = "goimports" }
auto-format = true
```

`goimports` is a Go tool (not Homebrew) and must be installed separately, otherwise the auto-format above is a no-op:

```bash
go install golang.org/x/tools/cmd/goimports@latest
```
