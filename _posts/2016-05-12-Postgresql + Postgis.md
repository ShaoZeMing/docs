---
layout: blog
title: ubuntu12安装Postgresql + Postgis
category: blogs
---
如果系统是ubuntu12，系统默认自带Postgresql9.1，首先我们把它删除了！来演示ubuntu12安装Postgresql9.4 + Postgis

# 步骤：

### 列出关于postgre9.1的包

```
dpkg -l | grep postgres
```

如果是9.1，下面列表的8.3就是9.1，有一些包可能没有，是系统而定。

```
ii  postgresql                            8.3.17-0ubuntu0.8.04.1           object-relational SQL database (latest versi
ii  postgresql-8.3                        8.3.9-0ubuntu8.04                object-relational SQL database, version 8.3
ii  postgresql-client                     8.3.9-0ubuntu8.04                front-end programs for PostgreSQL (latest ve
ii  postgresql-client-8.3                 8.3.9-0ubuntu8.04                front-end programs for PostgreSQL 8.3
ii  postgresql-client-common              87ubuntu2                        manager for multiple PostgreSQL client versi
ii  postgresql-common                     87ubuntu2                        PostgreSQL database-cluster manager
ii  postgresql-contrib                    8.3.9-0ubuntu8.04                additional facilities for PostgreSQL (latest
ii  postgresql-contrib-8.3                8.3.9-0ubuntu8.04                additional facilities for PostgreSQL
```

### 执行以下命令删除

```
sudo apt-get --purge remove postgresql postgresql-8.3  postgresql-client  postgresql-client-8.3 postgresql-client-common postgresql-common  postgresql-contrib postgresql-contrib-8.3
```

### 如果彻底删除，还要删除以下目录，可选。

```
sudo rm -rf /var/lib/postgresql/
sudo rm -rf /var/log/postgresql/
sudo rm -rf /etc/postgresql/
```

## 开始安装Postgresql 9.4。

1. 首先更新apt-get的关于Postgresql的repository。在Postgresql官方网站下载页，有一个下拉列表，里面选择你的ubuntu版本，然后就会显示出你需要的连接和方法，我这里选择ubuntu12。
2. 把`deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main`添加到`/etc/apt/sources.list.d/pgdg.list`文件里。
3. 在终端执行以下命令：
    ```
    ii wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | \
    ii sudo apt-key add -
    ii sudo apt-get update
    ```
4. 执行`sudo apt-get install postgresql-9.4`安装`postgresql`。
5. 安装完后，执行`sudo -u postgres psql template1`，以管理员身份进入默认数据库。
6. 执行`ALTER USER postgres with encrypted password 'xxxxxxx';`为管理员设置密码。
7. 退出数据库，在终端执行sudo vim /etc/postgresql/9.4/main/pg_hba.conf修改配置文件。把关于postgres管理员这行的peer改为md5。

    ```
    local    all    postgres    peer md5
    ```
8. 重启数据库。`sudo /etc/init.d/postgresql restart`

## 支持用系统管理员登录数据库

1. 首先创建一个与系统管理员同名的数据库用户（不知道系统管理员名字？执行`whoami`！）
  ```
  createuser -U postgres -d -e -E -l -P -r -s <my_name>
  ```
2. 修改配置文件pg_hba.conf（同上配置文件），修改如下：
  ```
  local    all    all    peer md5
  ```
3. 重启数据库。
  ```
  sudo /etc/init.d/postgresql restart
  ```

## 为数据库安装postgis扩展。

1. 用`apt-cache search postgresql postgis`查找最新的版本
2. 我们这里找到的是`postgresql-9.4-postgis-2.1`，注意这个包只是postgis的包并不包含postgresql 它的含义是适合postgresql 9.4的postgis 2.1版本，执行如下命令：
  ```
  apt-get install postgresql-9.4-postgis-2.1
  ```
3. 用数据库管理员身份连接postgresql与postgis(赋予postgresql空间数据库的能力)

  ```
  系统管理员=# `CREATE EXTENSION postgis;`
  CREATE EXTENSION
  系统管理员=# `CREATE EXTENSION postgis_topology;`（支持拓扑）
  CREATE EXTENSION
  ```
4. 测试一下版本信息
  ```
  系统管理员=# SELECT version();//显示postgresql的版本
  系统管理员=# SELECT postgis_full_version();//显示postgis的版本
  ```

## 以下是在rails中使用Postgresql+Postgis

1. 使用gem activerecord-postgis-adapter，执行bundle install。
2. 新创建数据库 RAILS_ENV=production bundle exec rake db:create
已存在数据库RAILS_ENV=production bundle exec rake db:gis:setup