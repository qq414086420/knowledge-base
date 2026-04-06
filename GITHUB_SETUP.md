# GitHub 仓库设置指南

## 当前仓库状态

✅ Git 仓库已初始化
✅ 初始提交已创建（commit: 80ba14a）
✅ 分支名称：main

## 上传到 GitHub 的步骤

### 1. 在 GitHub 上创建仓库

1. 访问：https://github.com/new
2. 仓库名称：`knowledge-base`
3. 描述：`Knowledge base for development insights, tool evaluations, and best practices`
4. 可见性：Public
5. **不要**勾选 "Initialize this repository with a README"（我们已经有本地提交了）
6. 点击 "Create repository"

### 2. 添加远程仓库并推送

创建仓库后，GitHub 会显示推送命令。你需要运行：

```bash
cd /mnt/e/code/docs

# 添加远程仓库
git remote add origin https://github.com/qq414086420/knowledge-base.git

# 推送到 GitHub
git push -u origin main
```

或者使用 SSH（如果你配置了 SSH key）：

```bash
git remote add origin git@github.com:qq414086420/knowledge-base.git
git push -u origin main
```

### 3. 验证上传成功

推送完成后，访问你的仓库：
https://github.com/qq414086420/knowledge-base

## 快速命令（复制粘贴）

```bash
cd /mnt/e/code/docs
git remote add origin https://github.com/qq414086420/knowledge-base.git
git push -u origin main
```

## 可选：安装 GitHub CLI

如果你以后想使用 `gh` 命令：

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install gh

# 登录 GitHub
gh auth login

# 创建仓库并推送（一行命令）
gh repo create knowledge-base --public --source=. --push
```
