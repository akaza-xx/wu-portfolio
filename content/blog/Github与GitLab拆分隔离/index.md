---
title: "GitHub 与 GitLab 拆分隔离实践"
date: 2026-07-06
summary: "如何在同一台电脑上隔离公司 GitLab 和私人 GitHub，避免账号、SSH Key、提交身份和远程仓库误用。"
tags:
  - Git
  - GitHub
  - GitLab
  - SSH
authors:
  - me
featured: false
---

同一台电脑同时处理公司 GitLab 和私人 GitHub 项目时，最容易出问题的是三件事：提交身份混用、SSH Key 混用、公司代码误推到公开仓库。

推荐做法是把账号、目录、SSH Key、Git identity 和 remote 全部分开。

## 目标结构

```text
~/company/   -> 公司项目，只推公司 GitLab
~/personal/  -> 个人项目，只推私人 GitHub
```

账号建议：

- 公司 GitLab：使用公司邮箱，例如 `user@company.com`
- 私人 GitHub：使用私人邮箱，例如 `user@gmail.com`

## 1. 使用目录隔离 Git 身份

不要把个人身份写成全局默认值。更安全的方式是用 `includeIf` 按目录加载不同配置。

编辑 `~/.gitconfig`：

```gitconfig
[includeIf "gitdir:~/company/**"]
    path = ~/.gitconfig-company

[includeIf "gitdir:~/personal/**"]
    path = ~/.gitconfig-personal
```

个人配置 `~/.gitconfig-personal`：

```gitconfig
[user]
    name = akazan
    email = user@gmail.com
```

公司配置 `~/.gitconfig-company`：

```gitconfig
[user]
    name = CompanyName
    email = user@company.com
```

验证当前仓库实际使用的 Git 身份：

```bash
git config user.name
git config user.email
git config --show-origin user.email
```

`--show-origin` 可以看到配置来自哪个文件，适合排查是否命中了正确的 `includeIf`。

## 2. 分别创建 SSH Key

建议使用 `ed25519`，除非公司环境有兼容性限制。

生成私人 GitHub key：

```bash
ssh-keygen -t ed25519 -C "user@gmail.com" -f ~/.ssh/id_ed25519_github_personal
```

生成公司 GitLab key：

```bash
ssh-keygen -t ed25519 -C "user@company.com" -f ~/.ssh/id_ed25519_gitlab_company
```

把公钥添加到对应平台。

GitHub 公钥：

```bash
cat ~/.ssh/id_ed25519_github_personal.pub
```

添加到：

```text
GitHub -> Settings -> SSH and GPG keys -> New SSH key
```

GitLab 公钥：

```bash
cat ~/.ssh/id_ed25519_gitlab_company.pub
```

添加到：

```text
GitLab -> Preferences -> SSH Keys -> Add key
```

## 3. 配置 SSH Host alias

编辑 `~/.ssh/config`：

```sshconfig
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github_personal
  IdentitiesOnly yes

Host gitlab-company
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab_company
  IdentitiesOnly yes
```

如果公司 GitLab 是自建域名，把 `HostName gitlab.com` 改成公司 GitLab 域名，例如：

```sshconfig
Host gitlab-company
  HostName gitlab.company.com
  User git
  IdentityFile ~/.ssh/id_ed25519_gitlab_company
  IdentitiesOnly yes
```

测试连接时要测试 alias，而不是直接测试真实域名：

```bash
ssh -T git@github-personal
ssh -T git@gitlab-company
```

如果测试 `git@github.com` 或 `git@gitlab.com`，可能不会走你指定的专用 key，结果会误导排查。

## 4. Remote 也使用 alias

私人 GitHub 项目：

```bash
git remote set-url origin git@github-personal:akaza-xx/wu-portfolio.git
```

公司 GitLab 项目：

```bash
git remote set-url origin git@gitlab-company:group/project.git
```

检查 remote：

```bash
git remote -v
```

个人项目应该类似：

```text
origin  git@github-personal:akaza-xx/wu-portfolio.git (fetch)
origin  git@github-personal:akaza-xx/wu-portfolio.git (push)
```

公司项目应该类似：

```text
origin  git@gitlab-company:group/project.git (fetch)
origin  git@gitlab-company:group/project.git (push)
```

## 5. 给公司项目增加 pre-push 防护

在公司项目中创建 `.git/hooks/pre-push`：

```sh
#!/bin/sh

remote_name="$1"
remote_url="$2"

case "$remote_url" in
  *github.com*|*github-personal*)
    echo "Blocked: company project cannot be pushed to GitHub remote: $remote_name <$remote_url>"
    exit 1
    ;;
esac

exit 0
```

赋予执行权限：

```bash
chmod +x .git/hooks/pre-push
```

这个 hook 使用 Git 传入的真实 push 目标 `$2`，比固定检查 `origin` 更可靠。否则下面这种命令可能绕过检查：

```bash
git push github main
```

## 6. 额外检查敏感信息

公开推送到 GitHub 前，可以先搜索常见敏感信息：

```bash
rg -n "password|token|secret|apikey|api_key|AKIA|BEGIN PRIVATE KEY|gitlab|internal|corp" .
```

如果公司内容曾经进入 Git 历史，不要只删除文件再提交。Git 历史里仍然可能保留敏感内容。更安全的方式是新建一个干净公开仓库，只复制允许公开的文件，不复制 `.git` 目录。

## 7. 注意 hook 不是强安全边界

`pre-push` hook 只能降低误操作概率，不能作为公司代码保护的唯一手段。

它可以被绕过：

```bash
git push --no-verify
```

真正的强约束应该放在：

- 公司 GitLab 权限管理
- 保护分支
- 代码扫描
- Secret scanning
- 组织级仓库策略
- 严格的公开内容审查

## 推荐最终状态

```text
公司项目目录：~/company/
公司 Git 身份：~/.gitconfig-company
公司 SSH key：~/.ssh/id_ed25519_gitlab_company
公司 remote：git@gitlab-company:group/project.git

个人项目目录：~/personal/
个人 Git 身份：~/.gitconfig-personal
个人 SSH key：~/.ssh/id_ed25519_github_personal
个人 remote：git@github-personal:akaza-xx/wu-portfolio.git
```

核心原则：公司代码留在公司 GitLab，公开 GitHub 只放脱敏后的个人作品集、项目介绍和可公开内容。
