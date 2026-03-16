---
description: 安装 skills（支持项目级或全局，来源支持 SkillHub、Git repo、本地 personal-skills）
---

# 安装 Skills

你的任务是帮用户安装 skill，以软链接方式挂载到指定目录。支持三种来源：

| 来源 | 说明 | 文件位置 |
|------|------|---------|
| SkillHub | 从腾讯 SkillHub 在线下载 | 下载到 `skill-hub/<slug>/`，再软链接 |
| personal-skills | 用户手动放入 `personal-skills/` 的自建 skills | 直接从 `personal-skills/<name>/` 软链接 |
| git-repo-skills | `git-repo-skills/` 下的 Git 仓库（支持 clone 新增） | 直接从 `git-repo-skills/<repo>/` 软链接 |

**路径约定**（贯穿全流程）：
- `AGENT_SKILLS_DIR="$HOME/.agent-skills"` — skills 中央仓库，存放源文件
- `SKILLS_DIR` — 软链接安装目标，由第一步用户选择决定

---

## 执行步骤

### 0. 选择安装目标

使用 question 工具询问用户：

```
请选择安装目标：
```
- 选项1（默认）：当前项目（`$(pwd)/.opencode/skills/`，仅当前项目生效）
- 选项2：全局（`~/.config/opencode/skills/`，所有项目均生效）

根据选择设置变量：
- 选项1：`SKILLS_DIR="$(pwd)/.opencode/skills"`，后续汇报用"当前项目"
- 选项2：`SKILLS_DIR="$HOME/.config/opencode/skills"`，后续汇报用"全局"

---

### 1. 选择安装来源

使用 question 工具询问用户：

```
请选择安装来源：
```
- 选项1（默认）：SkillHub（在线下载，国内高速，收录 1.3 万个 AI Skills）
- 选项2：git-repo-skills（Git 仓库，支持输入 clone 地址新增）
- 选项3：personal-skills（本地自建，手动放入 personal-skills/ 目录）

---

## 分支 A：从 SkillHub 安装

### A1. 扫描本地已有的 SkillHub skills

```bash
SKILL_HUB_DIR="$HOME/.agent-skills/skill-hub"
if [ -d "$SKILL_HUB_DIR" ]; then
    for d in "$SKILL_HUB_DIR"/*/; do
        [ -d "$d" ] || continue
        slug=$(basename "$d")
        skill_md="$d/SKILL.md"
        name=$(grep -m1 '^# ' "$skill_md" 2>/dev/null | sed 's/^# //' || echo "$slug")
        link="$SKILLS_DIR/$slug"
        if [ -L "$link" ]; then
            echo "INSTALLED|$slug|$name"
        else
            echo "AVAILABLE|$slug|$name"
        fi
    done
else
    echo "EMPTY"
fi
```

**如果本地有已下载的 SkillHub skills**，用 question 工具展示列表，并询问操作（`[已安装]` 表示已安装到所选目标）：

```
skill-hub/ 中已有以下 skills（标注了安装状态）：

  [1] brave-search   Brave 网页搜索  [已安装]
  [2] github         GitHub 操作     [未安装]
  [3] weather        天气查询        [未安装]

请选择操作：
```
- 选项1（默认）：直接安装（使用本地已有版本）
- 选项2：更新后安装（重新从 SkillHub 下载最新版覆盖，再安装）
- 选项3：下载新技能（从 SkillHub 下载本地没有的技能，下载后安装）

**如果本地没有任何 SkillHub skills**（EMPTY），跳过此询问，直接执行 A3（下载+安装）。

### A2. 直接安装

用 question 工具让用户从列表中选择要安装的 skills（输入编号，多个空格分隔，all=全选，exit=取消）。

对每个选中的 slug 创建软链接（跳至 A6）。

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

### A4. 选择要下载/更新的 skills

**如果是"更新后安装"**：用 question 工具让用户从 A1 展示的本地列表中选择要更新的 skills（输入编号，多个空格分隔，all=全选，exit=取消）。记录选中的 slug 列表，直接进入 A5（不再询问意图）。

**如果是"下载新技能"或首次下载（本地为空）**：用 question 工具询问用户意图：
- 选项1（默认）：浏览精选榜单（默认展示前 10 条）
- 选项2：直接输入技能名称标识（如 `brave-search`、`github`）
- 选项3：搜索关键词
- 选项4：描述我的需求，帮我找合适的技能

**A4a. 浏览精选榜单**：

