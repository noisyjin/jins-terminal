# iTerm2 终端环境使用手册

> 最后更新: 2026-03-26
> 环境: macOS Sequoia 15.7 (Apple Silicon) / iTerm2 3.6.x / Zsh 5.9

---

## 目录

- [1. 架构总览](#1-架构总览)
- [2. iTerm2 终端](#2-iterm2-终端)
- [3. iTerm2 高级功能](#3-iterm2-高级功能)
- [4. tmux 终端复用](#4-tmux-终端复用)
- [5. Zsh + Zinit 插件体系](#5-zsh--zinit-插件体系)
- [6. Starship 提示符](#6-starship-提示符)
- [7. Neovim (LazyVim)](#7-neovim-lazyvim)
- [8. Yazi 文件管理器](#8-yazi-文件管理器)
- [9. CLI 效率工具集](#9-cli-效率工具集)
- [10. Git 工具链](#10-git-工具链)
- [11. 自定义别名与函数速查](#11-自定义别名与函数速查)
- [12. 配置文件索引](#12-配置文件索引)
- [13. 常见问题排查](#13-常见问题排查)

---

## 1. 架构总览

```
┌──────────────────────────────────────────────────────┐
│                     iTerm2                           │
│  ┌────────────────────────────────────────────────┐  │
│  │  tmux (-CC 模式, 原生标签页集成)               │  │
│  │  ┌──────────────────────────────────────────┐  │  │
│  │  │  Zsh Shell                               │  │  │
│  │  │  ├─ Zinit (插件管理, Turbo Mode)         │  │  │
│  │  │  ├─ Starship (提示符)                    │  │  │
│  │  │  ├─ fzf (模糊搜索)                      │  │  │
│  │  │  ├─ zoxide (智能目录跳转)                │  │  │
│  │  │  ├─ bat / eza (现代 cat/ls)              │  │  │
│  │  │  └─ forgit (Git 交互 TUI)               │  │  │
│  │  ├──────────────────────────────────────────┤  │  │
│  │  │  Neovim (LazyVim) / Yazi 文件管理器      │  │  │
│  │  └──────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────┘  │
│  字体: JetBrainsMono Nerd Font 13                    │
│  配色: 自定义暗色 + Catppuccin Mocha (tmux)          │
└──────────────────────────────────────────────────────┘
```

**核心数据流**:
- iTerm2 作为 GUI 外壳, 提供窗口管理、快捷键、字体渲染
- tmux 通过 `-CC` 控制模式与 iTerm2 深度集成, 会话可持久化
- Zsh 是实际的命令解释器, Zinit 管理所有插件
- Starship 渲染提示符, 各 CLI 工具提供增强功能

---

## 2. iTerm2 终端

### 2.1 配置文件 (Profiles)

| 配置文件 | 字体 | 用途 |
|----------|------|------|
| **Default** | JetBrainsMono Nerd Font 13 | 日常终端使用 |
| **tmux** | Monaco 12 | tmux 集成模式 |
| **webrowser** | Monaco 12 | 终端内浏览器模式 |

### 2.2 全局快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + Tab` | 循环切换标签页 |
| `Ctrl + ← / →` | 上一个 / 下一个标签页 |
| `Ctrl + Shift + ← / →` | 左移 / 右移当前标签页 |
| `Ctrl + ↑` | 滚动到顶部 |
| `Ctrl + ↓` | 滚动到底部 |
| `Cmd + Home / End` | 滚动到顶 / 底 (备用) |
| `Cmd + PageUp / PageDown` | 向上 / 向下翻页 |
| `Shift + Tab` | 撤销 |
| `Cmd + Ctrl + B` | 打开 webrowser 配置 |

### 2.3 鼠标 / 触控板手势

| 手势 | 功能 |
|------|------|
| 右键单击 | 上下文菜单 |
| 中键单击 | 粘贴剪贴板内容 |
| 三指左 / 右滑 | 切换标签页 |
| 三指上 / 下滑 | 切换窗口 |

### 2.4 触发器 (Triggers)

自动化规则, 当终端输出匹配特定模式时触发动作:

| 触发器 | 匹配模式 | 动作 | 场景 |
|--------|---------|------|------|
| Zmodem 发送 | `rz waiting to receive.*B0100` | 调用 `iterm2-send-zmodem.sh` | SSH 中 `rz` 上传文件到远程 |
| Zmodem 接收 | `*B00000000000000` | 调用 `iterm2-recv-zmodem.sh` | SSH 中 `sz` 从远程下载文件 |
| 错误高亮 | `ERROR\|Exception\|FATAL` | 红字黄底高亮 | 查看日志时快速定位异常 |

**Zmodem 使用方法** (需要远程服务器安装 `lrzsz`):
```bash
# 在 SSH 连接中:
rz          # 上传: 远程执行后弹出本地文件选择窗口
sz file.log # 下载: 远程执行后弹出本地保存路径选择
```

### 2.5 Shell Integration (未启用)

文件 `~/.iterm2_shell_integration.zsh` 存在于磁盘，但**未被任何配置文件 source，当前未生效**。

Shell Integration 可提供的功能:
- 命令标记 (`Cmd+Shift+↑/↓` 跳转命令)
- 命令状态可视化
- 自动下载/上传 (拖拽文件)
- 当前目录和主机名上报

**如需启用**, 在 `~/.zshrc` 末尾添加:
```bash
source ~/.iterm2_shell_integration.zsh
```
或通过 iTerm2 菜单: Install Shell Integration (会自动添加)。

> 注: Starship 和 Zinit 已覆盖大部分提示符功能，启用 Shell Integration 主要价值在于命令标记导航和拖拽上传。

### 2.6 AI 终端功能

iTerm2 内置 AI 助手 (已配置 GPT-5.2):
- `Cmd + Shift + .` 打开 AI Compose (自然语言生成命令)
- 选中终端输出后可请求 AI 解释

### 2.7 配色方案

支持 Light/Dark 双模式自动切换:
- **暗色模式**: 背景 `#14191f`, 前景 `#dcdcdc`
- **亮色模式**: 背景 `#fafafa`, 前景 `#101010`

### 2.8 其他重要设置

| 设置 | 值 | 说明 |
|------|-----|------|
| 终端类型 | `xterm-256color` | 256色支持 |
| 回滚缓冲区 | 10,000 行 | 可查看历史输出 |
| 鼠标报告 | 开启 | 终端内应用可响应鼠标 |
| 视觉响铃 | 开启 | 闪烁代替蜂鸣 |

---

## 3. iTerm2 高级功能

### 3.1 即时回放 (Instant Replay)

iTerm2 在后台持续记录终端屏幕帧，你可以"回放"终端画面——像视频倒带一样查看过去的终端内容。

**快捷键**: `Cmd + Opt + B`

| 操作 | 按键 |
|------|------|
| 进入即时回放模式 | `Cmd + Opt + B` |
| 后退 | `← / Shift + ←` (快退) |
| 前进 | `→ / Shift + →` (快进) |
| 退出回放 | `Esc` |

**典型场景**:
- 快速刷过的编译输出来不及看——回放找到关键错误
- `clear` 后想看之前的输出——回放回去
- 滚动缓冲区已被覆盖——回放仍可找到

> 回放缓冲区默认在内存中保留 4MB，可在 Settings → General → Instant Replay 中调整。

### 3.2 广播输入 (Broadcast Input)

将键盘输入同时发送到**多个面板或标签页**——适合需要在多台服务器上同步执行相同命令的场景。

| 操作 | 菜单路径 | 快捷键 |
|------|---------|--------|
| 广播到当前标签页的所有面板 | Shell → Broadcast Input → Broadcast Input to All Panes in Current Tab | `Cmd + Opt + I` |
| 广播到所有标签页的所有面板 | Shell → Broadcast Input → Broadcast Input to All Panes in All Tabs | — |
| 切换单个面板的广播状态 | Shell → Broadcast Input → Toggle Broadcast Input to Current Session | — |
| 关闭广播 | Shell → Broadcast Input → Send Input to Current Session Only | `Cmd + Opt + I` (再次按) |

**典型场景**:
```
1. 打开 4 个面板，分别 SSH 到 4 台服务器
2. Cmd+Opt+I 开启广播
3. 输入命令（如 tail -f /var/log/app.log）同时在 4 台执行
4. Cmd+Opt+I 关闭广播，恢复独立输入
```

> 开启广播后，面板边框会变为**高亮颜色**提示当前处于广播模式。

### 3.3 密码管理器 (Password Manager)

安全存储常用密码，在终端提示时快速填充。

**打开方式**: `Cmd + Opt + F` 或菜单 Window → Password Manager

| 功能 | 说明 |
|------|------|
| 添加密码 | 点击 `+`，输入账户名和密码 |
| 使用密码 | 在终端出现密码提示时，双击对应条目自动输入 |
| 存储位置 | macOS Keychain（与系统钥匙串集成） |

**场景**: 频繁 SSH 到需要密码认证的服务器（不支持密钥的跳板机等）。

### 3.4 Annotations (批注)

在终端输出的任意位置添加文字批注——类似在代码上做标注。

| 操作 | 方法 |
|------|------|
| 添加批注 | 选中文本 → 右键 → Annotate Selection，或 `Cmd + Shift + A` |
| 查看批注 | 鼠标悬停在有批注的文本上 |
| 浏览所有批注 | View → Show Annotations |
| 删除批注 | 右键批注 → Remove Annotation |

**场景**: 排查长日志时标记关键发现点，方便后续回顾。

### 3.5 捕获输出 (Captured Output / 命令输出工具栏)

自动识别终端输出中的文件名和行号 (如编译错误 `src/Main.java:42`)，汇总到工具栏中可直接点击跳转。

**打开方式**: View → Show Captured Output 或 `Cmd + Shift + P`（需启用 Shell Integration）

### 3.6 触发器高级用法 (Triggers)

已配置的触发器见 [2.4 节](#24-触发器-triggers)。触发器还支持更多动作类型:

| 动作类型 | 说明 | 场景 |
|----------|------|------|
| **Highlight Text** | 匹配文本高亮 | 日志中标红 ERROR (已配置) |
| **Show Alert** | 弹出系统通知 | 长任务完成时通知 |
| **Send Text** | 自动输入文本 | 自动回复 "yes/no" 确认提示 |
| **Run Command** | 执行 shell 命令 | 匹配到特定模式时自动执行动作 |
| **Coprocess** | 启动协同进程 | Zmodem 文件传输 (已配置) |
| **Set Title** | 修改标签页标题 | SSH 到不同服务器时自动改标题 |
| **Stop Processing Triggers** | 停止后续触发器 | 避免规则冲突 |

**添加触发器**: Settings → Profiles → Advanced → Triggers → Edit

**实用示例 — 部署完成通知**:
```
正则: deploy.*success|部署.*完成
动作: Show Alert
参数: 部署完成!
```

### 3.7 Snippets (代码片段)

保存常用命令片段, 快捷调用:

**打开方式**: 菜单 Toolbelt → Snippets, 或 `Cmd + Shift + 9`

| 操作 | 说明 |
|------|------|
| 添加片段 | 打开 Toolbelt → Snippets → `+` |
| 使用片段 | 点击对应片段直接发送到终端 |

**场景**: 保存常用的 SSH 隧道命令、复杂的 awk/sed 管道等不值得建别名但偶尔需要的长命令。

### 3.8 Profile 自动切换

iTerm2 可以根据条件自动切换 Profile (字体、配色等):

**配置路径**: Settings → Profiles → Advanced → Automatic Profile Switching

| 条件 | 说明 |
|------|------|
| 用户名@主机名 | SSH 到不同服务器时自动切换配色 |
| 路径匹配 | 进入特定目录时切换 Profile |

**场景**: SSH 到生产环境时自动切换为**红色背景** Profile，防止误操作。需要 Shell Integration 配合生效。

### 3.9 选区与搜索增强

| 快捷键 | 功能 |
|--------|------|
| `Cmd + F` | 终端内搜索（支持正则） |
| `Cmd + Shift + F` | 全局搜索所有标签页 |
| `Cmd + 单击` | URL/文件路径可点击打开 |
| `双击` | 选中一个单词 |
| `三击` | 选中整行 |
| `Cmd + 拖动` | 矩形选择（列选择模式） |
| `Cmd + Opt + 拖动` | 不连续多段选择 |
| `选中 → Cmd + K` | 将选中内容创建为代码片段 |

### 3.10 分屏与布局

iTerm2 原生分屏（不依赖 tmux）:

| 快捷键 | 功能 |
|--------|------|
| `Cmd + D` | 水平分屏 (左右) |
| `Cmd + Shift + D` | 垂直分屏 (上下) |
| `Cmd + ]` / `Cmd + [` | 切换到下一个/上一个面板 |
| `Cmd + Opt + 方向键` | 移动到指定方向的面板 |
| `Cmd + Shift + Enter` | 当前面板全屏（再按恢复） |
| `Cmd + Shift + Opt + 拖动` | 拖拽调整面板大小 |

> 注: 使用 tmux -CC 集成模式时，tmux 的分屏会映射为 iTerm2 原生分屏，两者操作一致。

### 3.11 其他实用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + T` | 新建标签页 |
| `Cmd + W` | 关闭当前标签页/面板 |
| `Cmd + N` | 新建窗口 |
| `Cmd + Enter` | 全屏切换 |
| `Cmd + /` | 高亮显示光标位置（闪烁定位） |
| `Cmd + ;` | 自动补全 (基于终端历史) |
| `Cmd + Shift + H` | 粘贴历史 (Paste History) |
| `Cmd + Shift + ;` | 命令历史弹窗 |
| `Cmd + Shift + .` | AI Compose (自然语言生成命令) |
| `Cmd + Opt + E` | 打开窗口排列选择器 (Expose All Tabs) |
| `Cmd + Opt + B` | 即时回放 (Instant Replay) |
| `Cmd + Opt + I` | 广播输入开关 |
| `Cmd + Opt + F` | 密码管理器 |
| `Cmd + Shift + A` | 添加批注 |

---

## 4. tmux 终端复用

### 4.1 核心概念

tmux 管理**会话 (session) > 窗口 (window) > 面板 (pane)** 三级结构:
- **会话**: 独立的工作上下文, 断开后持续运行, 重连后恢复
- **窗口**: 会话内的标签页
- **面板**: 窗口内的分屏区域

### 4.2 iTerm2 + tmux -CC 集成模式

本机使用 `tmux -CC` 控制模式, **tmux 的窗口/面板直接映射为 iTerm2 原生标签页和分屏**, 操作体验与原生 iTerm2 完全一致, 同时享受 tmux 的会话持久化能力。

**使用方式** (已在 .zshrc 中定义快捷命令):

```bash
t             # 创建或附加名为 "main" 的会话
t work        # 创建或附加名为 "work" 的会话
tls           # 列出所有会话
tk work       # 关闭 "work" 会话
tka           # 关闭所有 tmux 会话
```

**典型工作流**:
1. `t work` 开启工作会话
2. 在 iTerm2 中正常使用标签页和分屏
3. 关闭终端窗口 (会话保持)
4. 下次 `t work` 恢复之前所有窗口和运行中的进程

### 4.3 前缀键

tmux 快捷键需要先按**前缀键**再按功能键。本机前缀键为:

```
Ctrl + A
```

下文 `prefix` 均指 `Ctrl + A`。

### 4.4 快捷键速查

#### 会话管理

| 按键 | 功能 |
|------|------|
| `prefix + d` | 脱离当前会话 (detach, 会话后台继续运行) |
| `prefix + s` | 交互式选择会话 |
| `prefix + $` | 重命名当前会话 |

#### 窗口管理

| 按键 | 功能 |
|------|------|
| `prefix + c` | 新建窗口 (保持当前目录) |
| `prefix + ,` | 重命名当前窗口 |
| `prefix + &` | 关闭当前窗口 |
| `prefix + 数字` | 跳转到第 N 个窗口 (从 1 开始) |
| `prefix + w` | 交互式选择窗口 |
| `Shift + ←` | 上一个窗口 (无需前缀) |
| `Shift + →` | 下一个窗口 (无需前缀) |

#### 面板 (分屏) 管理

| 按键 | 功能 |
|------|------|
| `prefix + \|` | 水平分屏 (左右, 保持当前目录) |
| `prefix + -` | 垂直分屏 (上下, 保持当前目录) |
| `prefix + x` | 关闭当前面板 |
| `prefix + z` | 面板全屏切换 (zoom/unzoom) |
| `prefix + h / j / k / l` | Vim 风格切换面板 (左/下/上/右) |
| `Alt + 方向键` | 切换面板 (无需前缀) |
| `prefix + 空格` | 循环切换面板布局 |

#### 其他

| 按键 | 功能 |
|------|------|
| `prefix + r` | 重新加载 tmux 配置 |
| `prefix + [` | 进入复制模式 (可用 vim 键位滚动/选择) |
| `prefix + ]` | 粘贴 tmux 缓冲区内容 |

### 4.5 关键配置说明

| 配置 | 值 | 为什么 |
|------|-----|--------|
| 前缀键 | `Ctrl+A` | 比默认 `Ctrl+B` 更顺手, 与 GNU screen 一致 |
| 历史行数 | 50,000 | 大量回滚缓冲, 不丢失长输出 |
| Escape 延迟 | 10ms | Vim/Neovim 中按 ESC 无卡顿 |
| 鼠标支持 | 开启 | 鼠标点击切换面板、滚动、调整大小 |
| 基础索引 | 1 | 窗口/面板编号从 1 开始, 符合键盘布局 |
| 剪贴板同步 | 开启 | tmux 内复制直接进 macOS 剪贴板 |
| 主题 | Catppuccin Mocha | 深色底 (`#1e1e2e`) 配蓝色高亮 (`#89b4fa`) |

### 4.6 配置文件

```
~/.tmux.conf
```

修改后执行 `prefix + r` 或 `tmux source-file ~/.tmux.conf` 重新加载。

---

## 5. Zsh + Zinit 插件体系

### 5.1 插件管理器: Zinit

Zinit 是高性能 Zsh 插件管理器, 位于 `~/.local/share/zinit/`。核心优势是 **Turbo Mode** — 插件在后台异步加载, Shell 几乎瞬间启动。

**Zinit 常用命令**:

```bash
zinit update --all     # 更新所有插件
zinit self-update      # 更新 Zinit 自身
zinit list             # 列出已安装插件
zinit delete 插件名     # 删除指定插件
zinit times            # 查看各插件加载耗时
```

### 5.2 已安装插件一览

#### fast-syntax-highlighting (语法高亮)

命令行输入时实时着色:
- **绿色**: 有效命令
- **红色**: 无效命令/不存在的命令
- **黄色/青色**: 字符串、路径
- **紫色**: 内置关键字

无需任何操作, 输入时自动生效。

#### zsh-autosuggestions (历史建议)

根据历史记录在光标后显示灰色建议:

| 按键 | 功能 |
|------|------|
| `→` (右箭头) | 接受整个建议 |
| `Ctrl + →` | 接受建议的一个单词 |
| `Ctrl + F` | 接受整个建议 (替代) |

输入越多, 建议越精确。

#### zsh-completions (扩展补全)

按 `Tab` 触发补全, 为大量命令提供参数/选项补全。支持:
- 命令名补全
- 文件路径补全 (含模糊匹配)
- Git 子命令和分支名补全
- Docker / kubectl / brew 等命令的参数补全

#### OMZ git 插件 (Git 别名)

提供大量 git 操作的缩写, 完整列表见 [第 10 节](#10-自定义别名与函数速查)。

#### OMZ 核心库

| 库 | 功能 |
|-----|------|
| `completion.zsh` | 补全系统基础设置, Tab 行为优化 |
| `history.zsh` | 历史记录配置 (大小、去重、共享) |
| `key-bindings.zsh` | Emacs 风格键绑定 (`Ctrl+A` 行首, `Ctrl+E` 行尾等) |
| `directories.zsh` | 目录栈管理 (`d` 查看栈, `1`-`9` 快速跳转) |

**目录栈用法**:
```bash
cd ~/projects     # 进入目录 (自动入栈)
cd ~/Documents    # 再进入
d                 # 显示目录栈:
                  #   0  ~/Documents
                  #   1  ~/projects
                  #   2  ~
1                 # 跳回 ~/projects
```

#### forgit (Git 交互式 TUI)

结合 fzf 的交互式 Git 操作界面:

| 命令 | 功能 |
|------|------|
| `forgit::log` 或 `glo` | 交互式查看 git log, 实时预览每个 commit 的 diff |
| `forgit::diff` 或 `gd` | 交互式查看 diff, 可选择文件 |
| `forgit::add` 或 `ga` | 交互式选择文件 add |
| `forgit::stash::show` | 交互式查看 stash |
| `forgit::cherry::pick` | 交互式 cherry-pick |
| `forgit::rebase` | 交互式 rebase |
| `forgit::checkout::branch` | 交互式切换分支 |
| `forgit::clean` | 交互式清理未跟踪文件 |

### 5.3 环境变量

```bash
JAVA_HOME=$HOME/path/to/your/jdk/Contents/Home  # 默认 JDK，按实际路径修改
EDITOR=nvim                    # 默认编辑器
VISUAL=nvim                    # 可视化编辑器
```

---

## 6. Starship 提示符

### 6.1 外观

两行式提示符, 使用 box-drawing 字符连接:

```
╭─  ~/projects/myapp  main ?1 ~2  v1.8.0  took 5s
╰─ ❯ _
```

各段含义:
- ` ~/projects/myapp` — 当前目录 (粗体青色, 截断到 3 级)
- ` main` — Git 分支名 (粗体紫色)
- `?1 ~2` — Git 状态: 1 个未跟踪文件, 2 个已修改 (粗体黄色)
- ` v1.8.0` — Java 版本 (粗体红色, 仅在 Java 项目中显示)
- ` took 5s` — 上一条命令耗时 (仅 >3 秒时显示)
- `❯` — 输入提示符 (绿色=上条成功, 红色`✗`=上条失败)

### 6.2 Git 状态符号

| 符号 | 含义 |
|------|------|
| `?N` | N 个未跟踪文件 (untracked) |
| `~N` | N 个已修改文件 (modified) |
| `+N` | N 个已暂存文件 (staged) |
| `✘N` | N 个已删除文件 (deleted) |
| `»N` | N 个重命名文件 (renamed) |
| `=` | 存在合并冲突 (conflicted) |
| `*N` | N 个 stash |
| `⇡N` | 领先远程 N 个提交 (ahead) |
| `⇣N` | 落后远程 N 个提交 (behind) |
| `⇕⇡N⇣M` | 分叉: 领先 N 落后 M (diverged) |

### 6.3 语言检测

| 语言 | 图标 | 触发条件 | 显示内容 |
|------|------|---------|---------|
| Java | ` ` (红色) | 目录含 `pom.xml` / `build.gradle` / `.java` 文件 | JDK 版本号 |
| Node.js | ` ` (绿色) | 目录含 `package.json` | Node 版本号 |
| Python | ` ` (黄色) | 目录含 `.py` 文件或虚拟环境 | Python 版本 + virtualenv 名 |

### 6.4 配置文件

```
~/.config/starship.toml
```

修改后立即生效 (下一次 prompt 渲染时自动读取)。

---

## 7. Neovim (LazyVim)

### 7.1 概述

Neovim 0.11.6 + LazyVim 预配置框架, 提供开箱即用的现代 IDE 体验。

**启动方式**:
```bash
nvim              # 打开 Neovim (首次启动自动安装插件)
nvim file.py      # 打开指定文件
nvim .            # 打开文件浏览器
```

### 7.2 核心功能

LazyVim 开箱即带:
- **LSP**: 语言服务器协议, 代码补全/跳转/重命名/诊断
- **Treesitter**: 语法高亮、代码折叠、增量选择
- **Telescope**: 模糊文件查找、全局搜索
- **Neo-tree**: 侧边文件树
- **Which-key**: 按键提示 (按下 leader 后显示可用操作)
- **自动格式化**: 保存时自动格式化代码
- **Git 集成**: gitsigns 显示行级变更

### 7.3 基础操作

LazyVim 的 Leader 键为 **空格键**。

#### 文件操作

| 按键 | 功能 |
|------|------|
| `Space + f + f` | 查找文件 (Telescope) |
| `Space + f + g` | 全局搜索内容 (grep) |
| `Space + f + r` | 最近打开的文件 |
| `Space + e` | 切换文件树 (Neo-tree) |
| `Space + b + d` | 关闭当前 buffer |
| `Space + b + b` | 切换 buffer |

#### 代码导航

| 按键 | 功能 |
|------|------|
| `g + d` | 跳转到定义 (Go to Definition) |
| `g + r` | 查找引用 (Go to References) |
| `g + I` | 跳转到实现 (Go to Implementation) |
| `K` | 悬停文档 (Hover) |
| `Space + c + a` | 代码动作 (Code Action) |
| `Space + c + r` | 重命名符号 (Rename) |
| `] + d` / `[ + d` | 下一个/上一个诊断 |

#### 窗口管理

| 按键 | 功能 |
|------|------|
| `Ctrl + h / j / k / l` | Vim 风格切换窗口 |
| `Space + -` | 水平分屏 |
| `Space + \|` | 垂直分屏 |
| `Space + w + d` | 关闭窗口 |

#### Git 操作

| 按键 | 功能 |
|------|------|
| `] + h` / `[ + h` | 下一个/上一个 hunk (变更块) |
| `Space + g + s` | Git status |
| `Space + g + b` | Git blame (当前行) |

### 7.4 插件管理

```
:Lazy              # 打开 Lazy.nvim 插件管理器 UI
:Lazy update       # 更新所有插件
:Lazy sync         # 同步 (安装缺失 + 清理多余 + 更新)
:Mason             # 打开 LSP/格式化器/调试器安装管理
```

### 7.5 自定义扩展

LazyVim 配置结构:

```
~/.config/nvim/
├── init.lua                   # 入口 (无需修改)
├── lua/
│   ├── config/
│   │   ├── autocmds.lua       # 自动命令
│   │   ├── keymaps.lua        # 自定义快捷键
│   │   ├── lazy.lua           # Lazy.nvim 配置 (无需修改)
│   │   └── options.lua        # 编辑器选项
│   └── plugins/
│       └── example.lua        # 在此添加/覆盖插件配置
```

**添加新插件示例** — 编辑 `~/.config/nvim/lua/plugins/example.lua`:
```lua
return {
  -- 示例: 添加一个新主题
  { "catppuccin/nvim", name = "catppuccin", priority = 1000 },
}
```

---

## 8. Yazi 文件管理器

### 8.1 概述

Yazi 是 Rust 编写的终端文件管理器, 以极快速度和异步 I/O 著称。

**启动方式**:
```bash
y               # 启动 Yazi, 退出后 cd 到浏览目录
y ~/projects    # 从指定目录启动
yazi            # 直接启动 (退出后不切换目录)
```

`y` 命令是 `.zshrc` 中定义的包装函数 — 退出 Yazi 后自动 `cd` 到最后浏览的目录。

### 8.2 基础操作

#### 导航

| 按键 | 功能 |
|------|------|
| `h` | 返回上级目录 |
| `j / k` | 下移 / 上移 |
| `l` 或 `Enter` | 进入目录 / 打开文件 |
| `H / L` | 向后 / 向前 (历史导航) |
| `~` | 跳转到 Home 目录 |
| `/` | 搜索文件名 |
| `g + g` | 跳到顶部 |
| `G` | 跳到底部 |

#### 文件操作

| 按键 | 功能 |
|------|------|
| `Space` | 选择/取消选择当前文件 |
| `V` | 进入可视选择模式 |
| `y` | 复制 (yank) 选中文件 |
| `x` | 剪切选中文件 |
| `p` | 粘贴到当前目录 |
| `d` | 移到回收站 |
| `D` | 永久删除 |
| `a` | 新建文件/目录 (末尾加 `/` 创建目录) |
| `r` | 重命名 |
| `c` | 批量重命名 (在编辑器中) |
| `.` | 显示/隐藏 隐藏文件 |

#### 排序与视图

| 按键 | 功能 |
|------|------|
| `, + m` | 按修改时间排序 |
| `, + n` | 按名称排序 |
| `, + s` | 按大小排序 |
| `1 / 2 / 3` | 切换视图布局 |
| `Tab` | 新建标签页 |
| `Ctrl + c` | 关闭标签页 |
| `[ / ]` | 切换标签页 |

#### 退出

| 按键 | 功能 |
|------|------|
| `q` | 退出并切换到浏览目录 (使用 `y` 命令启动时) |
| `Q` | 退出但不切换目录 |

### 8.3 预览能力

Yazi 支持在右侧面板实时预览:
- 文本文件 (语法高亮)
- 图片 (iTerm2 内联图片协议)
- PDF (首页预览)
- 目录 (文件列表)
- 压缩包 (内容列表)

### 8.4 配置文件

```
~/.config/yazi/           # 配置目录 (当前使用默认配置)
├── yazi.toml             # 主配置
├── keymap.toml           # 快捷键
└── theme.toml            # 主题
```

如需自定义, 创建上述文件即可覆盖默认值。

---

## 9. CLI 效率工具集

### 9.1 fzf — 模糊搜索

通用模糊查找工具, 集成到 Shell 中:

| 快捷键 | 功能 | 场景 |
|--------|------|------|
| `Ctrl + R` | 搜索命令历史 | 快速找到之前执行过的命令 |
| `Ctrl + T` | 搜索当前目录下的文件 | 找到文件后粘贴路径到命令行 |
| `Alt + C` | 搜索目录并 cd 进入 | 快速切换到子目录 |

**管道用法**:
```bash
# 搜索并打开文件
nvim $(fzf)

# 搜索进程并 kill
ps aux | fzf | awk '{print $2}' | xargs kill

# 搜索 git 分支并切换
git branch | fzf | xargs git checkout

# 预览文件内容
fzf --preview 'bat --color=always {}'
```

### 9.2 zoxide — 智能目录跳转

学习你的 `cd` 习惯, 跳转到最常/最近访问的目录:

```bash
z foo           # 跳转到最匹配 "foo" 的目录 (如 ~/projects/foo-app)
z foo bar       # 跳转到同时匹配 "foo" 和 "bar" 的目录
zi              # 交互式选择 (结合 fzf)
z -             # 跳回上一个目录
```

**工作原理**: 每次 `cd` 时 zoxide 自动记录目录, 按频率和最近度打分。用得越多越准。

### 9.3 bat — 语法高亮查看器 (替代 cat)

```bash
cat file.py          # 已别名到 bat --paging=never, 语法高亮输出
catp file.py         # 带分页器的 bat (长文件翻页查看)
bat --diff file.py   # 显示 git diff 标记
bat -l json file     # 指定语言高亮
bat -A file          # 显示不可见字符
```

**已设置别名**: `cat` → `bat --paging=never`, `catp` → `bat` (带分页)。

### 9.4 eza — 现代 ls (替代 ls)

```bash
ls                # 带图标的文件列表
ll                # 详细列表 + 图标 + git 状态
la                # 包含隐藏文件
tree              # 树状视图
tree -L 2         # 限制深度为 2 层
ll --sort=time    # 按时间排序
ll --sort=size    # 按大小排序
ll -T             # 详细列表 + 树形展开
```

**已设置别名**: `ls` → `eza --icons`, `ll` → `eza -la --icons --git`, `tree` → `eza --tree --icons`。

### 9.5 tldr (tlrc) — 简化版 man

```bash
tldr tar          # 查看 tar 命令常用示例
tldr git commit   # 查看 git commit 常用用法
tldr --update     # 更新 tldr 数据库
tldr --list       # 列出所有可用条目
```

显示社区维护的简明实用示例, 比 `man` 更直观。

### 9.6 ripgrep (rg) — 超快代码搜索

```bash
rg "pattern"                 # 递归搜索当前目录
rg "pattern" src/            # 搜索指定目录
rg -i "pattern"              # 忽略大小写
rg -t java "pattern"         # 只搜索 Java 文件
rg -g "*.json" "key"         # 通过 glob 过滤文件
rg -l "pattern"              # 只列出匹配的文件名
rg --hidden "pattern"        # 包含隐藏文件
rg -C 3 "pattern"            # 显示匹配行上下各 3 行
rg "TODO|FIXME" --stats      # 搜索并显示统计
```

自动忽略 `.gitignore` 中的文件, 速度极快。

### 9.7 fd — 快速文件查找

```bash
fd "pattern"                 # 模糊匹配文件名
fd -e java                   # 查找所有 .java 文件
fd -e java -x wc -l          # 统计每个 Java 文件的行数
fd -H ".env"                 # 包含隐藏文件搜索
fd -t d "config"             # 只查找目录
fd -t f -s "README"          # 大小写敏感搜索
fd --changed-within 1d       # 1 天内修改过的文件
```

### 9.8 delta — Git Diff 美化器

已在 `.gitconfig` 中配置为默认 pager, 所有 `git diff/log/show` 自动使用:

- **Side-by-side**: 双栏对比视图
- **行号**: 显示行号
- **语法高亮**: Catppuccin Mocha 主题
- **导航**: `n/N` 在 diff hunk 之间跳转 (navigate = true)

```bash
git diff                     # 自动使用 delta 渲染
git log -p                   # commit 详情也使用 delta
git show HEAD                # 同上
delta file1 file2            # 直接比较两个文件
```

### 9.9 jq — JSON 处理器

```bash
cat data.json | jq '.'              # 格式化 JSON
cat data.json | jq '.name'          # 提取字段
cat data.json | jq '.items[]'       # 遍历数组
cat data.json | jq '.items | length' # 数组长度
curl -s api_url | jq '.data'        # 处理 API 响应
```

---

## 10. Git 工具链

### 10.1 核心配置

| 配置 | 值 | 作用 |
|------|-----|------|
| `core.quotepath = false` | 中文文件名正常显示 | 不再显示为八进制转义 |
| `core.autocrlf = input` | 提交时 CRLF→LF | 跨平台换行符统一 |
| `core.untrackedcache = true` | 缓存未跟踪文件 | 加速 `git status` |
| `core.pager = delta` | 使用 delta 渲染 | diff 语法高亮 + 双栏 |
| `http.postBuffer = 625MB` | 大推送缓冲区 | 支持推送大仓库 |
| `pull.rebase = false` | pull 时 merge | 不自动 rebase |
| `init.defaultBranch = master` | 默认分支名 | 新仓库默认 master |

### 10.2 Git LFS

已配置高性能 LFS:
- 并发传输: 32 路
- 超时: 3 秒 (快速失败)
- 重试: 最多 1 次, 延迟 2 秒

### 10.3 GitHub CLI (gh)

```bash
gh pr list                   # 列出 PR
gh pr create                 # 创建 PR
gh pr view 123               # 查看 PR 详情
gh issue list                # 列出 Issue
gh repo clone owner/repo     # 克隆仓库
gh auth status               # 查看登录状态
```

### 10.4 OMZ Git 别名 (高频)

| 别名 | 等价命令 | 功能 |
|------|---------|------|
| `gst` | `git status` | 查看状态 |
| `ga` | `git add` | 添加文件 |
| `gaa` | `git add --all` | 添加所有 |
| `gc` | `git commit -v` | 提交 |
| `gcmsg "msg"` | `git commit -m "msg"` | 提交 (带消息) |
| `gc!` | `git commit -v --amend` | 修改上次提交 |
| `gco` | `git checkout` | 切换分支 |
| `gcb` | `git checkout -b` | 创建并切换分支 |
| `gp` | `git push` | 推送 |
| `gl` | `git pull` | 拉取 |
| `gf` | `git fetch` | 获取远程 |
| `gb` | `git branch` | 查看分支 |
| `gd` | `git diff` | 查看差异 |
| `gds` | `git diff --staged` | 查看暂存区差异 |
| `glg` | `git log --stat` | 查看日志 (带文件统计) |
| `glog` | `git log --oneline --graph` | 查看图形化日志 |
| `gm` | `git merge` | 合并 |
| `grb` | `git rebase` | 变基 |
| `gsta` | `git stash push` | 暂存当前修改 |
| `gstp` | `git stash pop` | 弹出暂存 |
| `gstl` | `git stash list` | 列出暂存 |

---

## 11. 自定义别名与函数速查

### 服务器连接

| 命令 | 功能 |
|------|------|
| `gojumpserv` | SSH 跳板机连接脚本 |
| `goenvserv` | SSH 服务器管理脚本 |

### Java 开发

| 命令 | 功能 |
|------|------|
| `usejdk8` | 切换到 JDK 8 |
| `usejdk17` | 切换到 JDK 17 |
| `arthas` | 启动 Arthas Java 诊断工具 |

### tmux 会话管理

| 命令 | 功能 |
|------|------|
| `t` | 创建/附加 "main" 会话 |
| `t <name>` | 创建/附加指定名称的会话 |
| `tls` | 列出所有 tmux 会话 |
| `tk <name>` | 杀死指定会话 |
| `tka` | 杀死所有 tmux 会话 |

### 文件查看与浏览

| 命令 | 实际执行 | 功能 |
|------|---------|------|
| `cat` | `bat --paging=never` | 语法高亮查看文件 |
| `catp` | `bat` | 语法高亮 + 分页查看 |
| `ls` | `eza --icons` | 带图标列出文件 |
| `ll` | `eza -la --icons --git` | 详细列表 + git 状态 |
| `la` | `eza -a --icons` | 显示隐藏文件 |
| `tree` | `eza --tree --icons` | 树状视图 |
| `y` | Yazi 包装函数 | 文件管理器 (退出后 cd) |

### 其他

| 命令 | 功能 |
|------|------|
| `conf2db` | Excel 导表工具 (Python) |
| `z <keyword>` | 智能目录跳转 (zoxide) |
| `zi` | 交互式目录选择 (zoxide + fzf) |

---

## 12. 配置文件索引

| 文件路径 | 用途 | 修改后生效方式 |
|----------|------|---------------|
| `~/.zshrc` | Zsh 主配置 (插件、别名、函数) | `source ~/.zshrc` 或新开终端 |
| `~/.tmux.conf` | tmux 配置 (快捷键、外观) | `prefix + r` |
| `~/.config/starship.toml` | Starship 提示符配置 | 立即生效 (下次渲染) |
| `~/.config/nvim/` | Neovim (LazyVim) 配置 | 重启 nvim |
| `~/.config/yazi/` | Yazi 配置 (目前默认) | 重启 yazi |
| `~/.gitconfig` | Git 全局配置 | 立即生效 |
| `~/.zprofile` | Zsh login shell 配置 (Homebrew, OrbStack) | 重新登录 |
| `~/Library/Preferences/com.googlecode.iterm2.plist` | iTerm2 偏好设置 | 重启 iTerm2 |
| `~/.iterm2_shell_integration.zsh` | iTerm2 Shell Integration | 自动加载 |

---

## 13. 常见问题排查

### Q: Shell 启动变慢

```bash
# 测量启动时间
time zsh -i -c exit

# 查看 Zinit 各插件加载耗时
zinit times

# 如果某个插件耗时异常, 可以禁用排查
# 在 .zshrc 中注释对应的 zinit 行
```

### Q: Nerd Font 图标显示为方块/问号

1. 确认 iTerm2 → Profiles → Text → Font 设置为 **JetBrainsMono Nerd Font**
2. 如果仍有个别图标缺失, 确认 Non-ASCII Font 也设置为同一字体

### Q: tmux 中颜色不正确

确认 `~/.tmux.conf` 中有:
```
set -g default-terminal "tmux-256color"
set -as terminal-overrides ",xterm*:Tc"
```
并且 iTerm2 的 Terminal Type 为 `xterm-256color`。

### Q: git diff 没有使用 delta

```bash
# 确认 delta 已安装
which delta

# 确认 gitconfig 配置
git config --global core.pager

# 临时禁用 delta (排查问题)
git --no-pager diff
```

### Q: zoxide 跳不到想要的目录

```bash
# 查看 zoxide 数据库
zoxide query -ls

# 手动添加目录
zoxide add ~/projects/important

# 删除错误条目
zoxide remove ~/old/deleted/path
```

### Q: Neovim LSP 不工作

```bash
# 在 Neovim 中执行
:Mason              # 安装对应语言的 LSP server
:LspInfo            # 查看当前 LSP 状态
:checkhealth        # 全面健康检查
```

### Q: 如何恢复原始 cat/ls 命令

```bash
command cat file     # 绕过别名, 使用原始 cat
command ls           # 绕过别名, 使用原始 ls
\cat file            # 同上 (反斜杠前缀)
```
