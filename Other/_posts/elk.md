ELK是Elasticsearch, Logstash, Kibana的简称，它实际上包含了Elasticsearch，Logstash，Kibana，Beat四个部分。
Elasticsearch是一个开源的实时全文搜索引擎，它是一个分布式系统，支持数据的分布式存储和访问，保证了数据的可靠性。
Logstash是一个收集数据和分析日志的工具，将没有结构的日志数据进行结构化分析，并输出给下游。
Kibana是一个数据可视化的web系统，是用户查看ELK系统的界面。
Beat是数据采集器，将数据采集输送给Logstash，辅助Logstash收集日志数据，它是一个统称，包括Filebeat，Packetbeat,Topbeat, Winlogbeat,Metricbeat,Auditbeat,Heartbeat,Functionbeat这些子组件。

参考资料
https://en.wikipedia.org/wiki/Elasticsearch
https://www.elastic.co/guide/en/logstash/current/introduction.html
https://www.elastic.co/guide/en/elasticsearch/reference/6.0/getting-started.html

使用elk是否有价值？ https://coralogix.com/log-analytics-blog/how-much-does-free-elk-stack-cost-you/
