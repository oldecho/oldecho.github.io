---
title: osx 重置 mysql root 密码
date: 2018-11-05 13:30:07
categories:
- osx
tag:
- mac
---

实验环境 mysql 5.7.24 for osx 10.14

1. 在系统偏好设置中停止 mysql 服务

2. 以安全模式启动 mysql

   ```
   cd /usr/local/mysql/bin
   sudo ./mysqld_safe --skip-grand-tables
   ```

3. 新建一个命令行窗口，无密码进入 mysql 并重置 root 密码

   ```mysql
   update mysql.user set authentication_string=PASSWORD('newpasswd') where User='root';
   flush privileges;
   ```

4. 关闭 mysql 并在 system preferences 中重启服务

   步骤 2 中启动的进程没找到方法关闭，直接 kill 了

5. 进入 mysql 再次 set password

   ```mysql
   set password for root@'localhost' = PASSWORD('newpassword');
   ```


