# MySQL出带宽被占满的问题定位

1. 问题现象
	每隔2~3天的中午12:00到12:10会收到该实例的流量告警
	流量突增
	qps突增
	慢查询没有变化
2. 使用cdb查询这个实例的语句执行情况，没有发现有变化特别明显的语句
3. 使用DBTraceTool_new.py来查询对比前后几天的同一时间段的语句执行情况，也没有变化
4. 使用tcpdump，抓取该时间段的执行语句，使用tshark分析语句，发现有一条语句执行总量占60%，但是没有在cdb或者DBTraceTool_new.py中查到。
5. cdb中数据追踪功能和DBTraceTool_new.py使用的是同一套数据源，所以没有结果应该是同一个问题导致的
6. 这条语句之前由于CPU告警优化过MySQL的参数，调整了时区转换函数，现在没有cpu告警，但是并发没有下来，导致的结果是执行的频度更高了，出带宽占满。