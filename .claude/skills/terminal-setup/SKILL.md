---
name: terminal-setup
description: >
  macOS 终端开发环境自动化安装与配置。安装 iTerm2 生态工具链（tmux, neovim, fzf, zoxide, bat, eza, starship 等），
  部署 Zsh/tmux/Starship/Git 配置文件，并输出 iTerm2 GUI 手动配置指引。
  当用户提到 setup terminal、配置终端环境、初始化开发环境、install dev tools、装终端工具 时使用此技能。
---

# Terminal Setup Skill

自动化安装和配置 macOS 终端开发环境。基于 iTerm2 + tmux + Zsh(Zinit) + Starship + Neovim + 现代 CLI 工具链。

> **注意**: 本 skill 的模板文件位于同项目的 `dotfiles/` 目录下。执行时通过相对路径引用。

---

## 执行总览

```
Phase 0: 参数收集  → 询问 Git 信息、JDK 需求、确认安装清单
Phase 1: 环境检测  → 检测已有工具和配置文件，输出状态矩阵
Phase 2: 包安装    → Homebrew + brew bundle 批量安装
Phase 3: Zinit     → 安装 Zinit 插件管理器
Phase 4: 部署配置  → 备份旧文件 → 写入模板配置
Phase 5: 验证      → 逐项验证安装结果
Phase 6: 手动指引  → 输出 iTerm2 GUI 配置步骤
```

---

## Phase 0: 参数收集

通过 `AskUserQuestion` 收集以下信息：

1. **Git 用户信息**: `user.name` 和 `user.email`（用于 gitconfig 模板占位符替换）
2. **JDK 需求**: 是否需要 Java 开发环境？如需要，确认安装版本
3. **确认安装清单**: 展示将要安装的组件列表，用户确认后继续

---

## Phase 1: 环境检测

运行以下检测，输出状态矩阵（已安装 / 缺失）：

```bash
# 系统架构
uname -m

# Homebrew
command -v brew

# 核心工具（逐个检测）
for cmd in tmux nvim fzf zoxide bat eza fd rg delta jq yazi starship gh forgit; do
    command -v "$cmd" && echo "$cmd: installed" || echo "$cmd: MISSING"
done

# Zinit
test -d "${XDG_DATA_HOME:-$HOME/.local/share}/zinit/" && echo "Zinit: installed" || echo "Zinit: MISSING"

# 现有配置文件
for f in ~/.zshrc ~/.tmux.conf ~/.config/starship.toml ~/.gitconfig; do
    test -f "$f" && echo "$f: EXISTS (需备份)" || echo "$f: not found"
done

# Nerd Font
ls ~/Library/Fonts/*JetBrains*Nerd* 2>/dev/null && echo "Nerd Font: installed" || echo "Nerd Font: MISSING"
```

输出两个清单：
- **缺失组件清单**: 需要安装的工具
- **已存在配置文件清单**: 涉及备份策略的文件

---

## Phase 2: Homebrew + 包安装

### 2.1 安装 Homebrew（如未安装）

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装后根据架构配置 PATH：
- Apple Silicon: `eval "$(/opt/homebrew/bin/brew shellenv)"`
- Intel: `eval "$(/usr/local/bin/brew shellenv)"`

### 2.2 批量安装工具

使用项目中的 Brewfile：

```bash
brew bundle --file="<repo_root>/dotfiles/Brewfile"
```

`brew bundle` 是幂等的，重复运行安全。

### 2.3 验证安装

```bash
for cmd in tmux nvim fzf zoxide bat eza fd rg delta jq yazi starship gh; do
    command -v "$cmd" >/dev/null && echo "OK: $cmd" || echo "FAIL: $cmd"
done
```

---

## Phase 3: Zinit 安装

检测 `${XDG_DATA_HOME:-$HOME/.local/share}/zinit/` 是否存在。如不存在：

```bash
bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh)"
```

Zinit 插件无需手动安装，首次 `source ~/.zshrc` 时 Zinit 会自动拉取所有在 zshrc 中声明的插件。

---

## Phase 4: Dotfiles 部署

### 4.1 备份策略

对每个已存在的目标文件，先备份到日期目录：

