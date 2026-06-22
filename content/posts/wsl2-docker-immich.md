---
title: "WSL2 跑 Docker，顺手装了 Immich"
date: 2026-06-22
tags: [WSL2, Docker, Immich, 自托管, Linux]
description: "在 Windows 上用 WSL2 跑 Docker：无桌面端安装 Immich 私有相册全记录"
---

最近折腾了一下在 Windows 上用 WSL2 跑 Docker，全程不依赖 Docker Desktop 或任何桌面端工具，纯命令行完成。顺带把 Immich 私有相册也部署起来了。这篇文章记录完整过程，适合第一次上手的朋友参考。

## 为什么不用 Docker Desktop

Docker Desktop 本身没什么问题，但它是 GUI 应用，占资源，而且商业使用需要付费授权。WSL2 里直接跑 Docker Engine 更轻量，和 Linux 原生体验几乎一致，也更适合服务器思维的使用方式。

---

## 一、安装 WSL2 并把 Ubuntu 装到 D 盘

默认情况下，WSL 会把发行版装在 C 盘。如果你的 C 盘空间有限，可以在安装时直接指定路径。

以管理员身份打开 PowerShell，执行：

```powershell
wsl --install Ubuntu --web-download --location "D:\WSL\Ubuntu"
```

如果网络不好导致下载失败，可以手动下载安装包再安装：

```powershell
Invoke-WebRequest -Uri https://aka.ms/wslubuntu -OutFile Ubuntu.appx
Add-AppxPackage .\Ubuntu.appx
wsl --install Ubuntu --location "D:\WSL\Ubuntu"
```

安装完成后会自动进入 Ubuntu 初始化界面，按提示创建用户名和密码。输密码时屏幕不会有任何显示，这是正常的，直接输入按 Enter 就好。

---

## 二、基础系统配置

进入 Ubuntu 后先更新软件包，再装几个常用工具：

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim build-essential net-tools
```

顺手把默认目录设到家目录，避免每次打开都在根目录：

```bash
cd ~
echo "cd ~" >> ~/.bashrc
source ~/.bashrc
```

---

## 三、安装 Docker Engine

WSL2 环境下按照 Docker 官方方式安装 Docker Engine，不需要 Docker Desktop。

```bash
# 先卸载可能存在的旧版本
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove -y $pkg
done

# 添加 Docker 官方 GPG 密钥和软件源
sudo apt update && sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
sudo apt update && sudo apt install -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# 把当前用户加入 docker 组，避免每次都要 sudo
sudo usermod -aG docker $USER
```

重新登录（或执行 `newgrp docker`）后，测试是否安装成功：

```bash
docker run hello-world
```

设置 Docker 开机自启动：

```bash
sudo systemctl enable --now docker
```

---

## 四、配置国内镜像加速

Docker Hub 在国内访问速度很慢，甚至经常超时。可以配置镜像加速器来解决这个问题。

编辑（或新建）Docker 的 daemon 配置文件：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://mirror.baidubce.com"
  ]
}
EOF
```

重启 Docker 使配置生效：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

验证镜像加速是否生效：

```bash
docker info | grep -A 5 "Registry Mirrors"
```

> **说明：** 国内镜像加速节点时效性较短，若某个地址失效，可以替换为其他可用节点。DaoCloud 和百度云目前稳定性相对较好。

---

## 五、部署 Immich 私有相册

[Immich](https://immich.app) 是一个开源的自托管相册方案，功能对标 Google Photos：自动备份、人脸识别、地图视图、时间线浏览一应俱全，而且数据完全在自己手里。

### 1. 创建目录并下载配置文件

```bash
mkdir -p ~/immich && cd ~/immich

# 下载官方 docker-compose 配置
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### 2. 修改环境变量

用 vim 或 nano 打开 `.env` 文件，修改以下两项：

```bash
vim .env
```

关键配置项：

```env
# 相册数据存储路径，改成你想要的目录
UPLOAD_LOCATION=./library

# 数据库密码，建议改成一个强密码
DB_PASSWORD=your_strong_password_here
```

如果你想把照片存在 Windows 的某个盘（比如 H 盘），可以这样设置路径：

```env
UPLOAD_LOCATION=/mnt/h/ImmichLibrary
```

WSL2 会自动把 Windows 的 H 盘挂载到 `/mnt/h`。

### 3. 启动 Immich

```bash
docker compose up -d
```

第一次启动会拉取多个镜像，国内环境配好加速后一般 5-10 分钟内完成。

### 4. 访问 Web 界面

启动成功后，在 Windows 浏览器中访问：

```
http://localhost:2283
```

第一次访问需要注册管理员账号。注册完成后就可以开始使用了。

### 5. 常用管理命令

```bash
# 查看运行状态
docker compose ps

# 查看日志
docker compose logs -f

# 停止服务
docker compose down

# 更新到最新版本
docker compose pull && docker compose up -d
```

---

## 六、一些使用注意事项

**存储位置：** Ubuntu 系统文件和通过 `apt` 安装的软件都在 `D:\WSL\Ubuntu` 下，不占 C 盘空间。

**访问 Windows 文件：** 在 Ubuntu 中，`C:\` 对应 `/mnt/c/`，`D:\` 对应 `/mnt/d/`，以此类推。

**WSL 常用命令（在 PowerShell 里执行）：**

```powershell
wsl --list --verbose   # 查看已安装的发行版和状态
wsl --shutdown         # 关闭所有 WSL 实例
```

**资源占用：** Immich 空闲时内存占用约 500MB-1GB，主要是数据库和机器学习服务。如果机器内存紧张，可以在 `.env` 中禁用机器学习功能（会失去人脸识别和智能搜索）。

**手机端备份：** Immich 有官方 iOS 和 Android 客户端，设置服务器地址为 `http://[你的Windows内网IP]:2283` 即可自动备份手机相册。查看 Windows 内网 IP 可以在 PowerShell 里执行 `ipconfig`。

---

第一次搭 WSL2 + Docker 的组合，整个过程下来比预想的顺利很多。没有桌面端的加持反而让整个环境更干净，理解也更深。Immich 的体验也超出预期，推荐有私有部署需求的朋友试试。
