---
layout:     post
title:      "MySQL 8.0 Access Denied for root"
subtitle:   "Steps to access MySQL 8.0 as root from localhost or remote client"
date:       2018-01-06 12:00:00
author:     "Shengwen"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - MySQL
    - Linux
---

## Connect to MySQL from localhost

After installation, you may find yourself unable to connect to MySQL: "Access denied for user 'root'@'localhost'...". Here's the solution that works for me (MySQL 8.0 on Centos 7).

1. Modify my.cnf to allow connection without password.

   `vim /etc/my.cnf`

   Add `skip-grant-table` at the end of the file.

2. Restart MySQL service.

   ```
   systemctl stop mysqld
   systemctl start mysqld
   ```

   or `systemctl restart mysqld`

3. Connect to MySQL without password.

   `mysql -u root`

4. Change password for root.

   ```
   mysql> flush privileges;
   mysql> alter user 'root'@'localhost' identified by 'YourNewPassword';
   mysql> exit;
   ```

5. Open my.cnf again and delete the line added in step 1.

6. Restart MySQL service again. Same as step 2.

7. Now you can connect to MySQL as root using the new password.

   `mysql -u root -p`

## Connect to MySQL remotely

 If you cannot connect to MySQL as root using the password, try these steps:

1. On the server where MySQL installed, connect to MySQL

   `mysql -uroot -p`

2. Update user table.

   ```
   mysql> use mysql;
   mysql> update user set host='%' where user='root';
   mysql> exit;
   ```

3. Restart MySQL service. You know how to do it.

4. Connect to MySQL remotely using Navicat or Workbench or whatever you like.

## Attention

If you stuck in any step above and see this: 

"The MySQL server is running with the  --skip-grant-tables option so it cannot execute this statement"

Do these :

```
mysql> set global read_only=0;
mysql> flush privileges;
```

Then try that step again. Then:

```
mysql> set global read_only=1;
mysql> flush privileges;
```

