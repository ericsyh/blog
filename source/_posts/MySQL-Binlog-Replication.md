---
title: MySQL Binlog Replication
date: 2020-01-05 21:32:41
tags: 
	- MySQL 
	- Binlog
---

MySQL 开启 binlog 配置主从同步步骤整理。


### 示例服务器

Azure VM 上创建两台 centos 7.5 服务器分别作为 MySQL master 和 My SQL slave：

* master: 13.75.108.150
* slave: 13.75.109.41

### 安装 MySQL 

#### 添加 MySQL 5.7仓库

```
sudo rpm -ivh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

#### 安装 MySQL 5.7

```
sudo yum -y install mysql-community-server
```

#### MySQL 服务配置

##### 启动 MySQL 服务

```
sudo systemctl start mysqld
```

##### 设置开启自启

```
sudo systemctl enable mysqld
```

#### 安全设置

##### 获取 MySQL 临时密码

```
cat /var/log/mysqld.log | grep -i 'temporary password'
```

##### 配置安全设置

```
mysql_secure_installation
```

### 配置 binlog 主从同步

#### master 服务器配置

##### 创建同步用户

```
mysql> CREATE USER 'slave'@'%' IDENTIFIED BY '{YOUR PASSWORD}';

mysql> GRANT ALL PRIVILEGES ON *.* TO slave@"%" IDENTIFIED BY "{YOUR PASSWORD}";
```

##### 开启 binlog

修改 `my.cnf` 文件，添加内容如下：

```
log-bin=mysql-bin
server-id=10
```

重启 MySQL 服务：

```
systemctl restart mysqld.service
```

##### 获取初始同步位置

```
mysql> show master status;
```

#### slave 服务器配置

##### 修改 MySQL 配置

修改 `my.cnf` 文件，添加内容如下：

```
server-id=12
replicate-do-db=test
skip-slave-start=true
```

重启 MySQL 服务：

```
systemctl restart mysqld.service
```

##### 配置主从同步

停止 slave 服务进程：

```
mysql> stop slave;
```

配置同步上游信息：

```
mysql> change master to 
master_host='13.75.108.150',
master_user='slave',
master_password='{YOUR PASSWORD}',
master_log_file='mysql-bin.000002', 
master_log_pos=1472;
```

开启 slave 服务进程：

```
mysql> start slave;
```

查看 slave 同步状态：

```
mysql> show slave status\G;
```