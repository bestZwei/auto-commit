# GitHub 自动提交

## 项目简介

本项目利用 GitHub Actions 工作流实现自动提交 GitHub 项目，帮助您点亮 GitHub 贡献图（GitHub Contributions Graph）。通过定时提交保持仓库的活跃度，让贡献图不断更新。

### 特点
- 每天定时自动运行，按照设置的时间（如每天美国时间凌晨 0:00）执行。
- 根据随机概率控制提交（0-6 的概率），如果需要提交，则随机生成 1-3 个空提交（empty commit），并将其推送到 GitHub 仓库。

## 使用方法

1. **Fork 项目**
   - 将本项目 Fork 到您的 GitHub 仓库，随便取一个名字。

2. **配置操作权限**
   - 进入您的 GitHub 仓库。
   - 在仓库页面，点击 `Settings`。
   - 在左侧菜单中选择 `Actions` -> `General`。
   - 确保勾选 `Allow GitHub Actions to create and approve pull requests` 和 `Read and write permissions`。

3. **配置读取和写入权限**
   - 在同一个 `Actions` 设置页面，确保勾选 `Read and write permissions`。

4. **配置提交者用户名和邮箱**
   - 打开 `.github/workflows/main.yml` 文件。
   - 修改 `git config` 部分以使用您的用户名和邮箱。

5. **手动触发一次工作流进行测试**
   - 在仓库页面，点击 `Actions` 选项卡。
   - 找到工作流 `bestZwei`，点击 `Run workflow` 按钮。
   - 观察日志以确保工作流成功运行，并且有新的提交记录。

> **注意** 
>
> 提交者的 **邮箱** 和 **账号** 必须配置成自己的，不然无法成功统计 commit，别到时候给别人提交 commit 了。
>
> - 邮箱地址可以填您 **GitHub 绑定的邮箱地址**。
> - 也可以填您的 **私密邮箱**。

## 如何设置私密邮箱

要隐藏您的电子邮件，可以通过 GitHub 的用户名设置一个 `noreply` 邮箱地址，这样就可以保护您的真实电子邮件地址。GitHub 提供了一个类似 `username@users.noreply.github.com` 的邮箱，您可以在 GitHub 的邮箱设置中找到并使用这个邮箱。

具体的步骤如下：

1. **在 GitHub 设置邮箱**：
   - 登录到 GitHub。
   - 进入 [GitHub Settings](https://github.com/settings/emails)。
   - 在 "Primary email address" 下，启用 "Keep my email address private"（保持我的电子邮件地址私密）。
   - 选择一个类似 `username@users.noreply.github.com` 的邮件地址。

2. **修改 Git 配置**：
   - 更新您的 Git 配置文件，使用 GitHub 提供的 `noreply` 邮箱地址。您可以在 `workflow` 中的 `git config` 步骤中修改为这个 `noreply` 邮箱：

   ```yaml
   git config --local user.email "your-username@users.noreply.github.com"
   git config --local user.name "ikunycj"
   ```

   这样，每次提交时，GitHub 会使用 `noreply` 邮箱来代替您个人的邮箱地址，保证隐私。

## 示例配置

以下是一个示例的 `.github/workflows/main.yml` 配置文件，您可以参考它进行修改：

```yaml
name: bestZwei
on:
  workflow_dispatch:
  push:
    branches:
      - main
  schedule:
    - cron: "0 0 * * *" # 每天的 0:00 UTC 执行

jobs:
  autogreen:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v4

      - name: Setup random commit script
        run: |
          echo '#!/bin/bash' > random_commit.sh
          echo 'RANDOM_PROB=$(( RANDOM % 7 ))' >> random_commit.sh     # 每日提交概率[0,6]
          echo 'if [[ $RANDOM_PROB -ne 0 ]]; then' >> random_commit.sh
          echo '  RANDOM_NUM=$(( 1 + RANDOM % 3 ))' >> random_commit.sh  # 每次提交数目[1,3]'
          echo '  for i in $(seq 1 $RANDOM_NUM); do' >> random_commit.sh
          echo '    git commit --allow-empty -m "Random empty commit $i"' >> random_commit.sh
          echo '  done' >> random_commit.sh
          echo 'fi' >> random_commit.sh
          chmod +x random_commit.sh

      - name: Auto commit with random number
        run: |
          git config --local user.email "your-username@users.noreply.github.com" 
          git config --local user.name "your-username"
          git remote set-url origin https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/${{ github.repository }}
          git pull --rebase
          ./random_commit.sh
          git push
```


