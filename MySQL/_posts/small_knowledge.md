### MASTER_POS_WAIT函数
select master_pos_wait('$file',$pos,10);
如果slave SQL thread没有启动，那么执行结果就是NULL
10s表示超时等待时间，如果在10s内可以追上'$file',$pos指定的日志位置，那么返回0
如果追不少，则返回-1
>https://dev.mysql.com/doc/refman/5.6/en/miscellaneous-functions.html#function_master-pos-wait
