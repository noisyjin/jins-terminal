# terminal-setup Skill 规划文档

> 供 skill-creator 插件参考，用于生成 `terminal-setup` 技能

---

## 1. 目标

创建一个项目级 skill（`.claude/skills/terminal-setup/SKILL.md`），指导 Claude Code agent 自动化安装和配置 `iTerm2useful.md` 中描述的 macOS 终端环境。

---

## 2. Skill 定位

| 属性 | 值 |
|------|-----|
| 名称 | `terminal-setup` |
| 位置 | 项目级 `.claude/skills/terminal-setup/SKILL.md` |
| 触发词 | `setup terminal` / `配置终端环境` / `初始化开发环境` / `install dev tools` |
| 目标环境 | macOS (Apple Silicon / Intel) |
| 运行模式 | 自动检测环境 → 全新走全量安装，已有走增量补装 |
| iTerm2 GUI | 不自动化，仅输出手动配置指引 |

---

## 3. 需要的 allowed-tools

```
Bash, Read, Write, Edit, Glob, Grep, Agent, AskUserQuestion
```

---

## 4. 配套模板文件

Skill 应引用 repo 中 `dotfiles/` 目录下的模板文件。需要一并创建：

```
dotfiles/
├── Brewfile              # Homebrew 包清单
├── zshrc                 # Zsh 配置模板
├── tmux.conf             # tmux 配置模板
├── starship.toml         # Starship 提示符配置
└── gitconfig             # Git 配置模板（含占位符）
```

### 4.1 Brewfile 内容

```ruby
# 终端核心
brew "tmux"
brew "neovim"

# 模糊搜索 & 导航
brew "fzf"
brew "zoxide"

# 现代 CLI 替代品
brew "bat"                # cat 替代
brew "eza"                # ls 替代
brew "fd"                 # find 替代
brew "ripgrep"            # grep 替代
brew "git-delta"          # diff 美化
brew "jq"                 # JSON 处理
brew "tlrc"               # tldr 客户端

# 文件管理 & 提示符
brew "yazi"
brew "starship"

# Git 增强
brew "gh"
brew "forgit"

# 可选: SSH 文件传输
brew "lrzsz"

# 字体 (Nerd Font)
cask "font-jetbrains-mono-nerd-font"
```

### 4.2 zshrc 模板要点

基于当前 `~/.zshrc`（118 行），需做以下处理：
- 所有硬编码用户路径替换为 `$HOME`
- JDK 相关路径和版本用注释标记 `# [CUSTOMIZE: JDK path]`
- 自定义别名（gojumpserv, goenvserv, conf2db, arthas）标记为 `# [CUSTOM] 按需启用` 并默认注释
- 保留核心结构：Zinit 插件加载、fzf/zoxide/starship 集成、bat/eza 别名、tmux 函数 (t/tls/tk/tka)、yazi 包装函数 (y)

### 4.3 tmux.conf 模板要点

基于当前 `~/.tmux.conf`（99 行），无敏感数据，原样保留：
- 前缀键 Ctrl+A、vim 风格面板导航
- Catppuccin Mocha 主题色（bg #1e1e2e, accent #89b4fa）
- 鼠标支持、escape-time 10ms、history 50000、base-index 1
- 剪贴板同步、focus-events

### 4.4 starship.toml 模板要点

基于当前 `~/.config/starship.toml`（143 行），无需参数化，原样保留：
- 两行提示符（╭─ / ╰─）
- 目录 bold cyan、git_branch bold purple、git_status 自定义符号
- Java/Node/Python 语言检测
- command_duration > 3s 显示
- 大量不需要的模块已禁用

### 4.5 gitconfig 模板要点

基于当前 `~/.gitconfig`（54 行），需参数化：
- `user.name` → `{{GIT_USER_NAME}}`
- `user.email` → `{{GIT_USER_EMAIL}}`
- 保留：delta pager + Catppuccin Mocha、side-by-side、行号、navigate
- 保留：LFS 高性能配置（32 并发、3s 超时）
- 保留：core 设置（quotepath=false、autocrlf=input、untrackedcache=true）

---

## 5. SKILL.md 执行流程设计

### Phase 0: 参数收集

通过 `AskUserQuestion` 询问：
1. Git user.name 和 user.email（用于 gitconfig 占位符替换）
2. 是否需要 JDK 环境（若需要，确认安装版本）
3. 确认是否继续（展示将安装的组件列表）

### Phase 1: 环境检测

执行检测命令，输出状态矩阵：

