---
title: Docker 安装 mysql
date: 2018-08-21 17:07:21
categories:
- docker
tag:
- docker
- mysql
---

docker 安装 mysql

```shell
# 创建主机挂载目录
mkdir -p /data/mysql

# 搜索并下载 mysql docker image，tag=5.7.19
docker search mysql
docker pull mysql:5.7.19

# 运行容器
# 将容器的 3306 端口映射到主机的 13306
# 将主机当前目录下的 /data/mysql 挂载到容器的 /var/lib/mysql
# -d 表示容器在后台运行
docker run -p 13306:3306 -v /data/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${init_root_passwd} --name ${container_name} -d ${docker_image_id}
# 查看 container
docker ps
```

进入容器配置 mysql 远程访问

```shell
# -t 让docker分配一个伪终端，并绑定到容器的标准输入上
# -i 让容器的标准输入保持打开
docker exec -it ${container_name} bash

mysql -u root -p
Enter password:
mysql> grant all privileges on *.* to root@'%' identified by 'passwd' with grant option;
mysql> flush privileges;
```

需要修改 mysql 配置文件，但是在使用 docker 容器时，有时候里边没有安装vi，敲vi命令时提示说：vi: command not found，这个时候就需要安装  `apt-get install vi` 

提示 `E: Unable to locate package vi`

输入 `apt-get update` 作用是同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。  

再次 `apt-get install vi` 即可

