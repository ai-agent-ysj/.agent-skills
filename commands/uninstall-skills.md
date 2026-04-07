---
description: 卸载 skills（支持 OpenCode 和 Claude Code 双平台，从当前项目或全局移除已安装的 skills）
---

# 卸载 Skills

交互式移除已安装的 skills 软链接，支持当前项目和全局两种目标，适用于 OpenCode 和 Claude Code 双平台。

## 执行步骤

### 0. 选择目标平台

使用 question 工具询问用户：

```
请选择目标平台：
```
- 选项1（默认）：OpenCode
- 选项2：Claude Code

设置 `PLATFORM` 变量。

### 1. 选择卸载目标

使用 question 工具询问用户：

```
请选择卸载目标：
```

**OpenCode**：
- 选项1（默认）：当前项目（`$(pwd)/.opencode/skills/`）
- 选项2：全局（`~/.config/opencode/skills/`）

**Claude Code**：
- 选项1（默认）：当前项目（`$(pwd)/.claude/skills/`）
- 选项2：全局（`~/.claude/skills/`）

路径映射表：

| 平台 | 范围 | SKILLS_DIR |
|------|------|------------|
| OpenCode | 当前项目 | `$(pwd)/.opencode/skills` |
| OpenCode | 全局 | `$HOME/.config/opencode/skills` |
| Claude Code | 当前项目 | `$(pwd)/.claude/skills` |
| Claude Code | 全局 | `$HOME/.claude/skills` |

### 2. 扫描已安装的 skills

```bash
if [ -d "$SKILLS_DIR" ]; then
    for item in "$SKILLS_DIR"/*; do
        [ -e "$item" ] || [ -L "$item" ] || continue
        name=$(basename "$item")
        if [ -L "$item" ]; then
            target=$(readlink "$item")
            # 检查是否由本仓库管理
            case "$target" in
                $HOME/.agent-skills/*) managed="yes" ;;
                *) managed="no" ;;
            esac
            # 检查是否是分组（软链接目标目录内含多个 SKILL.md）— 仅 OpenCode 下有此情况
            if [ "$managed" = "yes" ] && [ -d "$item" ] && ls "$item"/*/SKILL.md >/dev/null 2>&1; then
                count=$(ls -d "$item"/*/SKILL.md 2>/dev/null | wc -l)
                echo "group|$name|$target|$count|$managed"
            else
                echo "single|$name|$target|$managed"
            fi
        else
            # 非软链接的真实目录/文件
            echo "real|$name"
        fi
    done
else
    echo "NO_SKILLS_DIR"
fi
```

如果目标目录不存在或没有任何条目，告知用户所选目标没有已安装的 skills，终止流程。

### 3. 展示并让用户选择

根据扫描结果构建带编号的选项列表。

**展示方式因平台而异：**

#### OpenCode 平台

支持分组整组卸载，使用 question 工具展示：

```
请选择要卸载的内容（输入编号，多个用空格分隔，all=全选，exit=取消）：

分组（整组卸载）：
[1] anthropics-skills（17 个 skills）
[2] superpowers（12 个 skills）

独立 skills：
[3] my-custom-skill
[4] another-skill

非本仓库管理：
[5] some-external ⚠️ 非软链接（真实目录）
[6] other-tool ⚠️ 指向其他位置
```

- 输入组号 = 整组卸载（移除分组软链接）
- 输入独立编号 = 卸载该 skill 的软链接
- 支持混合选择，如 `1 3`
- 输入 `all` 全选（不含非本仓库管理的条目）
- 输入 `exit` 取消

#### Claude Code 平台

所有 skills 均为平铺结构（无分组），使用 question 工具展示：

```
请选择要卸载的 skills（输入编号，多个用空格分隔，all=全选，exit=取消）：

由本仓库管理：
  [1] frontend-design
  [2] github
  [3] brainstorming

非本仓库管理：
  [4] learned ⚠️ 非软链接（真实目录）
  [5] some-external ⚠️ 指向其他位置
```

- `⚠️` 标记非本仓库来源的条目
- `all` 仅选中"由本仓库管理"的条目
- 如果用户显式选择了非本仓库管理的项，需二次确认

### 4. 移除

```bash
# 软链接（本仓库管理或用户确认覆盖的外部软链接）：
rm "$SKILLS_DIR/<name>"

# 非软链接的真实目录/文件（用户二次确认后）：
rm -rf "$SKILLS_DIR/<name>"
```

只移除目标目录中的链接/条目，**不删除** `~/.agent-skills/` 中的源文件。

### 5. 汇报并提醒

列出本次卸载的所有 skills（区分已卸载/跳过），提醒用户：**请重启当前平台（OpenCode / Claude Code）以使变更生效。**