| 检测项 | 命令 |
|--------|------|
| 架构 | `uname -m` |
| Homebrew | `which brew` |
| 核心工具 | `which tmux nvim fzf zoxide bat eza fd rg delta jq yazi starship gh` |
| Zinit | `test -d ~/.local/share/zinit/` |
| 现有 dotfiles | `ls ~/.zshrc ~/.tmux.conf ~/.config/starship.toml ~/.gitconfig 2>/dev/null` |
| Nerd Font | `ls ~/Library/Fonts/*JetBrains*Nerd* 2>/dev/null` |

输出：**缺失组件清单** + **已存在配置文件清单**（后者涉及备份策略）

### Phase 2: Homebrew + 包安装

1. 若 Homebrew 未安装 → 执行官方安装脚本
2. `brew tap homebrew/cask-fonts`（字体 cask tap）
3. `brew bundle --file=<repo>/dotfiles/Brewfile` 批量安装
4. `fzf --version && zoxide --version && ...` 逐项验证

### Phase 3: Zinit 安装

若 `~/.local/share/zinit/` 不存在：
```bash
bash -c "$(curl --fail --show-error --silent --location https://raw.githubusercontent.com/zdharma-continuum/zinit/HEAD/scripts/install.sh)"
```
> 插件本身无需手动安装，首次 `source ~/.zshrc` 时 Zinit 自动拉取

### Phase 4: Dotfiles 部署

对每个文件执行 **备份 → 替换/写入** 流程：

| 模板源 | 目标路径 | 是否需要目录创建 |
|--------|---------|-----------------|
| `dotfiles/zshrc` | `~/.zshrc` | 否 |
| `dotfiles/tmux.conf` | `~/.tmux.conf` | 否 |
| `dotfiles/starship.toml` | `~/.config/starship.toml` | 可能需要 `mkdir -p ~/.config` |
| `dotfiles/gitconfig` | `~/.gitconfig` | 否 |

**备份策略**: 若目标已存在 → `cp <target> ~/.dotfiles_backup/<date>/<filename>`

**增量模式**: 若用户选择不全量替换 `.zshrc` 或 `.gitconfig`：
- 读取现有文件内容
- 对比模板，提示用户哪些段落缺失
- 仅追加缺失部分（如 Zinit 段、bat/eza 别名段等）

### Phase 5: 验证清单

```bash
# 工具版本
tmux -V && nvim --version | head -1 && bat --version && eza --version
fzf --version && zoxide --version && starship --version
rg --version | head -1 && fd --version && delta --version | head -1
jq --version && yazi --version && gh --version

# Git pager
git config --global core.pager  # 应输出 "delta"

# 字体
ls ~/Library/Fonts/*JetBrains*Nerd* | head -1

# Zinit
test -d ~/.local/share/zinit && echo "Zinit OK"
```

### Phase 6: iTerm2 手动配置指引

输出以下需在 iTerm2 GUI 中手动设置的项目：

1. **Profiles → Text → Font**: JetBrainsMono Nerd Font, 13pt
2. **Profiles → Terminal → Terminal Type**: `xterm-256color`
3. **Profiles → Terminal → Scrollback Lines**: 10,000
4. **Profiles → Terminal → Enable mouse reporting**: 勾选
5. **Profiles → Colors**: 暗色背景 `#14191f`, 前景 `#dcdcdc`
6. **Profiles → Advanced → Triggers**:
   - Zmodem 发送: `rz waiting to receive.*B0100` → Run Coprocess `iterm2-send-zmodem.sh`
   - Zmodem 接收: `\*\*B00000000000000` → Run Coprocess `iterm2-recv-zmodem.sh`
   - 错误高亮: `ERROR|Exception|FATAL` → Highlight Text (红字黄底)
7. **General → AI → API Key**: 配置 GPT API Key（用于 AI Compose）

---

## 6. 关键设计决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 配置文件策略 | 模板文件 + 占位符 | 比纯指令生成更可靠、可版本管理 |
| 包管理 | Brewfile + `brew bundle` | 幂等、声明式，重复执行安全 |
| 备份策略 | 日期目录备份 | 非破坏性，可回滚 |
| iTerm2 GUI | 手动指引 | plist 修改脆弱且版本敏感 |
| Zinit 插件安装 | 依赖 .zshrc 自动拉取 | Zinit 原生机制，无需额外脚本 |
| 增量 vs 全量 | 自动检测 + 用户确认 | 避免覆盖用户已有定制 |

---

## 7. 参考文件

- `iTerm2useful.md` — 完整的终端环境文档（1096 行）
- 当前 `~/.zshrc` (118 行)、`~/.tmux.conf` (99 行)、`~/.config/starship.toml` (143 行)、`~/.gitconfig` (54 行)