```bash
curl -fsSL "https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/skills.json" 2>/dev/null | \
  python3 -c "
import json, sys
data = json.load(sys.stdin)
skills = data.get('skills', [])
for s in skills[:10]:
    slug = s.get('slug', '')
    name = s.get('name', slug)
    desc = s.get('description', '')[:60]
    version = s.get('version', '')
    stars = s.get('stars', 0)
    rank = s.get('rank', '')
    print(f'{rank:>2}. [{slug}] {name} v{version} ★{stars}')
    print(f'    {desc}')
"
```

用 question 工具展示，让用户输入编号选择（多个用空格分隔，`more` 加载更多，`exit` 取消）。若输入 `more`，每次在当前基础上 +10 重新获取并展示（10→20→30...最多50），达到50后不再显示 `more` 选项。

**A4b. 直接输入技能名称**：

用 question 工具提示用户输入 slug（支持空格分隔多个，可在 https://skillhub.tencent.com 查看）。

**A4c. 搜索关键词**：

用 question 工具让用户输入关键词，执行：
```bash
skillhub search "<关键词>" 2>&1 | head -80
```
解析结果，用 question 工具展示供选择。

**A4d. 描述需求**：

用 question 工具让用户自然语言描述需求，提取 1-3 个关键词依次搜索，去重合并后展示。

### A5. 执行下载（覆盖已有或新增）

对每个需要下载的 slug：

```bash
TARGET_DIR="$HOME/.agent-skills/skill-hub"
mkdir -p "$TARGET_DIR"

TMP_DIR=$(mktemp -d)
cd "$TMP_DIR"
skillhub install "$SKILL_SLUG" --force 2>&1
cd - > /dev/null

INSTALLED_DIR="$TMP_DIR/skills/$SKILL_SLUG"
if [ -d "$INSTALLED_DIR" ]; then
    rm -rf "$TARGET_DIR/$SKILL_SLUG"
    mv "$INSTALLED_DIR" "$TARGET_DIR/$SKILL_SLUG"
else
    FOUND=$(ls "$TMP_DIR/skills/" 2>/dev/null | head -1)
    [ -n "$FOUND" ] && rm -rf "$TARGET_DIR/$SKILL_SLUG" && mv "$TMP_DIR/skills/$FOUND" "$TARGET_DIR/$SKILL_SLUG"
fi
rm -rf "$TMP_DIR"
```

### A6. 创建软链接

对每个成功下载（或直接安装）的 slug：

```bash
mkdir -p "$SKILLS_DIR"
LINK_PATH="$SKILLS_DIR/$SKILL_SLUG"
[ -L "$LINK_PATH" ] && rm "$LINK_PATH"
ln -s "$HOME/.agent-skills/skill-hub/$SKILL_SLUG" "$LINK_PATH"
```

### A7. 汇报并提醒

列出成功安装/更新、跳过、失败的 skills，提醒用户**重启 OpenCode** 生效。

- 若安装目标为"当前项目"：提示 Skills 软链接在 `$(pwd)/.opencode/skills/<slug>`，可用 `uninstall-skills` 卸载。
- 若安装目标为"全局"：提示 Skills 在任何项目中均可使用，软链接在 `~/.config/opencode/skills/<slug>`。

---

## 分支 B：从 personal-skills 安装

### B1. 扫描 personal-skills/

```bash
LOCAL_SKILLS_DIR="$HOME/.agent-skills/personal-skills"
if [ -d "$LOCAL_SKILLS_DIR" ]; then
    find "$LOCAL_SKILLS_DIR" -maxdepth 2 -name "SKILL.md" | while read f; do
        skill_dir=$(dirname "$f")
        echo "$(basename "$skill_dir")|$(realpath "$skill_dir")"
    done
else
    echo "NOT_FOUND"
fi
```

如果 `personal-skills/` 不存在或为空，告知用户：将 skill 目录（含 `SKILL.md`）手动放入 `~/.agent-skills/personal-skills/` 后重试，终止流程。

### B2. 展示并选择

检查 `$SKILLS_DIR` 已安装情况，用 question 工具展示：

```
personal-skills/ 中可用的 skills（输入编号，多个用空格分隔，all=全选，exit=取消）：

  [1] github [已安装]
  [2] nano-pdf
  [3] skill-vetter
```

### B3. 创建软链接

