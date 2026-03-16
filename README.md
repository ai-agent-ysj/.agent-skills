# Agent Skills 管理仓库

本仓库用于集中管理你的 OpenCode Agent Skills，方便你在不同设备间同步和更新。

> **平台说明**：目前仅支持 macOS。

## 这是什么？

这是一个技能仓库管理器：
- **存放位置**：`~/.agent-skills/` - 集中存放所有 skills（SkillHub 下载、本地自建、Git 仓库）
- **管理命令**：安装几个快捷命令到 OpenCode，方便你通过 `/download-skills`、`/install-skills` 管理 skills

## 前置要求

在开始之前，请确保已安装以下工具：

| 工具 | 说明 | 安装方式 |
|------|------|----------|
| [OpenCode](https://opencode.ai) | AI 编程助手（本仓库的宿主） | 参见官网文档 |
| [git](https://git-scm.com) | 版本控制，用于克隆仓库和管理子模块 | `brew install git`（macOS） |
| [skillhub CLI](https://skillhub.tencent.com) | SkillHub 官方命令行工具，用于搜索和下载在线技能 | `npm install -g skillhub` |

> **注意**：skillhub CLI 仅在使用 SkillHub 在线来源时才需要。如果你只使用本地或 Git 仓库来源，可以跳过安装。

## 快速设置

### 第一步：克隆仓库到 ~/.agent-skills/

方式1：使用 HTTPS

```bash
git clone --recurse-submodules https://github.com/mac-home-config-ysj/.agent-skills.git ~/.agent-skills
```

方式2：使用 SSH（推荐，已配置 SSH key 的用户）

```bash
git clone --recurse-submodules git@github.com:mac-home-config-ysj/.agent-skills.git ~/.agent-skills
```

### 第二步：安装管理命令

将这些命令通过软链接安装到 OpenCode 全局配置，方便在任何项目使用：

```bash
# 创建命令目录（如果不存在）
mkdir -p ~/.config/opencode/commands

# 创建软链接
ln -s ~/.agent-skills/commands/download-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/install-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/uninstall-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/commands/update-skills.md ~/.config/opencode/commands/
```

### 第三步：验证

启动 OpenCode，输入 `/` 查看可用命令，你应该看到：
- `/download-skills` - 只下载 skills 到 `~/.agent-skills/`，**不创建软链接**（支持 SkillHub 在线下载和 Git clone）
- `/install-skills` - 安装 skills 并创建软链接（先选安装目标：当前项目 or 全局；再选来源：SkillHub、personal-skills、git-repo-skills）
- `/uninstall-skills` - 卸载已安装的 skills（移除软链接）
- `/update-skills` - 更新已有 skills（支持更新 SkillHub 来源和 git-repo-skills 子模块）

## 日常使用

### 安装 skills

**只下载不安装**（仅下载到 `~/.agent-skills/`，不创建软链接）：

```
/download-skills
```

**安装 skills**（项目级或全局，按提示选择）：

```
/install-skills
```

两个命令说明：
- `/download-skills`：支持 SkillHub 在线下载和 Git 仓库 clone，只下载源文件不安装
- `/install-skills`：先询问安装目标（当前项目 or 全局），再支持三种来源（SkillHub、git-repo-skills、personal-skills），安装时会先执行下载流程再创建软链接

### 添加 Git 仓库 skill

通过 `/install-skills` → 选择 git-repo-skills 来源，按提示输入 clone 地址即可。

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

然后通过 `/install-skills` → 选择 personal-skills 来源安装。

### 更新已有 skills

```
/update-skills
```

或手动更新：

```bash
cd ~/.agent-skills
git submodule update --remote git-repo-skills/
git add .
git commit -m "更新 git-repo-skills 子模块"
git push
```

## 目录结构

```
~/.agent-skills/                    # 本仓库位置
├── commands/                       # 管理命令（软链接到 ~/.config/opencode/commands/）
│   ├── download-skills.md
│   ├── install-skills.md
│   ├── uninstall-skills.md
│   └── update-skills.md
│
├── skill-hub/                      # SkillHub 下载的 skills（由 /install-skills 自动管理）
│   └── <slug>/
│
├── personal-skills/                # 个人自建 skills
│   ├── github/
│   ├── nano-pdf/
│   └── skill-vetter/
│
├── git-repo-skills/                # Git 仓库 skills（子模块）
│   ├── anthropics-skills/          # 子模块：Anthropic 官方 skills
│   └── superpowers/                # 子模块：Superpowers skills
│
├── .gitmodules                     # Git 子模块配置
└── README.md                       # 本文件
```

软链接安装在**当前项目**目录下：

```
<你的项目>/
└── .opencode/
    └── skills/                     # 软链接由 /install-skills 自动创建，指向 ~/.agent-skills/ 中的源文件
        ├── brave-search -> ~/.agent-skills/skill-hub/brave-search/
        ├── github -> ~/.agent-skills/personal-skills/github/
        └── frontend-design -> ~/.agent-skills/git-repo-skills/anthropics-skills/skills/frontend-design/
```

## 工作原理

1. **`~/.agent-skills/`**：作为中央仓库，集中存放三类 skills 的源文件
2. **`<当前项目>/.opencode/skills/`**：软链接安装目标，指向 `~/.agent-skills/` 中的源文件
3. **`~/.config/opencode/commands/`**：存放全局可用的管理命令软链接

当你运行 `/download-skills` 时：
- SkillHub 来源：下载源文件到 `~/.agent-skills/skill-hub/<slug>/`，不创建软链接
- Git 仓库来源：clone 到 `~/.agent-skills/git-repo-skills/<repo>/`，不创建软链接

当你运行 `/install-skills` 时：
- SkillHub 来源：先执行下载（同 `/download-skills`），再在目标目录创建软链接
- personal-skills / git-repo-skills 来源：（如需 clone 则先执行），再创建指向源目录的软链接

## 常见问题

**Q: 为什么要把仓库放在 ~/.agent-skills/？**
A: 这是约定的位置，方便管理命令用绝对路径找到所有 skills。

**Q: SkillHub、git-repo-skills、personal-skills 有什么区别？**
A: SkillHub 是从 [skillhub.tencent.com](https://skillhub.tencent.com) 在线下载的第三方 skills；git-repo-skills 是通过 git submodule 跟踪的外部仓库，可以统一执行 `git submodule update --remote` 更新；personal-skills 是你自己创建和维护的技能。

