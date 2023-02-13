---
tags: [Linux]
title: docker containerd 安装
created: '2022-05-28T08:31:13.096Z'
modified: '2022-05-28T08:32:24.087Z'
---

# docker containerd 安装

警告：切勿在没有配置 Docker APT 源的情况下直接使用 apt 命令安装 Docker.

## Debian 安装

[官方](https://docs.docker.com/engine/install/debian/)
[中文](https://yeasy.gitbook.io/docker_practice/install/debian)

## CentOS 安装

```
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io
```

## 卸载旧版本

旧版本的 `Docker` 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```
$ sudo apt-get remove docker \
               docker-engine \
               docker.io
```

## 使用 APT 安装

新版的容器已经改名 `docker-ce` 运行容器 `containerd.io`：

```
# apt install apt-transport-https ca-certificates curl gnupg lsb-release

# curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# apt update && apt install docker-ce docker-ce-cli containerd.io
```

启动 Docker

## 配置中国镜像和修改默认路径

```
# vim /etc/docker/daemon.json
{
    "registry-mirrors": ["https://vm5nawb6.mirror.aliyuncs.com", "https://mirror.ccs.tencentyun.com"],
    "data-root": "/data01/docker/lib"
}

# mv /var/lib/docker/* /new/path/
```

## 配置权限

```
groupadd docker
usermod -aG docker dev
```

## CentOS 安装

### 配置自动补全

```
# ~/.bashrc
# For docker completion
source /usr/share/bash-completion/bash_completion
source /usr/share/bash-completion/completions/docker
```