```bash
mkdir -p "$SKILLS_DIR"
SKILL_DIR="<skill的绝对路径>"
LINK_PATH="$SKILLS_DIR/<skillname>"
if [ ! -L "$LINK_PATH" ]; then
    ln -s "$SKILL_DIR" "$LINK_PATH"
    echo "LINKED: <skillname>"
else
    echo "SKIP: <skillname>（已安装）"
fi
```

### B4. 汇报并提醒

列出安装结果，提醒用户**重启 OpenCode** 生效。

- 若安装目标为"当前项目"：提示 Skills 软链接在 `$(pwd)/.opencode/skills/`。
- 若安装目标为"全局"：提示 Skills 在任何项目中均可使用。

---

## 分支 C：从 git-repo-skills 安装

### C1. 扫描本地已有的 Git repo

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
if [ -d "$GIT_REPO_DIR" ] && [ -n "$(ls -A "$GIT_REPO_DIR" 2>/dev/null)" ]; then
    ls -1 "$GIT_REPO_DIR"
else
    echo "EMPTY"
fi
```

**如果本地有已有的 Git repo**，用 question 工具展示列表，并询问操作：

```
git-repo-skills/ 中已有以下仓库：

  [1] anthropics-skills
  [2] superpowers

请选择操作：
```
- 选项1（默认）：直接使用已有仓库安装（不更新）
- 选项2：git pull 更新后再安装

**如果本地没有任何 Git repo**（EMPTY），跳过此询问，直接执行 C2b（clone 新仓库）。

### C2a. 直接使用已有仓库

用 question 工具让用户选择要使用的仓库（可多选）。进入 C3（扫描 skills）。

### C2b. 更新或 clone

**如果是"git pull 更新后安装"**：用 question 工具让用户选择要更新的仓库（可多选），对每个执行：

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
git -C "$GIT_REPO_DIR/<repo>" pull 2>&1
```

pull 完成后，进入 C3。

**如果本地为空（首次 clone）**：用 question 工具让用户输入 Git 仓库地址，执行：

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
mkdir -p "$GIT_REPO_DIR"
REPO_URL="<用户输入的地址>"
REPO_NAME=$(basename "$REPO_URL" .git)
git clone "$REPO_URL" "$GIT_REPO_DIR/$REPO_NAME" 2>&1
```

clone 完成后，进入 C3。提示：若需持久化管理，建议手动注册为 git submodule。

### C3. 选择要安装的 skills

对用户选中的仓库目录，递归扫描 `SKILL.md`：

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
REPO_DIR="$GIT_REPO_DIR/<repo>"
find "$REPO_DIR" -name "SKILL.md" | while read skill_md; do
    skill_dir=$(dirname "$skill_md")
    skill_name=$(basename "$skill_dir")
    rel_path="${skill_dir#$REPO_DIR/}"
    depth=$(echo "$rel_path" | tr -cd '/' | wc -c)
    if [ "$depth" -eq 0 ]; then
        echo "standalone|$skill_name|$(realpath "$skill_dir")"
    else
        group_skills_dir=$(dirname "$skill_dir")
        echo "grouped|$skill_name|$(realpath "$skill_dir")|$(realpath "$group_skills_dir")"
    fi
done
```

检查 `$SKILLS_DIR` 已安装情况，用 question 工具展示供选择（支持整组或单独安装）：

```
请选择要安装的 skills（输入编号，多个用空格分隔，all=全选，exit=取消）：

[1] anthropics-skills 整组（17 个 skills）[部分已安装]
    [1a] frontend-design [已安装]
    [1b] skill-creator
    ...
[2] superpowers 整组（12 个 skills）
    [2a] brainstorming
    ...
```

- 输入组号 = 整组安装（软链接指向该组 skills 的公共父目录）
- 输入子编号 = 单独安装（软链接指向具体 skill 目录）

### C4. 创建软链接

```bash
mkdir -p "$SKILLS_DIR"
LINK_PATH="$SKILLS_DIR/<skillname>"
[ -L "$LINK_PATH" ] && rm "$LINK_PATH"
ln -s "<skill目录绝对路径>" "$LINK_PATH"
```

整组安装后做去重检查（移除已被整组包含的单独软链接）。

### C5. 汇报并提醒

列出安装结果，提醒用户**重启 OpenCode** 生效。

- 若安装目标为"当前项目"：提示 Skills 软链接在 `$(pwd)/.opencode/skills/`。
- 若安装目标为"全局"：提示 Skills 在任何项目中均可使用。
