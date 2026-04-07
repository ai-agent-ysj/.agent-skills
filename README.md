# Agent Skills 管理仓库

本仓库用于集中管理你的 AI Agent Skills，支持 **OpenCode** 和 **Claude Code** 双平台，方便你在不同设备间同步和更新。

> **平台说明**：目前仅支持 macOS。

## 这是什么？

这是一个技能仓库管理器：

- **存放位置**：`~/.agent-skills/` - 集中存放所有 skills（SkillHub 下载、本地自建、Git 仓库）
- **管理命令**：安装几个快捷命令，方便你通过 `/install-skills`、`/uninstall-skills` 等命令管理 skills
- **双平台**：同一份源文件，可分别安装到 OpenCode 和 Claude Code

## 前置要求

在开始之前，请确保已安装以下工具：

| 工具                                           | 说明                           | 安装方式                      |
| -------------------------------------------- | ---------------------------- | ------------------------- |
| [OpenCode](https://opencode.ai) 或 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AI 编程助手（至少安装其中一个） | 参见各自官网文档 |
| [git](https://git-scm.com)                   | 版本控制，用于克隆仓库和管理子模块            | `brew install git`（macOS） |
| [skillhub CLI](https://skillhub.tencent.com) | SkillHub 官方命令行工具，用于搜索和下载在线技能 | `curl -fsSL https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/install/install.sh \| bash -s -- --cli-only` |

> **注意**：skillhub CLI 仅在使用 SkillHub 在线来源时才需要。如果你只使用本地或 Git 仓库来源，可以跳过安装。

## 快速设置

### 第一步：克隆仓库到 \~/.agent-skills/

方式1：使用 HTTPS

```bash
git clone --recurse-submodules https://github.com/ysjai/.agent-skills.git ~/.agent-skills
```

方式2：使用 SSH（推荐，已配置 SSH key 的用户）

```bash
git clone --recurse-submodules git@github.com:ysjai/.agent-skills.git ~/.agent-skills
```

### 第二步：安装管理命令

根据你使用的平台，将管理命令通过软链接安装到对应的全局配置目录：

#### OpenCode

```bash
# 创建命令目录（如果不存在）
mkdir -p ~/.config/opencode/commands

# 创建软链接
ln -s ~/.agent-skills/commands/download-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/install-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/uninstall-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/update-skills.md ~/.config/opencode/commands/
```

#### Claude Code

```bash
# 创建命令目录（如果不存在）
mkdir -p ~/.claude/commands

# 创建软链接（在 Claude Code 中以 /user:xxx 调用）
ln -s ~/.agent-skills/commands/download-skills.md ~/.claude/commands/
ln -s ~/.agent-skills/commands/install-skills.md ~/.claude/commands/
ln -s ~/.agent-skills/commands/uninstall-skills.md ~/.claude/commands/
ln -s ~/.agent-skills/commands/update-skills.md ~/.claude/commands/
```

> **两个平台都安装？** 完全可以！同一份命令文件通过软链接同时服务两个平台。

### 第三步：验证

#### OpenCode

启动 OpenCode，输入 `/` 查看可用命令，你应该看到：

- `/download-skills` - 只下载 skills 到 `~/.agent-skills/`，**不创建软链接**
- `/install-skills` - 安装 skills 并创建软链接（先选平台和安装目标，再选来源）
- `/uninstall-skills` - 卸载已安装的 skills（移除软链接）
- `/update-skills` - 更新已有 skills

#### Claude Code

启动 Claude Code，输入 `/` 查看可用命令，你应该看到：

- `/user:download-skills` - 只下载 skills 到 `~/.agent-skills/`
- `/user:install-skills` - 安装 skills（先选平台和安装目标，再选来源）
- `/user:uninstall-skills` - 卸载已安装的 skills
- `/user:update-skills` - 更新已有 skills

## 平台路径对照表

| 用途 | OpenCode | Claude Code |
|------|----------|-------------|
| Skills 全局目录 | `~/.config/opencode/skills/` | `~/.claude/skills/` |
| Skills 项目级目录 | `<项目>/.opencode/skills/` | `<项目>/.claude/skills/` |
| Commands 全局目录 | `~/.config/opencode/commands/` | `~/.claude/commands/` |
| Commands 项目级目录 | `<项目>/.opencode/commands/` | `<项目>/.claude/commands/` |
| 命令调用方式 | `/install-skills` | `/user:install-skills` |
| 整组安装 | ✅ 支持 | ❌ 不支持（需逐个安装） |

## 日常使用

### 安装 skills

**只下载不安装**（仅下载到 `~/.agent-skills/`，不创建软链接）：

```
/download-skills          # OpenCode
/user:download-skills     # Claude Code
```

**安装 skills**（选择平台、项目级或全局，按提示选择）：

```
/install-skills           # OpenCode
/user:install-skills      # Claude Code
```

两个命令说明：

- `download-skills`：支持 SkillHub 在线下载和 Git 仓库 clone，只下载源文件不安装
- `install-skills`：先询问目标平台和安装目标，再支持三种来源（SkillHub、git-repo-skills、personal-skills），按来源决定是否先下载再创建软链接

### 添加 Git 仓库 skill

通过 `install-skills` → 选择 git-repo-skills 来源，按提示输入 clone 地址即可。

也可以手动添加子模块：

```bash
cd ~/.agent-skills
git submodule add https://github.com/user/repo.git git-repo-skills/my-repo
git submodule update --init git-repo-skills/my-repo
```

### 添加本地自建 skill

在 `personal-skills/` 下创建目录，放入 skill 文件：

```bash
mkdir ~/.agent-skills/personal-skills/my-skill
# 创建 SKILL.md 等文件...
```

然后通过 `install-skills` → 选择 personal-skills 来源安装。

### 卸载 skills

```
/uninstall-skills         # OpenCode
/user:uninstall-skills    # Claude Code
```

选择平台和安装范围后，会列出已安装的 skills（包括软链接和非托管的真实目录/文件）。选择要卸载的 skill，确认后移除软链接。非托管项会标记 ⚠️，需额外确认。

### 更新已有 skills

```
/update-skills            # OpenCode
/user:update-skills       # Claude Code
```

或手动更新：

```bash
cd ~/.agent-skills
git submodule update --remote git-repo-skills/
git add .
git commit -m "Update git-repo-skills submodules"
git push
```

### 同步仓库到最新（换设备 / 拉取他人更新）

一键拉取主仓库最新内容，并同步所有子模块：

```bash
git -C ~/.agent-skills pull && git -C ~/.agent-skills submodule update --init --recursive --remote
```

## 目录结构

```
~/.agent-skills/                    # 本仓库位置（中央仓库，平台无关）
├── commands/                       # 管理命令（可软链接到 OpenCode 和/或 Claude Code）
│   ├── download-skills.md
│   ├── install-skills.md
│   ├── uninstall-skills.md
│   └── update-skills.md
│
├── skill-hub/                      # SkillHub 下载的 skills（由 /install-skills 自动管理）
│   └── <slug>/
│
├── personal-skills/                # 个人自建 skills
│   ├── brainstorming/
│   ├── writing-plans/
│   └── ...
│
├── git-repo-skills/                # Git 仓库 skills（子模块或手动 clone）
│   ├── anthropics-skills/          # 子模块：Anthropic 官方 skills
│   └── superpowers/                # 子模块：Superpowers skills（或手动 clone 的仓库）
│
├── .gitmodules                     # Git 子模块配置
└── README.md                       # 本文件
```

软链接安装在各平台对应的目录下：

```
# OpenCode（当前项目）
<你的项目>/
└── .opencode/
    └── skills/
        ├── brave-search -> ~/.agent-skills/skill-hub/brave-search/
        └── frontend-design -> ~/.agent-skills/git-repo-skills/anthropics-skills/skills/frontend-design/

# Claude Code（当前项目）
<你的项目>/
└── .claude/
    └── skills/
        ├── brave-search -> ~/.agent-skills/skill-hub/brave-search/
        └── frontend-design -> ~/.agent-skills/git-repo-skills/anthropics-skills/skills/frontend-design/
```

## 工作原理

1. **`~/.agent-skills/`**：作为中央仓库，集中存放三类 skills 的源文件（平台无关）
2. **安装目标目录**：通过软链接将源文件挂载到对应平台的 skills 目录
3. **命令目录**：同一份命令文件可同时软链接到 OpenCode 和 Claude Code

当你运行 `download-skills` 时：

- SkillHub 来源：下载源文件到 `~/.agent-skills/skill-hub/<slug>/`，不创建软链接
- Git 仓库来源：clone 到 `~/.agent-skills/git-repo-skills/<repo>/`，不创建软链接

当你运行 `install-skills` 时：

- 先选择目标平台（OpenCode / Claude Code）和安装范围（项目级 / 全局）
- 再选择来源：SkillHub 先下载再链接，personal-skills / git-repo-skills 直接链接
- Claude Code 不支持整组安装（需逐个 skill 安装）

## 常见问题

**Q: 为什么要把仓库放在 \~/.agent-skills/？**
A: 这是约定的位置，方便管理命令用绝对路径找到所有 skills。

**Q: SkillHub、git-repo-skills、personal-skills 有什么区别？**
A: SkillHub 是从 [skillhub.tencent.com](https://skillhub.tencent.com) 在线下载的第三方 skills；git-repo-skills 是通过 git submodule 跟踪的外部仓库，可以统一执行 `git submodule update --remote` 更新；personal-skills 是你自己创建和维护的技能。

**Q: 同一个仓库可以同时服务 OpenCode 和 Claude Code 吗？**
A: 可以。中央仓库 `~/.agent-skills/` 存放源文件，通过软链接分别安装到两个平台的 skills 目录。同一个 skill 可以同时安装到两个平台。

**Q: Claude Code 为什么不支持整组安装？**
A: Claude Code 要求 `skills/<name>/SKILL.md` 的扁平目录结构，不支持把一个包含多个 skill 的父目录作为单个入口安装。OpenCode 则支持嵌套结构的整组安装。

**Q: 在 Claude Code 中命令为什么是 /user:xxx 格式？**
A: Claude Code 区分用户级命令（`/user:xxx`，来自 `~/.claude/commands/`）和项目级命令（`/project:xxx`，来自 `.claude/commands/`）。本仓库的管理命令安装到用户级目录，因此以 `/user:` 前缀调用。
