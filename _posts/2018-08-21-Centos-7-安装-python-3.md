---
title: Centos 7 安装 python 3
date: 2018-08-21 16:40:58
categories:
- python
tags:
- python
- linux
---

Centos 7 安装 python 3，保留 python 2 不动

```shell
cd ~
# 安装依赖
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel 
# 创建安装目录
mkdir /usr/local/python3
# 下载相应版本并解压、安装
wget https://www.python.org/ftp/python/3.5.6/Python-3.5.6.tgz
tar -zxvf Python-3.5.6.tgz
cd Python-3.5.6
./configure --prefix=/usr/local/python3
make && make install
# 创建软连接
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
# 检查
python3 --version
pip3 --version
# 升级 pip
pip3 install -U pip
```

