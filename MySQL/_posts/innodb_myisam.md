## INNODB和MyISAM的区别

1. 对于AUTO_INCREMENT类型的字段，该表必须要有包含该字段为第一顺位的主键或者唯一索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。
现有的BI表中有相应的表结构有的表没有主键
2. InnoDB不支持FULLTEXT类型的索引
3. InnoDB 中不保存表的具体行数，使用的都是实时遍历查出来的值，MyISAM有统计值直接返回
4. DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。
5. LOAD TABLE FROM MASTER操作对InnoDB是不起作用的

https://mariadb.com/kb/en/library/converting-tables-from-myisam-to-innodb/
https://dev.mysql.com/doc/refman/5.7/en/converting-tables-to-innodb.html
https://mysqlserverteam.com/upgrading-directly-from-mysql-5-0-to-5-7-using-an-in-place-upgrade/


从MySQL5.1的MyISAM升级到MySQL5.7的InnoDB：
1. 关闭原实例old_mysql，将mysql_data目录拷贝到目的机器dest_machine
2. 在dest_machine上配置my.cnf文件，加入skip_grant_tables
3. 以5.7版本启动新实例
4. 以5.7版本升级新实例
5. 比较升级前后数据是否一致
6. 开启新实例复制


