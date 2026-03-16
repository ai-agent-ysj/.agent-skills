---
description: 卸载 skills（支持从当前项目或全局移除已安装的 skills）
---

# 卸载 Skills

交互式移除已安装的 skills 软链接，支持当前项目和全局两种目标。

## 执行步骤

### 0. 选择卸载目标

使用 question 工具询问用户：

```
请选择卸载目标：
```
- 选项1（默认）：当前项目（`$(pwd)/.opencode/skills/`）
- 选项2：全局（`~/.config/opencode/skills/`）

根据选择设置变量：
- 选项1：`SKILLS_DIR="$(pwd)/.opencode/skills"`，后续提示用"当前项目"
- 选项2：`SKILLS_DIR="$HOME/.config/opencode/skills"`，后续提示用"全局"

### 1. 扫描已安装的 skills

```bash
if [ -d "$SKILLS_DIR" ]; then
    for item in "$SKILLS_DIR"/*; do
        if [ -L "$item" ]; then
            name=$(basename "$item")
            target=$(readlink "$item")
            # 检查是否是分组（软链接目标目录内含多个 skill）
            if [ -d "$item" ] && ls "$item"/*/SKILL.md >/dev/null 2>&1; then
                count=$(ls -d "$item"/*/SKILL.md 2>/dev/null | wc -l)
                echo "group|$name|$target|$count"
            else
                echo "single|$name|$target"
            fi
        fi
    done
else
    echo "NO_SKILLS_DIR"
fi
```

如果目标目录不存在或没有任何软链接，告知用户所选目标没有已安装的 skills，终止流程。

### 2. 展示并让用户选择

根据扫描结果构建带编号的选项列表，使用 question 工具展示：

```
请选择要卸载的内容（输入编号，多个用空格分隔，all=全选，exit=取消）：

分组（输入组号=整组卸载，输入子编号=单独卸载）：
[1] anthropics-skills（17 个 skills）
    [1a] frontend-design
    [1b] github
    [1c] skill-creator
    ...
[2] superpowers（12 个 skills）
    [2a] weather
    [2b] browser
    ...

独立 skills：
[3] my-custom-skill
[4] another-skill
```

- 输入组号 = 整组卸载（移除分组软链接）
- 输入子编号 = 单独卸载（仅移除该 skill 的软链接，不影响同组其他 skills）
- 支持混合选择，如 `1 3`
- 输入 `all` 全选
- 输入 `exit` 取消

### 3. 移除软链接

```bash
# 整组卸载，移除分组软链接
rm "$SKILLS_DIR/<group>"

# 单个 skill 卸载，移除该 skill 的软链接
rm "$SKILLS_DIR/<skillname>"
```

只移除软链接，**不删除** `~/.agent-skills/` 中的源文件。

### 4. 汇报并提醒

列出本次卸载的所有 skills，提醒用户：**请重启 OpenCode 以使变更生效。**
