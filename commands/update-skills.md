---
description: 更新本地已有的 skills（支持 SkillHub skill-hub 和 git-repo-skills 子模块）
---

# 更新 Skills

更新 `~/.agent-skills/` 中已下载的 skills 到最新版本。支持两种来源：

| 来源 | 存放位置 | 更新方式 |
|------|---------|---------|
| SkillHub | `skill-hub/<slug>/` | `skillhub install <slug> --force` |
| git-repo-skills | `git-repo-skills/<repo>/` | `git pull` 或 `git submodule update --remote` |

---

## 执行步骤

### 1. 选择更新来源

使用 question 工具询问用户：

```
请选择要更新的来源：
```
- 选项1（默认）：SkillHub（更新 skill-hub/ 中已下载的 skills）
- 选项2：git-repo-skills（更新 git 仓库子模块）
- 选项3：全部更新（以上两种都更新）

---

## 分支 A：更新 SkillHub skills

### A1. 扫描已有的 skill-hub skills

```bash
SKILL_HUB_DIR="$HOME/.agent-skills/skill-hub"
if [ -d "$SKILL_HUB_DIR" ] && [ -n "$(ls -A "$SKILL_HUB_DIR" 2>/dev/null)" ]; then
    for d in "$SKILL_HUB_DIR"/*/; do
        [ -d "$d" ] || continue
        slug=$(basename "$d")
        skill_md="$d/SKILL.md"
        name=$(grep -m1 '^# ' "$skill_md" 2>/dev/null | sed 's/^# //' || echo "$slug")
        echo "$slug|$name"
    done
else
    echo "EMPTY"
fi
```

如果 `skill-hub/` 为空（EMPTY），告知用户本地没有已下载的 SkillHub skills，可先运行 `/install-skills` 下载，终止本分支。

### A2. 选择要更新的 skills

用 question 工具展示列表，让用户选择（输入编号，多个空格分隔，all=全选，exit=取消）：

```
skill-hub/ 中已有以下 skills，请选择要更新的：

  [1] brave-search   Brave 网页搜索
  [2] github         GitHub 操作
  [3] weather        天气查询
```

### A3. 检查 skillhub CLI

```bash
which skillhub 2>/dev/null && skillhub -v 2>&1 || echo "NOT_INSTALLED"
```

如果未安装，使用 question 工具询问：
- 选项1（默认）：立即安装 skillhub CLI
- 选项2：取消

选择"取消"则终止，提示用户手动安装：
```
curl -fsSL https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/install/install.sh | bash -s -- --cli-only
```
选择"安装"则执行上述命令，安装完成后重新验证，若仍失败则终止。

### A4. 执行更新下载

对每个选中的 slug：

```bash
TARGET_DIR="$HOME/.agent-skills/skill-hub"

TMP_DIR=$(mktemp -d)
cd "$TMP_DIR"
skillhub install "$SKILL_SLUG" --force 2>&1
cd - > /dev/null

INSTALLED_DIR="$TMP_DIR/skills/$SKILL_SLUG"
if [ -d "$INSTALLED_DIR" ]; then
    rm -rf "$TARGET_DIR/$SKILL_SLUG"
    mv "$INSTALLED_DIR" "$TARGET_DIR/$SKILL_SLUG"
    echo "UPDATED: $SKILL_SLUG"
else
    FOUND=$(ls "$TMP_DIR/skills/" 2>/dev/null | head -1)
    if [ -n "$FOUND" ]; then
        rm -rf "$TARGET_DIR/$SKILL_SLUG"
        mv "$TMP_DIR/skills/$FOUND" "$TARGET_DIR/$SKILL_SLUG"
        echo "UPDATED: $SKILL_SLUG"
    else
        echo "FAILED: $SKILL_SLUG"
    fi
fi
rm -rf "$TMP_DIR"
```

### A5. 汇报

列出成功更新、失败的 skills，告知用户源文件已在 `~/.agent-skills/skill-hub/`，已安装的软链接自动生效（无需重新安装），提醒**重启 OpenCode** 生效。

---

## 分支 B：更新 git-repo-skills

### B1. 检查仓库是否存在

```bash
ls -d "$HOME/.agent-skills/.git" 2>/dev/null || echo "NOT_GIT"
```

如果 `~/.agent-skills` 不是 git 仓库，告知用户并终止。

### B2. 检查工作区状态

```bash
git -C "$HOME/.agent-skills" status --short
```

如果有未提交的更改（staged 或 unstaged），立即停止并报告未提交的文件。未跟踪的文件可以忽略。

### B3. 扫描已有的 git-repo-skills

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
if [ -d "$GIT_REPO_DIR" ] && [ -n "$(ls -A "$GIT_REPO_DIR" 2>/dev/null)" ]; then
    ls -1 "$GIT_REPO_DIR"
else
    echo "EMPTY"
fi
```

如果 `git-repo-skills/` 为空（EMPTY），告知用户本地没有已克隆的 Git 仓库，可先运行 `/install-skills` 添加，终止本分支。

### B4. 检测未注册为子模块的仓库

扫描 `git-repo-skills/` 下包含 `.git` 的一级子目录，对比 `.gitmodules` 中已注册的子模块：

```bash
cd "$HOME/.agent-skills"
# 列出 git-repo-skills/ 下所有含 .git 的目录
for d in git-repo-skills/*/; do
    [ -d "$d/.git" ] || continue
    path="${d%/}"
    echo "HAS_GIT|$path"
done

# 查看已注册的子模块
git submodule status 2>/dev/null | awk '{print $2}'
```

对于未注册为子模块的目录，检查是否干净（无未推送提交、无未提交修改）：

```bash
git -C "$HOME/.agent-skills/$REPO_PATH" status --short
git -C "$HOME/.agent-skills/$REPO_PATH" remote get-url origin 2>/dev/null || echo "NO_REMOTE"
```

若干净且有 remote，询问用户是否注册为子模块：
- 选项1（默认）：注册为子模块（先删除目录再 `git submodule add`）
- 选项2：跳过（保持手动 clone 状态）

```bash
REMOTE_URL=$(git -C "$HOME/.agent-skills/$REPO_PATH" remote get-url origin)
rm -rf "$HOME/.agent-skills/$REPO_PATH"
git -C "$HOME/.agent-skills" submodule add "$REMOTE_URL" "$REPO_PATH"
```

### B5. 更新所有子模块

```bash
git -C "$HOME/.agent-skills" submodule update --init --recursive
git -C "$HOME/.agent-skills" submodule update --remote --recursive
```

### B6. 检查变更并提交

```bash
git -C "$HOME/.agent-skills" status --short
```

如果没有变更，报告"所有子模块已是最新版本"，结束。

如果有变更：

```bash
git -C "$HOME/.agent-skills" add .gitmodules
git -C "$HOME/.agent-skills" add git-repo-skills/
git -C "$HOME/.agent-skills" commit -m "更新 git-repo-skills 子模块到最新版本"
```

### B7. 汇报

列出各仓库的更新情况（旧 commit → 新 commit，如有变化），提醒**重启 OpenCode** 生效。不执行 `git push`，除非用户明确要求。

---

## 选项3：全部更新

依次执行分支 A（SkillHub）和分支 B（git-repo-skills），汇总两者的结果一并汇报。
