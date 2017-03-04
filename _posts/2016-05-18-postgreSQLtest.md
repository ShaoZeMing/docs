---
layout: blog
title: postgreSQL储存地理位置600万数据性能测试笔记
category: blogs
---

对于地理位置储存，在现在的项目中用的越来越多，支持的数据库也不少，像mangodb,redis-geo,postgreSQL.....
本文对postgreSQL point 类型对地理位置数据插入，索引，附近查询，进行测试；由于电脑配置不高，仅仅插入600多万条数据进行测试，但测试结果为何还不如那位大神上100亿测试的数据快。（下面解释为）

# 硬件预览
- 虚拟机系统：Ubuntu 16.1
- 内存：4G
- postgreSQL 9.5 （如果未安装postgreSQL，可以访问[postgreSQL 基础入门操作](http://blog.4d4k.com/index.php/archives/45/)安装）
- 操作界面：postgreSQL控制台。

# 开始测试

## 创建数据库

```
create table tbl_point(id serial8, poi point);
```

## 修改添加一个序列生成器

```
alter sequence tbl_point_id_seq cache 10000;
```

## 插入随机数据10000条

```
insert into tbl_point(poi) select point(trunc(100000*(0.5-random())), trunc(100000*(0.5-random()))) from generate_series(1,10000);

```

![ps001.jpg][1]

## 插入随机数据100000条

```
insert into tbl_point(poi) select point(trunc(100000*(0.5-random())), trunc(100000*(0.5-random()))) from generate_series(1,100000);

```

![ps002.jpg][2]

## 插入随机数据1000000条

```
insert into tbl_point(poi) select point(trunc(100000*(0.5-random())), trunc(100000*(0.5-random()))) from generate_series(1,1000000);

```

![ps003.jpg][3]

## 插入随机数据4000000条

```
insert into tbl_point(poi) select point(trunc(100000*(0.5-random())), trunc(100000*(0.5-random()))) from generate_series(1,4000000);

```

![ps004.jpg][4]

## 插入数据大小

```
\dt+ tbl_point
```

![ps005.jpg][5]


## 给point类型字段创建GiST索引

#### 注：创建索引比插入慢很多，创建索引成功后，插入相同数据量，速度也将变慢很多。

```
create index idx_tbl_point on tbl_point using gist(poi) with (buffering=on);  //创建索引

\di+        //查看索引信息
```

![ps006.jpg][6]


## 查询

### 查询一：

```
select *,poi <-> point(1000,1000) dist from tbl_point where poi <-> point(1000,1000) < 100 order by poi <-> point(1000,1000) limit 10;

```

![ps007.jpg][7]


### 查询二：
就是换个位置数据继续查10条。

```
select *,poi <-> point(4000,7000) dist from tbl_point where poi <-> point(4000,7000) < 100 order by poi <-> point(4000,7000) limit 10;
```

![PS008.jpg][8]

### 查询三：
刚才两次查询效率都不错，都是1毫秒多一点，来看看查50条记录效果。

```
select *,poi <-> point(4000,7000) dist from tbl_point where poi <-> point(4000,7000) < 100 order by poi <-> point(4000,7000) limit 50;
```

![ps009.jpg][9]


### 查询四：
查50条记录效果几乎到了不理想的状态，我们再把limit去掉，直接查询全部看看。

```
select *,poi <-> point(4000,7000) dist from tbl_point where poi <-> point(4000,7000) < 100 order by poi <-> point(4000,7000);
```

![ps010.jpg][10]

发现居然快了很多很多

# 总结

回答上面我们的疑问：当数据足够时，我们只查询10条时，并没有扫描全盘，找到10条就返回，所有都是1ms 左右，那速度是相当的快。
所以，别人600亿的数据找10条速度理想就可以理解了。

而当需要返回50条数据时，满足要求的并没有那么多，必然全盘扫描，并且多次核对。造成查询时间达到10条查询时间的36000多倍。。。。。。。。实乃恐惧。。。

最后奇怪的事情发生了，我执行全部查询，不添加 limit 50, 速度又快了很多很多。这个问题我也不是很懂，请各位赐教。

原创作者：邵泽明
原文链接：<http://blog.4d4k.com/index.php/archives/46/>


  [1]: http://blog.4d4k.com/usr/uploads/2017/03/2184140047.jpg
  [2]: http://blog.4d4k.com/usr/uploads/2017/03/194132396.jpg
  [3]: http://blog.4d4k.com/usr/uploads/2017/03/2571149806.jpg
  [4]: http://blog.4d4k.com/usr/uploads/2017/03/294715778.jpg
  [5]: http://blog.4d4k.com/usr/uploads/2017/03/156233624.jpg
  [6]: http://blog.4d4k.com/usr/uploads/2017/03/3783685977.jpg
  [7]: http://blog.4d4k.com/usr/uploads/2017/03/1930506550.jpg
  [8]: http://blog.4d4k.com/usr/uploads/2017/03/3570827542.jpg
  [9]: http://blog.4d4k.com/usr/uploads/2017/03/3450442186.jpg
  [10]: http://blog.4d4k.com/usr/uploads/2017/03/867267303.jpg