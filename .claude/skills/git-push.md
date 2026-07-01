---
name: git-push
description: 自动将本地项目提交并推送到 GitHub。初始化 git、创建仓库、生成 commit message、推送一气呵成。
version: 1.0.0
---

# git-push

自动将本地项目推送到 GitHub 的完整工作流。处理 git 初始化、仓库创建、智能提交信息生成和推送。

## 执行流程

### Step 1: 检查 Git 仓库状态

```bash
git status 2>&1
```

- 如果当前目录不是 git 仓库（`fatal: not a git repository`），执行 `git init` 初始化
- 如果是仓库，记录当前分支名和状态

### Step 2: 检查 Remote 配置

```bash
git remote -v
```

- 如果已配置 `origin`，直接跳到 Step 4
- 如果没有 `origin`：
  - **优先使用 `gh` CLI 创建仓库**（如果已安装且已登录）：
    ```bash
    gh repo create <repo-name> --private --source=. --remote=origin --push
    ```
    如果创建+推送成功，流程结束。
  - **如果没有 `gh` CLI**：询问用户提供 GitHub 仓库 URL（格式：`https://github.com/user/repo.git` 或 `git@github.com:user/repo.git`），然后执行：
    ```bash
    git remote add origin <url>
    ```

### Step 3: 添加 GitHub 仓库（手动方式）

如果用户提供了仓库 URL：
```bash
git remote add origin <url>
```

验证 remote 是否添加成功：
```bash
git remote -v
```

### Step 4: 暂存所有变更

```bash
git add .
```

### Step 5: 生成 Commit Message

查看变更内容，自动生成有意义的中文 commit message：

```bash
git diff --cached --stat
git diff --cached --name-only
```

根据 diff 内容智能生成 commit message，遵循以下规则：
- 新项目初始化 → `chore: 初始化项目`
- 新功能 → `feat: <功能描述>`
- 修复 → `fix: <修复描述>`
- 重构 → `refactor: <重构描述>`
- 文档 → `docs: <文档描述>`
- 配置 → `chore: <配置描述>`
- 多文件混合变更 → 根据主要变更内容生成概括性 message

### Step 6: 提交

```bash
git commit -m "<生成的commit message>"
```

如果没有任何变更（working tree clean），提示用户并结束。

### Step 7: 推送到 GitHub

```bash
git push -u origin <branch-name>
```

### Step 8: 错误处理

常见错误及处理方式：

| 错误 | 原因 | 处理 |
|------|------|------|
| `Permission denied (publickey)` | SSH key 未配置 | 提示用户配置 SSH key 或改用 HTTPS |
| `remote contains work that you do not have` | 远程有本地没有的提交 | 建议先 `git pull` 再 push |
| `failed to push some refs` | 推送冲突 | 展示具体错误，提示解决方案 |
| `Repository not found` | 仓库不存在或无权限 | 确认仓库 URL 是否正确，权限是否足够 |
| `fatal: branch 'main' has no upstream` | 首次推送缺少上游 | 使用 `git push -u origin <branch>` |
| `gh: command not found` | gh CLI 未安装 | 提示用户手动提供仓库 URL 或安装 gh CLI |

### Step 9: 输出结果

成功时输出：
- ✅ 提交的 commit message
- ✅ 推送的分支名
- ✅ 仓库 URL（如已知）

## 注意事项

1. **在 push 之前不执行 `git pull`**，避免自动合并产生意外。如果远程有冲突，明确告知用户。
2. **默认推送当前分支**，不会强制推送（不使用 `--force`）。
3. **提交前检查是否有实质性变更**，避免空提交。
4. **gh CLI 创建仓库时默认使用 `--private`**，用户可要求改为 `--public`。
