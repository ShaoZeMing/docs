---
layout: blog
title: PostGIS安装和创建模板template_postgis
category: blogs
---
PostGIS添加了对PostgreSQL数据库中地理对象的支持。本文档描述了安装PostGIS和创建模板PostGIS数据库的过程。假设PostgreSQL已经安装。如果没有，请参考PostgreSQL页面。



# 内容 [ 隐藏 ]
1. 安装PostGIS
2. 安装PostGIS扩展
3. 创建模板PostGIS数据库
4. 从模板创建PostGIS数据库
5. 更多资源
6. PostGIS使用json_tokener_error失败


----------


# 安装PostGIS

- 安装了PostGIS的包。
- 安装PostGIS扩展



因为[[PostgreSQL 9.1] [1] ]，首选的方法是安装PostGIS并为每个空间数据库启用postgis扩展。

```
$ psql

- 验证可用的扩展
SELECT名称，default_version，installed_version
FROM pg_available_extensions WHERE name LIKE'postgis％';

```
- 安装空间数据库mygisdb的扩展
```
\ c mygisdb
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
CREATE EXTENSION fuzzystrmatch;
CREATE EXTENSION postgis_tiger_geocoder;
```


**如果使用PostGIS扩展，则【不需要执行以下`创建模板PostGIS数据库`】步骤**。

---------
# 升级postgis扩展

```
$ psql

ALTER EXTENSION postgis UPDATE TO“2.1.0”;
```

迁移用postgis_template创建的空间数据库
转储和删除空间数据库，重新创建具有扩展名的空间数据库，并还原转储的数据库。按照http://www.postgis.net/docs/postgis_installation.html#hard_upgrade了解特定命令。

---------
# 创建模板PostGIS数据库

成为postgres用户。
```
＃su  -  postgres
```
如果你没有创建一个超级用户访问PostgreSQL，你可能想现在做。系统将提示您授予该用户的权限。
```
$ createuser [username]
```
创建一个名为“template_postgis”的新数据库。

```
$ createdb -O [username] template_postgis -E UTF-8
```
PostGIS需要在数据库上安装pl / pgSQL语言。
```
$ createlang plpgsql template_postgis
```

为PostgreSQL和空间参考系统加载PostGIS空间类型。“postgis.sql”和“spatial_ref_sys.sql”是PostGIS安装的一部分，并且可能驻留在“/usr/sharepostgresql/contrib/postgis-2.1/”之外的其他位置，具体取决于安装。（下面是默认postgis 2.1安装）

```
$ psql -d template_postgis -f /usr/share/postgresql/contrib/postgis-2.1/postgis.sql
$ psql -d template_postgis -f /usr/share/postgresql/contrib/postgis-2.1/spatial_ref_sys.sql
```

使它成为一个真正的模板。
```
$ psql

UPDATE pg_database SET datistemplate = TRUE WHERE datname ='template_postgis';
```

从模板创建PostGIS数据库
通常的做法是保留一个裸模板来创建新的PostGIS数据库。作为PostgreSQL超级用户，以下命令将创建一个新的数据库：

```
$ createdb -T template_postgis [new_postgis_db]
```

---------
# 更多资源

有关PostGIS的其他资源，请[查看PostGIS文档][1]。


# PostGIS使用json_tokener_error失败

这在添加postgis作为扩展时出现。libjson-c包已经改变，PostGIS还没有推出一个稳定版本。它在2.1.0rc1，但。错误报告是
<http://trac.osgeo.org/postgis/ticket/2213>
修复是下载postgis PKGBUILD，然后将版本更改为“2.1.0rc1”。不要忘了更改sha256sum。


  [1]: http://postgis.refractions.net/documentation/