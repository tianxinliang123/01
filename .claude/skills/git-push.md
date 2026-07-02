---
name: git-push
description: 自动将本地项目提交并推送到 GitHub。初始化 git、配置用户信息、生成 commit message、推送、捕获 commitID 一气呵成。
version: 2.0.0
---

# git-push

自动将本地项目推送到 GitHub 的完整工作流。处理 git 初始化、用户配置、仓库创建、智能提交信息生成、推送和 commitID 捕获。

## 参数

- `$1` — 可选的 commit message。如果提供，直接使用；如果省略，根据 diff 内容自动生成。

## 执行流程

### Step 1: 配置 Git 用户信息（项目级）

确保项目级 git 配置已设置（使用 `git config` 不带 `--global`，仅影响当前仓库）：

```bash
git config user.name "txl"
git config user.email "512280751@qq.com"
```

验证配置：
```bash
git config user.name && git config user.email
```

### Step 2: 检查 Git 仓库状态

```bash
git status 2>&1
```

- 如果当前目录不是 git 仓库（`fatal: not a git repository`），执行 `git init` 初始化
- 如果是仓库，记录当前分支名和状态

### Step 3: 检查 Remote 配置

```bash
git remote -v
```

- 如果已配置 `origin`，跳到 Step 5
- 如果没有 `origin`：
  - **优先使用 SSH 方式**：项目默认远程地址格式为 `git@github.com:tianxinliang123/{项目名}.git`
  - **其次使用 `gh` CLI 创建仓库**（如果已安装且已登录）：
    ```bash
    gh repo create <repo-name> --private --source=. --remote=origin --push
    ```
    如果创建+推送成功，跳到 Step 7
  - **如果没有 `gh` CLI**：询问用户提供 GitHub 仓库 URL，然后执行：
    ```bash
    git remote add origin <url>
    ```

### Step 4: 添加 GitHub 仓库（手动方式）

如果用户提供了仓库 URL：
```bash
git remote add origin <url>
```

验证 remote 是否添加成功：
```bash
git remote -v
```

如果用户希望使用 SSH 方式推送，设置远程地址为：
```bash
git remote set-url origin git@github.com:tianxinliang123/<项目名>.git
```

### Step 5: 暂存所有变更

```bash
git add .
```

注意：`.gitignore` 仅排除 node_modules 和训练记录文件；所有模型生成的文件（包括调试脚本、测试文件、构建输出）均按原样提交。

### Step 6: 生成 Commit Message

如果调用时已提供 commit message（如 `$1`），直接使用。

否则查看变更内容，自动生成有意义的中文 commit message：

```plain
## Current Context (Auto-detected)

Current branch:
!`git branch --show-current`

Recent commits on this branch:
!`git log origin/main..HEAD --oneline 2>/dev/null || echo "No commits ahead of main"`

Files changed:
!`git diff --stat origin/main 2>/dev/null || git diff --stat HEAD~3`
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
- 如果提供了 sessionID → 直接使用原始 sessionID 字符串作为 commit message

### Step 7: 提交

```bash
git commit -m "<commit message>"
```

如果没有任何变更（working tree clean），提示用户并结束。

### Step 8: 推送到 GitHub

```bash
git push -u origin <branch-name>
```

- 通过 SSH 使用 `git@github.com:tianxinliang123/{项目名}.git` 推送
- 如果推送失败（例如没有配置远程仓库），回退到向用户索要 commitID

### Step 9: 捕获 CommitID（阻塞步骤）

推送成功后，通过 `git rev-parse HEAD` 捕获完整提交哈希：

```bash
git rev-parse HEAD
```

- 记录的 `commitID` 必须是 `git rev-parse HEAD` 返回的完整 40 字符哈希
- 来自 `git log --oneline`、`git show --stat` 或 UI 摘要的短哈希禁止使用
- 如果推送失败，回退到向用户索要 commitID
- 在 commitID 确认之前不要继续后续操作

### Step 10: 错误处理

常见错误及处理方式：

| 错误 | 原因 | 处理 |
|------|------|------|
| `Permission denied (publickey)` | SSH key 未配置 | 提示用户配置 SSH key 或改用 HTTPS |
| `remote contains work that you do not have` | 远程有本地没有的提交 | 建议先 `git pull --rebase` 再 push |
| `failed to push some refs` | 推送冲突 | 展示具体错误，提示解决方案 |
| `Repository not found` | 仓库不存在或无权限 | 确认仓库 URL 是否正确，权限是否足够 |
| `fatal: branch 'main' has no upstream` | 首次推送缺少上游 | 使用 `git push -u origin <branch>` |
| `gh: command not found` | gh CLI 未安装 | 提示用户手动提供仓库 URL 或安装 gh CLI |
| `Could not resolve hostname` | SSH 配置问题 | 确认 SSH key 已添加到 GitHub 账户 |

### Step 11: 输出结果

成功时输出：
- ✅ 提交的 commit message
- ✅ 推送的分支名
- ✅ 仓库 URL（如已知）
- ✅ 完整 40 字符 commitID：`<commit_hash>`

## 工作流顺序（硬规则）

严格按照以下顺序执行，不要跳过任何步骤：

1. 配置 git 用户信息（`git config user.name` / `user.email`）
2. 检查仓库状态和 remote 配置
3. `git add .` 暂存所有变更
4. `git commit -m "<message>"` 提交
5. `git push` 推送到远程仓库
6. `git rev-parse HEAD` 捕获完整 40 字符提交哈希
7. 输出 commitID

## 注意事项

1. **推送前配置 git 用户信息**，确保使用正确的身份：`txl / 512280751@qq.com`
2. **在 push 之前不执行 `git pull`**，避免自动合并产生意外。如果远程有冲突，明确告知用户。
3. **默认推送当前分支**，不会强制推送（不使用 `--force`）。
4. **提交前检查是否有实质性变更**，避免空提交。
5. **gh CLI 创建仓库时默认使用 `--private`**，用户可要求改为 `--public`。
6. **优先使用 SSH 方式**（`git@github.com:tianxinliang123/{项目名}.git`），与训练工作流保持一致。
7. **commitID 必须是完整 40 字符哈希**，短哈希在任何记录中禁止使用。
8. **推送失败时回退**：向用户索要 commitID，而不是静默跳过。
