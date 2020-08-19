[返回首页](../index.md)

# 在ubuntu使用源码安装和配置PostgreSQL

postgres是一款开源数据库，源码安装会有各种问题产生，本文将简述源码安装的流程，并简述启动、配置数据库的一些细节。本次安装基于 `Ubuntu 16.04.3` 操作系统，所安装的pg版本为 `11.4`。安装过程中主要参考[官方文档](https://www.postgresql.org/docs/11/install-short.html)

### 准备工作

进行源码安装之前我们需要先安装一些依赖的应用，首先升级一下 `apt-get`，并测试一下是否已安装 `make` 编译器(要求高于3.80)

```bash
$ sudo apt-get update --fix-missing
$ make --version
```

如果尚未安装，则先安装编译器

```bash
$ sudo apt-get install make
```

安装 `gcc`、 `GNU Readline` 和 `zlib`

```bash
$ sudo apt-get install gcc
$ sudo apt-get install libreadline6 libreadline6-dev
$ sudo apt-get install zlib1g-dev
```

由于我们需要在pg中使用 `PL/Python`，所以还需要安装 `python3-dev`

```bash
$ sudo apt-get install python3-dev
```

最后，下载并解压源码，源码地址可以在[官方网址](https://www.postgresql.org/ftp/source/)找到

```bash
$ wget https://ftp.postgresql.org/pub/source/v11.4/postgresql-11.4.tar.bz2
$ gunzip postgresql-11.4.tar.gz
$ tar xf postgresql-11.4.tar
```

### 编译安装

编译和安装的过程文档介绍的较为详细，只需要遵照执行。唯一需要注意的是需要在configure时加上 `--with-python` 来支持 `PL/Python` 扩展

```bash
$ cd postgresql-11.4/

# Configuration
$ ./configure --with-python

# Build
$ make world

# Installing the Files
$ sudo make install-world

# Cleaning
$ make clean
```

### 添加用户并设置启动路径

一般我们会选择新建一个单独的user来运行PostgreSQL，user一般设为 **postgres**。我们后续将登录该账号进行数据库配置

```bash
$ sudo adduser postgres
$ sudo chown postgres /usr/local/pgsql
$ su - postgres
```

把安装好的目录加到 `PATH` 里，我们需要修改 `.profile` 或 `.bash_profile`文件，增加如下两行

```bash
PATH=/usr/local/pgsql/bin:$PATH
export PATH
```

随后运行如下命令使其生效

```bash
$ . ./.profile
```

### 启动数据库

一般我们将使用pg_ctl来启停数据库，在第一次启动数据库之前，我们要先确定启动数据库的目录，并指定postgres用户为其owner

```bash
$ initdb -D /usr/local/pgsql/data
$ pg_ctl -D /usr/local/pgsql/data -l /usr/local/pgsql/data/logfile start
```

### 配置连接和认证

至此我们以及可以在本地正常使用数据库了。但我们还需要继续配置数据库服务器，以便我们从客户端访问数据库。
其中一个简单的方法是修改 `postgresql.conf` 和 `pg_hba.conf` 文件

- postgresql.conf

```
listen_addresses = '*' 
```

- pg_hba.conf

```
# IPv4 host connections:
host    all             all             0.0.0.0/0            md5
# IPv6 host connections:
host    all             all             ::/0             md5
```

将以上内容分别填写在两个配置文件后，重启数据库服务

```bash
$ pg_ctl -D /usr/local/pgsql/data -l /usr/local/pgsql/data/logfile restart
```

利用psql创建数据库，并新建user

```bash
$ psql
postgres=# create database test;
postgres=# \c test;
postgres=# create user dbuser with encrypted password '123456';
```

至此，我们已经可以在客户端通过账户dbuser正常访问test数据库了。

### 设置开机自动启动

为保证数据库正常可用，我们需要配置开机自动启动，避免因为服务器重启引起的数据库故障。在上述环境下，我们需要修改 `/etc/rc.local` 文件，加人如下内容：

```
su postgres -c '/usr/local/pgsql/bin/pg_ctl start -l /usr/local/pgsql/data/logfile -D /usr/local/pgsql/data'
```

### 总结

在服务器搭建从源码安装的postgresql数据库涉及诸多细节，本文只是简述一个较简易的搭建场景，更多细节还应根据需要参照官方文档进行尝试，出现报错可以在google上进行查询，希望可以有所帮助。

 
---------------------------------------------------------------
2019-11-01