```bash
backup_dir="$HOME/.dotfiles_backup/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$backup_dir"

for f in .zshrc .tmux.conf .gitconfig; do
    [ -f "$HOME/$f" ] && cp "$HOME/$f" "$backup_dir/$f"
done
[ -f "$HOME/.config/starship.toml" ] && cp "$HOME/.config/starship.toml" "$backup_dir/starship.toml"
```

### 4.2 部署配置文件

| 模板源 (repo 内) | 目标路径 | 备注 |
|---|---|---|
| `dotfiles/zshrc` | `~/.zshrc` | — |
| `dotfiles/tmux.conf` | `~/.tmux.conf` | — |
| `dotfiles/starship.toml` | `~/.config/starship.toml` | 可能需要 `mkdir -p ~/.config` |
| `dotfiles/gitconfig` | `~/.gitconfig` | 需替换占位符 |

**gitconfig 占位符替换**：
- 将 `{{GIT_USER_NAME}}` 替换为用户在 Phase 0 提供的 name
- 将 `{{GIT_USER_EMAIL}}` 替换为用户在 Phase 0 提供的 email

**JDK 配置**（如用户在 Phase 0 表示需要）：
- 在 zshrc 中取消 JDK 相关注释行
- 修改路径为用户提供的实际 JDK 路径

### 4.3 增量模式

如果用户选择**不全量替换** `.zshrc` 或 `.gitconfig`：
1. 读取现有文件内容
2. 与模板对比，提示用户哪些段落缺失
3. 仅追加缺失部分（如 Zinit 插件段、bat/eza 别名段等）

部署前用 `AskUserQuestion` 确认用户选择全量替换还是增量追加。

---

## Phase 5: 验证清单

```bash
# 工具版本
tmux -V
nvim --version | head -1
bat --version
eza --version
fzf --version
zoxide --version
starship --version
rg --version | head -1
fd --version
delta --version | head -1
jq --version
yazi --version
gh --version | head -1

# Git pager 配置
git config --global core.pager  # 应输出 "delta"

# Nerd Font
ls ~/Library/Fonts/*JetBrains*Nerd* | head -1

# Zinit
test -d "${XDG_DATA_HOME:-$HOME/.local/share}/zinit" && echo "Zinit: OK"
```

如有失败项，输出修复建议。

---

## Phase 6: iTerm2 手动配置指引

以下设置需在 iTerm2 GUI 中手动配置，自动化不可靠（plist 修改脆弱且版本敏感）：

### 字体
- **Profiles > Text > Font**: `JetBrainsMono Nerd Font`, 13pt

### 终端
- **Profiles > Terminal > Terminal Type**: `xterm-256color`
- **Profiles > Terminal > Scrollback Lines**: `10,000`
- **Profiles > Terminal > Enable mouse reporting**: 勾选

### 配色
- **Profiles > Colors**: 暗色背景 `#14191f`, 前景 `#dcdcdc`

### Triggers（高级）
- **Profiles > Advanced > Triggers**:

| 正则表达式 | 动作 | 参数 |
|---|---|---|
| `rz waiting to receive.*B0100` | Run Coprocess | `iterm2-send-zmodem.sh` |
| `\*\*B00000000000000` | Run Coprocess | `iterm2-recv-zmodem.sh` |
| `ERROR\|Exception\|FATAL` | Highlight Text | 红字黄底 |

### AI 功能
- **General > AI > API Key**: 配置 GPT API Key（用于 AI Compose，可选）

---

## 关键设计决策

| 决策 | 选择 | 理由 |
|---|---|---|
| 配置文件策略 | 模板文件 + 占位符 | 比纯指令更可靠，可版本管理 |
| 包管理 | Brewfile + `brew bundle` | 幂等、声明式，重复执行安全 |
| 备份策略 | 日期目录备份 | 非破坏性，可回滚 |
| iTerm2 GUI | 手动指引 | plist 修改脆弱且版本敏感 |
| Zinit 插件 | 依赖 .zshrc 自动拉取 | Zinit 原生机制，无需额外脚本 |
| Homebrew 路径 | `$(brew --prefix)` 动态获取 | 兼容 Apple Silicon 和 Intel |
| 增量 vs 全量 | 自动检测 + 用户确认 | 避免覆盖用户已有定制 |
