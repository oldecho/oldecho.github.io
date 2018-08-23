---
title: Centos 7 安装 Docker CE
date: 2018-08-21 17:07:21
categories:
- docker
tag:
- docker
- linux
---

Centos 7 安装 Docker CE

```shell
# 卸载旧版本
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine
# 安装依赖
yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
# 添加 yum 源（国内源）
yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
# 获取最新版本的 docker ce
yum-config-manager --enable docker-ce-edge
yum makecache fast
yum install -y docker-ce
# 启动 docker ce
systemctl enable docker
systemctl start docker
# 建立 docker 用户组，将需要使用 docker 的用户加入 docker 用户组
groupadd docker
# usermod -aG docker $USER
# 测试
docker run hello-world
# Unable to find image 'hello-world:latest' locally
# latest: Pulling from library/hello-world
# 9db2ca6ccae0: Pull complete 
# Digest: sha256:4b8ff392a12ed9ea17784bd3c9a8b1fa3299cac44aca35a85c90c5e3c7afacdc
# Status: Downloaded newer image for hello-world:latest
# 
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
# 
# To generate this message, Docker took the following steps:
#  1. The Docker client contacted the Docker daemon.
#  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
#     (amd64)
#  3. The Docker daemon created a new container from that image which runs the
#     executable that produces the output you are currently reading.
#  4. The Docker daemon streamed that output to the Docker client, which sent it
#     to your terminal.
# 
# To try something more ambitious, you can run an Ubuntu container with:
#  $ docker run -it ubuntu bash
# 
# Share images, automate workflows, and more with a free Docker ID:
#  https://hub.docker.com/
# 
# For more examples and ideas, visit:
#  https://docs.docker.com/engine/userguide/
```

