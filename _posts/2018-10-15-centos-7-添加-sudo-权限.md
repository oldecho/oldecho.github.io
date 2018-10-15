---
title: centos 7 添加 sudo 权限
date: 2018-10-15 23:30:07
categories:
- linux
tag:
- centos 7
---

centos 7 给普通用户添加 sudo 权限

在 root 用户下使用命令 `visudo` 即可编辑文件 `/etc/sudoers` 

打开后可以看到第一段注释有写到

```
## This file must be edited with the 'visudo' command.
```

找到

```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

在下面添加一行

```
username    ALL=(ALL)       ALL
```

保存退出