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

* master: 
* slave:

### 安装 MySQL 

#### 添加Mysql5.7仓库

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