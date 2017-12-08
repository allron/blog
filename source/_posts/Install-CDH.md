---
title: Install CDH
date: 2017-12-08 14:45:50
tags: CDH
---

# CDH 大数据平台搭建

### *[INSTALL DOC](https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-installation/content/ch_Getting_Ready.html)*

# Software

1. [MySQL](https://dev.mysql.com/downloads/mysql/)  mysql-5.6.30-linux-glibc2.5-x86_64.tar.gz

2. [nginx](http://nginx.org/en/download.html) http://nginx.org/download/nginx-1.12.2.tar.gz

   ​

# Setting Base Environment(All Nodes)

1. turn off iptable：

  ```sh
  service iptables stop

  chkconfig iptables off

  systemctl disable firewalld

  service firewalld stop
  ```



2. 关闭SElinux：

> setenforce 0
>
> sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

3. install JDK

   > tar zxvfmysql-5.6.30-linux-glibc2.5-x86_64.tar.gz -C /usr/local/_
   >
   > _ln -sf mysql-5.6.30-linux-glibc2.5-x86_64 mysql

4. create  user & group (only at test_bd_3rddata )

   > groupadd mysql
   > useradd -r -g mysql mysql -M -s /sbin/nologin
   >
   > groupadd nginx
   > useradd -r -g nginx nginx -M -s /sbin/nologin

5. ​

# Install Mysql

1. 解压、软连接

   ```shell
   tar vxfz mysql-5.6.38-linux-glibc2.12-x86_64.tar.gz -C /usr/local
   cd /usr/local 
   ln -sf mysql-5.6.38-linux-glibc2.12-x86_64 mysql
   ```

2. 安装依赖库

   ```sh
   yum install -y perl-Module-Install.noarch libaio
   ```

3. 创建mysql用户和组

   ```shell
   groupadd mysql
   useradd -r -g mysql mysql -M -s /sbin/nologin
   ```

4. 修改配置my.cnf

   > [client]
   > port    = 3306
   > socket  = /data/mysql/mysql.sock
   >
   > [mysqld]
   > user    = mysql
   > port    = 3306
   > socket  = /data/mysql/mysql.sock
   > bind-address = 192.168.0.1
   > basedir = /usr/local/mysql
   > datadir = /data/mysql
   > character-set-server = utf8
   > skip_name_resolve = 1
   > open_files_limit = 65535
   > back_log = 1024
   > max_connections = 512
   > max_connect_errors = 1000000
   > slow_query_log = 1
   > slow_query_log_file = /data/mysql/slow.log
   > log-error = /data/mysql/error.log
   > long_query_time = 1
   > server-id = 1
   > log-bin = mysql-bin
   > sync_binlog = 0
   > binlog_cache_size = 4M
   > max_binlog_cache_size = 2G
   > max_binlog_size = 1G
   > expire_logs_days = 7
   > binlog_format = mixed
   > innodb_buffer_pool_size = 6G
   > sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
   >
   > [mysqldump]
   > quick
   > max_allowed_packet = 32M
   >
   > [mysqld_safe]
   > socket  = /data/mysql/mysql.sock
   > log-error=/data/mysql/mysql.log
   > pid-file=/data/mysql/mysql.pid

5. 初始化数据库

   ```shell
   /usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql
   ```

6. 修改目录权限

   ```shell
   chown mysql:mysql -R /usr/local/mysql*
   ```

7. 配置启动脚本

   ```she
   cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
   vim /etc/init.d/mysqld
   chmod +x /etc/init.d/mysqld
   chkconfig --add mysqld
   ```

   修改内容:

   > basedir=/usr/local/mysql
   > datadir=/data/mysql

8. 修改系统环境变量 vim /etc/profile

   > export PATH=/usr/local/mysql/bin:$PATH

   ```shell
   source /etc/profile
   ```

   启动与停止mysql命令

   ```shell
   service mysqld start
   service mysqld stop
   ```

9. 设置root密码、创建数据库、授权访问

   ```shell
   # 连接mysql 
    mysql -S /data/mysql/mysql.sock  -uroot -p
   # 设置密码
    SET PASSWORD=PASSWORD('121212');
    flush privileges;
   # create ambari
   create database ambari DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
   grant all privileges on ambari.* to 'ambari'@'%' identified by '2121212' with grant option;
   ```

   ​

   # Create a Local Repository

   `Centos7` `Ambari2.5.0` `HDP-2.6.0.3`

   > http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.0.3/ambari.repo
   >
   > http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.0.3/hdp.repo
   >
   > #local Repository 
   >
   > http://n1/hdp/centos7/HDP-2.6.0.3/
   >
   > http://n1/ambari/centos7/ambari-2.5.0.3

   1. Put repository configuration files for Ambari and the Stack in place on the host

      ```shell
      wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.0.3/ambari.repo -O /etc/yum.repos.d/ambari.repo 
      wget http://public-repo-1.hortonworks.com/HDP/centos7/2.x/updates/2.6.0.3/hdp.repo -O hdp.repo
      yum repolist
      ```

   2. Synchronize the repository contents to your mirror server

      ```shell
      mkdir -p /data/nginx/www/ambari/centos7
      cd /data/nginx/www/ambari/centos7
      reposync -r ambari-2.5.0.3
      mkdir -p /data/nginx/www/hdp/centos7
      cd /data/nginx/www/hdp/centos7
      reposync -r HDP-2.6.0.3
      ```

   3. Generate the repository metadata.

      ```shell
      createrepo /data/nginx/www/ambari/centos7/ambari-2.5.0.3
      createrepo /data/nginx/www/hdp/centos7/HDP-2.6.0.3
      ```

      ​

   4. Edit the `ambari.repo` file and replace the Ambari Base URL `baseurl` obtained when setting up your local repository

      ```shell
      # vim /etc/yum.repos.d/ambari.repo
      ```

      > [ambari-2.5.0.3]
      > name=ambari Version - ambari-2.5.0.3
      > baseurl=http://n1/ambari/centos7/ambari-2.5.0.3
      > gpgcheck=0
      > gpgkey=http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.5.0.3/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
      > enabled=1
      > priority=1

   5. Edit the `/etc/yum/pluginconf.d/priorities.conf` file to add the following:

      > ```
      > [main]
      > enabled=1
      > gpgcheck=0
      > ```

   6. install tool

      ``` shell
      yum install -y createrepo yum-plugin-priorities

      ```
   7. install mbari-server ,installs the default PostgreSQL Ambari database postgresql
   ```
   yum install ambari-server
   ambari-server setup
   ```
   
