# GTID相关的学习笔记

## 一次搭建主从复制环境时遇到的错误
在做一个脚本测试的时候，需要搭建一个主从环境，模拟并发写入，同时进行表重命名。
为了方便，写了一个自动初始化搭建环境的脚本，如果发现测试脚本有问题需要重新测试的时候就运行该脚本，能够很快回到刚刚搭建的时刻。

但是在搭建的过程中有个很奇怪的问题，将原始副本的数据拷贝到slave实例上启动，然后建立与master的复制后，发现数据有不一致。

> master

```
root:(none)> select count(*) from test_db.test_tb_10000000;
+----------+
| count(*) |
+----------+
|        0 |
+----------+
1 row in set (0.00 sec)
```

> slave

```
root:(none)> select count(*) from test_db.test_tb_10000000;
+----------+
| count(*) |
+----------+
|    61277 |
+----------+
1 row in set (0.04 sec)

```

MySQL在启动的时候会去搜索binlog中gtid_executed和gtid_purged，如果有老的binlog数据在上面，那么这个也会加入到gtid_executed和gtid_purged中
当master在旧的binlog下启动了，那么它就会多一些gtid，slave去请求的时候会将这些没有的gtid拉取到本地回放，这样就会发生master的数据和slave的数据不一致。

所以得出的结论是：
```
binlog也是数据的一部分，如果重新初始化数据库，binlog也要初始化为配套的binlog。
```


## 新概念
diamond topology

## 实际中的问题
https://bugs.mysql.com/bug.php?id=76959
https://github.com/mysql/mysql-server/commit/9dab9dad975d09b8f37f33bf3c522d36fdf1d0f9
https://yq.aliyun.com/articles/511065
https://www.imooc.com/article/275120
https://yq.aliyun.com/users/7c54npjb2zdg4?spm=a2c4e.11153940.blogrightarea511065.2.3a5e34eazVOkJO
