---
title: Archlinux安装MySQL/MariaDB
abbrlink: dc73344a
tags:
---

1. `yay -S mariadb mariadb-libs`

2. `mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`配置，可以对`datadir`进行自定义修改，这条指令需要在启动`mariadb.service`**之前**运行

3. 登录`sudo mysql -u root -p`初始密码为空，直接回车即可进入

4. 创建用户

   ```mysql
   # 其中name为用户名, pass为密码
   create user 'name'@'localhost' identified by 'pass';
   grant all privileges on *.* to 'samuel'@'localhost';
   flush privileges;
   quit
   ```

   