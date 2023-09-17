---
title: Postgresql编译调试安装
date: 2023-08-28 20:34:32
tags:
- Database
categories: 
- Database
index_img: /imgs/Postgresql.png
---

# PostgreSQL编译调试安装

## Pg编译安装

### 前提

- 安装好的WSL2
- Ubuntu 22.04 LTS版本

```shell
# Ubuntu环境下
sudo apt-get install build-essential 
sudo apt install libreadline-dev 
sudo apt-get install zlib1g.dev
```

### 下载Pg的源码

>  [Pg源码地址](https://www.postgresql.org/ftp/source/)

选择所需要的源码进行下载，下载完成后进行解压,例如：

```shell
wget https://ftp.postgresql.org/pub/source/v15.4/postgresql-15.4.tar.gz
tar -zxvf postgresql-15.4.tar.gz
cd postgresql-15.4
mv postgresql-15.4 postgresql15
```

### 进行编译

```shell
./configure --enable-debug
make -j
sudo make install
```

由于Pg不能在root用户下进行启动,这里添加一个postgres用户并设置其密码（注意：以下步骤不用做直接跳转到`VSCode编译调试Pg`）

```shell
adduser postgres
passwd postgres
```

### 进行数据库配置

```shell
# 创建pg的数据存放目录
sudo mkdir -p /var/postgresql/data
# 对/var/postgresql和/usr/local/pgsql进行授权
sudo chown postgres:postgres /var/postgresql -R
sudo chown postgres:postgres /usr/local/pgsql -R
```

### 切换用户并配置环境变量

```shell
su - postgres
export PATH=/usr/local/pgsql/bin:$PATH
```

### 初始化数据库并启动服务

```shell
# 使用-D指定初始化数据库存放的数据目录
initdb -D /var/postgresql/data
# 使用-l指定pg日志存放的目录
pg_ctl -D /var/postgresql/data -l /var/postgresql/logfile start
```

### 连接Pg

```shell
psql 
```

## VSCode编译调试Pg

### 前提

配置好pg的环境变量可以编辑`/etc/profile`在最后一行添加环境变量，如：`export PATH=$PATH:/usr/local/pgsql/bin`

安装好VSCode的插件拓展，包括`C/C++、C/C++ Extension Pack、CMake、CMakeTools`；

```shell
# 初始化目录为具有super权限的用户的目录下，这里我选择在Postgresql15目录下创建data目录
initdb -D /home/bluehole/postgresql15/data
# 启动服务
pg_ctl -D /home/bluehole/postgresql15/data -l logfile start
# 使用createdb创建数据库
createdb <dbname>
# 启动服务后登录数据库
psql -U <username> <dbname>
# 创建表以及插入数据
create table t1(id integer, name text);
insert into t1 values(1, 'pgsql1');
insert into t1 values(2, 'pgsql2');
# 使用这条语句活得pg的进程id
select pg_backend_pid();
```

### launch.json配置文件

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "postgres --help",
            "type": "cppdbg",
            "request": "launch",
            "program": "/usr/local/pgsql/bin/postgres",
            "args": [
                "--help"
            ],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        {
            "name": "initdb",
            "type": "cppdbg",
            "request": "launch",
            "program": "/usr/local/pgsql/bin/initdb",
            "args": [
                "-D",
                "/home/bluehole/postgresql15/data"
            ],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        },
        {
            "name": "postgres backend",
            "type": "cppdbg",
            "request": "attach",
            "program": "/usr/local/pgsql/bin/postgres",
            "processId": "${command:pickProcess}",
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

如图所示则表示调试成功：

![Pg查询调试图](/imgs/Git/Pg查询调试.png)

> Tips：调试需要启动两个终端，一个已经登陆的客户端，另一个是点击Debug所自动启动的终端
