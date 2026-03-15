# Agent Skills 管理仓库

本仓库用于集中管理你的 OpenCode Agent Skills，方便你在不同设备间同步和更新。

## 这是什么？

这是一个技能仓库管理器：
- **存放位置**：`~/.agent-skills/` - 存放所有 skill repos（包括子模块和本地 skills）
- **管理命令**：安装几个快捷命令到 OpenCode，方便你从 `~/.agent-skills/` 安装 skills 到项目中

## 快速设置

### 第一步：克隆仓库到 ~/.agent-skills/

```bash
# 方式1：使用 HTTPS
git clone --recurse-submodules https://github.com/mac-home-config-ysj/.agent-skills.git ~/.agent-skills

# 方式2：使用 SSH（推荐，已配置 SSH key 的用户）
git clone --recurse-submodules git@github.com:mac-home-config-ysj/.agent-skills.git ~/.agent-skills
```

### 第二步：安装管理命令

将这些命令通过软链接安装到 OpenCode 全局配置，方便在任何项目使用：

```bash
# 创建命令目录（如果不存在）
mkdir -p ~/.config/opencode/commands

# 创建软链接（排除 git-update.md）
ln -s ~/.agent-skills/.opencode/commands/install-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/.opencode/commands/uninstall-skills.md ~/.config/opencode/commands/
ln -s ~/.agent-skills/.opencode/commands/update-skills.md ~/.config/opencode/commands/
```

### 第三步：验证

启动 OpenCode，输入 `/` 查看可用命令，你应该看到：
- `/install-skills` - 从 ~/.agent-skills/ 安装 skills 到当前项目
- `/uninstall-skills` - 从当前项目卸载 skills
- `/update-skills` - 更新 ~/.agent-skills/ 中的所有 skills

## 日常使用

### 添加新的 skill repo

1. 进入 ~/.agent-skills/ 目录：
```bash
cd ~/.agent-skills
```

2. 添加为子模块（如果是外部仓库）：
```bash
git submodule add https://github.com/user/repo.git my-skill
```

3. 或者创建本地 skill 目录：
```bash
mkdir my-local-skill
cd my-local-skill
# 创建 skill 文件...
```

4. 提交到本仓库：
```bash
git add .
git commit -m "添加新的 skill: my-skill"
git push
```

### 在项目中使用 skills

在任何项目目录下启动 OpenCode，使用以下全局命令：

```bash
# 安装 skill 到当前项目
/install-skills

# 卸载 skill
/uninstall-skills
```

### 更新 ~/.agent-skills/ 中的 skills

定期更新你的中央 skills 仓库：

```bash
# 从任何项目运行，更新 ~/.agent-skills/ 中的所有子模块
/update-skills
```

### 手动更新（可选）

如果你不想使用 `/update-skills` 命令，也可以手动更新：

```bash
cd ~/.agent-skills

# 更新所有子模块到最新版本
git submodule update --remote

# 提交子模块变更
git add .
git commit -m "更新 skills"
git push
```

## 目录结构

```
~/.agent-skills/                    # 本仓库位置
├── .opencode/commands/             # 管理命令
│   ├── install-skills.md           # → 复制到 ~/.config/opencode/commands/
│   ├── uninstall-skills.md         # → 复制到 ~/.config/opencode/commands/
│   ├── update-skills.md            # → 复制到 ~/.config/opencode/commands/
│   └── git-update.md               # 保留在此，不复制
│
├── .gitmodules                     # Git 子模块配置
│
├── anthropics-skills/              # 子模块：Anthropic 官方 skills
├── superpowers/                    # 子模块：Superpowers skills
│
├── github/                         # 本地 skill
├── nano-pdf/                       # 本地 skill
├── skill-vetter/                   # 本地 skill
│
└── README.md                       # 本文件
```

## 工作原理

1. **~/.agent-skills/**：作为中央仓库，存放所有 skill repos
2. **~/.config/opencode/commands/**：存放全局可用的管理命令
3. **软链接**：skills/ 目录通过软链接引用子模块中的 skills，方便统一管理

当你运行 `/install-skills` 时，命令会从 `~/.agent-skills/` 复制或链接 skills 到你的项目目录。

## 常见问题

**Q: 为什么要把仓库放在 ~/.agent-skills/？**  
A: 这是约定的位置，方便管理命令找到你的 skills。

**Q: 子模块和本地 skills 有什么区别？**  
A: 子模块链接到外部仓库，可以跟踪更新；本地 skills 是你自己创建的，只存在于本仓库。

**Q: 如何添加自己的 skill？**  
A: 在 ~/.agent-skills/ 下创建目录，添加 skill 文件，然后提交到 git。

**Q: git-update.md 为什么不复制？**  
A: 它是项目特定的命令，只在 ~/.agent-skills/ 仓库内使用，不需要全局安装。
