

---

Docker Images Backup for Jellyfin 使用指南

欢迎使用 docker-images-backup-jellyfin 项目。本项目旨在通过 GitHub Actions 自动化地备份与 Jellyfin 相关的 Docker 镜像，方便你在需要时快速恢复或迁移到其他环境。

目录

1. 项目简介
2. 前置要求
3. 快速开始
4. 工作流详解
5. 配置说明
6. 常见问题

---

1. 项目简介

该项目利用 GitHub 的自动化流水线（GitHub Actions）定期拉取指定的 Jellyfin Docker 镜像，并将其推送到你指定的私有镜像仓库（如阿里云、腾讯云、Harbor 或 Docker Hub），作为异地备份或版本留存的一种手段。

核心文件包括：

· build-n1-jellyfin.yml： 针对特定平台（如 linux/arm64，即 N1 盒子等设备）构建或拉取 Jellyfin 镜像。
· main.yml： 主备份流程，负责镜像的拉取和推送。

2. 前置要求

在使用本工具前，请确保你具备以下条件：

· 一个 GitHub 账号。
· 一个用于存放备份镜像的 目标容器镜像仓库（例如 Docker Hub，或者 阿里云容器镜像服务）。
· 了解基础的 GitHub Actions 和 Docker 概念。

3. 快速开始

按照以下步骤，你可以在 5 分钟内配置好自动备份。

步骤 1: Fork 本仓库

点击本仓库右上角的 Fork 按钮，将项目复制到你自己的 GitHub 账户下。

步骤 2: 配置仓库 Secrets

为了让工作流有权限推送镜像到你的仓库，需要添加密钥（Secrets）。

1. 在你 Fork 后的仓库页面，点击 Settings -> Secrets and variables -> Actions。
2. 点击 New repository secret，添加以下内容：
   · Name: DOCKER_USERNAME （你的镜像仓库用户名）
   · Secret: 你的用户名
   · Name: DOCKER_PASSWORD （你的镜像仓库密码/Token）
   · Secret: 你的密码或访问令牌

步骤 3: 修改配置文件

你需要告诉脚本要把镜像备份到哪里。

1. 打开仓库中的 .github/workflows/main.yml 文件。
2. 找到环境变量部分（通常在 env 字段），将 IMAGE_TARGET 修改为你自己的目标仓库地址。
   · 示例：IMAGE_TARGET: docker.io/你的用户名/jellyfin-backup
   · 示例：IMAGE_TARGET: registry.cn-hangzhou.aliyuncs.com/你的命名空间/jellyfin

步骤 4: 手动触发运行

1. 点击仓库上方的 Actions 标签页。
2. 在左侧列表中选择 main.yml 或 build-n1-jellyfin.yml。
3. 点击 Run workflow 下拉菜单，选择分支（main），然后点击绿色的 Run workflow 按钮。
4. 等待几分钟，查看工作流运行结果。如果显示绿色对勾，说明备份成功。

4. 工作流详解

main.yml (主备份流程)

· 触发方式：通常设置为定时触发（例如每天凌晨）或手动触发。
· 执行逻辑：
  1. 登录到你指定的目标镜像仓库。
  2. 拉取最新的 Jellyfin 官方镜像。
  3. 重新标记（Tag）镜像，指向你的私有仓库。
  4. 将镜像推送到你的私有仓库。

build-n1-jellyfin.yml (N1适配版本)

· 适用场景：如果你使用的是 Phicomm N1 盒子（ARMv8 架构）这类设备，标准的 Jellyfin 镜像可能无法直接运行。此工作流会拉取源码或基础镜像，构建出适用于 ARM 架构的版本并进行备份。

5. 配置说明

修改备份的镜像版本

默认备份的是 latest 标签。如果你想备份特定版本，可以修改工作流文件中的 IMAGE_SOURCE 变量：

```yaml
env:
  IMAGE_SOURCE: jellyfin/jellyfin:10.9.0   # 改为你需要的版本
  IMAGE_TARGET: 你的仓库地址/jellyfin:10.9.0
```

修改定时备份时间

在 main.yml 文件中找到 on -> schedule 部分：

```yaml
on:
  schedule:
    # 每天北京时间凌晨 4 点运行（UTC 时间 20:00）
    - cron: '0 20 * * *'
```

修改 Cron 表达式可以调整备份频率。

6. 常见问题

Q: 为什么备份会失败？

· A: 最常见的原因是 Secrets 配置错误，导致登录镜像仓库失败。请检查 DOCKER_USERNAME 和 DOCKER_PASSWORD 是否正确，以及该账号是否有推送权限。

Q: 如何恢复镜像？

· A: 当你需要使用时，在任何安装有 Docker 的机器上，运行以下命令拉取你的备份镜像即可：
  ```bash
  docker login 你的仓库地址 -u 用户名 -p 密码
  docker pull 你的仓库地址/jellyfin:latest
  ```

Q: 我不想把镜像放在公有仓库怎么办？

· A: 你可以使用 GitHub Packages 作为备份仓库，或者在 Secrets 中配置私有仓库的登录信息，GitHub Actions 同样可以推送到公司内部的私有仓库。

---


