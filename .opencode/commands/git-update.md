---
description: 检测新子仓库并更新所有 skills 子模块到最新版本
---

请执行以下操作来更新所有 skills：

## 1. 检查当前 git 状态

- 如果工作区有未提交的更改（staged 或 unstaged），立即停止并报告未提交的文件
- 未跟踪的文件可以忽略，不影响后续操作

## 2. 检测未注册的子仓库

扫描项目根目录下所有包含 `.git` 的一级子目录，对比 `.gitmodules` 中已注册的子模块，找出手动 clone 但还未注册为 submodule 的仓库。

具体步骤：

1. 列出根目录下所有包含 `.git` 目录或文件的一级子目录（排除 `.git` 本身）
2. 读取 `.gitmodules` 获取已注册的子模块 path 列表
3. 对比两者，找出未注册的子仓库
4. 对于每个未注册的子仓库：
   - 读取其 remote origin URL：`git -C <dir> remote get-url origin`
   - 如果能获取到 URL，先**删除该目录**，然后执行 `git submodule add <url> <dir>` 将其注册为子模块
   - 如果获取不到 URL，跳过并报告该目录无法自动添加
5. 如果有新增子模块，报告新增了哪些

> 注意：`git submodule add` 要求目标目录不存在，所以必须先删除手动 clone 的目录再添加。
> 删除前确认该目录是一个干净的 git clone（没有未推送的本地提交或未提交的修改），如果不干净则跳过并警告用户。

## 3. 拉取主仓库最新代码

```bash
git pull
```

## 4. 初始化并更新所有子模块到远端最新版本

```bash
git submodule update --init --recursive
git submodule update --remote --recursive
```

- `--init` 确保首次克隆时也能自动初始化子模块
- `--remote` 拉取子模块远端最新提交
- `--recursive` 处理嵌套子模块

## 5. 检查是否有变更需要提交

- 运行 `git status --short` 查看所有子模块目录和 `.gitmodules` 是否有变更
- 如果没有任何变更，报告"所有子模块已是最新版本，无需提交"，然后结束

## 6. 提交变更

```bash
git add .gitmodules
git add */  # 添加所有子模块目录
git commit -m "更新 skills 子模块到最新版本"
```

> 注意：不要硬编码子模块名称，使用动态方式添加所有已注册子模块的变更。

## 7. 总结更新内容

- 新增了哪些子模块（如有）
- 各子模块的旧 commit → 新 commit（如有更新）
- 各子模块的主要更新内容（如有）

## 8. 不要执行 `git push`，除非用户明确要求
