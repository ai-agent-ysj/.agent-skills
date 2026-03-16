---
description: 下载 skills 到 ~/.agent-skills/（支持 SkillHub 下载、Git repo clone），不创建软链接
---

# 下载 Skills

你的任务是帮用户把 skills 下载/克隆到 `~/.agent-skills/` 中央仓库，**不创建任何软链接**。  
下载完成后，可通过 `/install-skills` 安装到具体位置。

**路径约定**：
- `SKILL_HUB_DIR="$HOME/.agent-skills/skill-hub"` — SkillHub 下载目标
- `GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"` — Git 仓库克隆目标

---

## 执行步骤

### 1. 选择下载来源

使用 question 工具询问用户：

```
请选择下载来源：
```
- 选项1（默认）：SkillHub（在线下载，国内高速，收录 1.3 万个 AI Skills）
- 选项2：Git 仓库（clone 到 git-repo-skills/）

---

## 分支 A：从 SkillHub 下载

### A1. 检查 skillhub CLI 是否已安装

```bash
which skillhub 2>/dev/null && skillhub -v 2>&1 || echo "NOT_INSTALLED"
```

如果未安装（输出 `NOT_INSTALLED`），使用 question 工具询问：
- 选项1（默认）：立即安装 skillhub CLI
- 选项2：取消

选择"取消"则终止流程，提示用户手动安装：
```
curl -fsSL https://skillhub-1388575217.cos.ap-guangzhou.myqcloud.com/install/install.sh | bash -s -- --cli-only
```

选择"安装"则执行上述命令，安装完成后重新验证，若仍失败则终止并告知用户。

### A2. 获取用户意图

使用 question 工具询问：
- 选项1（默认）：浏览精选榜单（默认展示前 10 条）
- 选项2：直接输入技能名称标识（如 `brave-search`、`github`，可在 https://skillhub.tencent.com 查看）
- 选项3：搜索关键词
- 选项4：描述我的需求，帮我找合适的技能

### A3a. 浏览精选榜单

默认展示前 10 个精选技能：

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

用 question 工具展示，让用户输入编号选择（多个用空格分隔，`more` 加载更多，`exit` 取消）。

若用户输入 `more`，每次在当前基础上 +10 重新获取并展示（10→20→30...最多50），达到50后不再显示 `more` 选项。

### A3b. 直接输入技能名称

用 question 工具提示用户输入技能的名称标识（即 slug）：

```
请输入要下载的技能名称标识（支持同时输入多个，用空格分隔）：

示例：
  brave-search          → Brave 网页搜索技能
  github                → GitHub 操作技能
  brave-search github   → 同时下载以上两个

可在 https://skillhub.tencent.com 查看所有技能名称
```

### A3c. 搜索关键词

用 question 工具让用户输入关键词，执行：

```bash
skillhub search "<关键词>" 2>&1 | head -80
```

解析结果，用 question 工具展示供用户选择。

### A3d. 描述需求，智能匹配技能

用 question 工具让用户用自然语言描述想解决的问题，例如：

> "我想让 AI 能搜索网络"
> "帮我处理 PDF 文件"
> "我需要查看 GitHub 上的 issue 和 PR"

根据用户描述，理解其核心意图，提取 1-3 个最相关的搜索关键词，然后依次执行搜索：

```bash
skillhub search "<关键词1>" 2>&1 | head -80
skillhub search "<关键词2>" 2>&1 | head -80  # 如有多个关键词
```

对搜索结果去重合并，按相关度排序，用 question 工具展示给用户，并在列表上方用一句话说明"根据您的描述，为您找到以下相关技能"。若搜索结果为空，直接告知用户未找到相关技能，询问是否换个关键词重试。

### A4. 检查已有，确认覆盖

```bash
TARGET_DIR="$HOME/.agent-skills/skill-hub"
mkdir -p "$TARGET_DIR"
for SLUG in <选中的slugs>; do
    if [ -d "$TARGET_DIR/$SLUG" ]; then echo "EXISTS: $SLUG"; else echo "NEW: $SLUG"; fi
done
```

如果有已存在的 skill，用 question 工具**一次性**展示所有冲突，询问：
- 选项1（默认）：全部覆盖更新
- 选项2：跳过已存在，只下载新的
- 选项3：取消

### A5. 执行下载

对每个需要下载（或覆盖）的 slug：

```bash
TARGET_DIR="$HOME/.agent-skills/skill-hub"
mkdir -p "$TARGET_DIR"

# skillhub install 默认安装到当前目录的 skills/ 下，因此切换到临时目录执行
TMP_DIR=$(mktemp -d)
cd "$TMP_DIR"
skillhub install "$SKILL_SLUG" --force 2>&1
cd - > /dev/null

# 将安装结果移到 skill-hub/
INSTALLED_DIR="$TMP_DIR/skills/$SKILL_SLUG"
if [ -d "$INSTALLED_DIR" ]; then
    rm -rf "$TARGET_DIR/$SKILL_SLUG"
    mv "$INSTALLED_DIR" "$TARGET_DIR/$SKILL_SLUG"
else
    # 兜底：找 skills/ 下第一个目录
    FOUND=$(ls "$TMP_DIR/skills/" 2>/dev/null | head -1)
    [ -n "$FOUND" ] && rm -rf "$TARGET_DIR/$SKILL_SLUG" && mv "$TMP_DIR/skills/$FOUND" "$TARGET_DIR/$SKILL_SLUG"
fi
rm -rf "$TMP_DIR"
```

### A6. 汇报

列出成功下载/更新、跳过、失败的 skills，告知用户文件已存放在 `~/.agent-skills/skill-hub/`。

> 下一步：运行 `/install-skills` 将这些 skills 安装到具体位置。

---

## 分支 B：从 Git 仓库 Clone

### B1. 获取仓库地址

用 question 工具让用户输入 Git 仓库地址（如 `https://github.com/xxx/skills.git`）。

### B2. Clone 仓库

```bash
GIT_REPO_DIR="$HOME/.agent-skills/git-repo-skills"
mkdir -p "$GIT_REPO_DIR"
REPO_URL="<用户输入的地址>"
REPO_NAME=$(basename "$REPO_URL" .git)

# 检查是否已存在
if [ -d "$GIT_REPO_DIR/$REPO_NAME" ]; then
    echo "EXISTS: git-repo-skills/$REPO_NAME"
else
    git clone "$REPO_URL" "$GIT_REPO_DIR/$REPO_NAME"
fi
```

若仓库已存在，用 question 工具询问：
- 选项1（默认）：pull 更新已有仓库
- 选项2：跳过
- 选项3：取消

若选择更新：
```bash
git -C "$GIT_REPO_DIR/$REPO_NAME" pull 2>&1
```

### B3. 汇报

告知用户仓库已克隆/更新到 `~/.agent-skills/git-repo-skills/<repo-name>/`。

> 下一步：运行 `/install-skills` 从该仓库选择 skills 安装。

> 提示：若需持久化管理，建议手动注册为 git submodule：
> `git submodule add <url> git-repo-skills/<name>`
