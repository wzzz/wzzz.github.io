# sentinel monitor命令
>sentinel monitor name ip port quorum

使用sentinel monitor加入监控组时，需要注意：
name不能与当前sentinel实例已经监控的其他master重名，否则不能添加
如果对同一个redis实例集群的sentinel监控组使用不同的name监控，那么就会被sentinel当做新的监控组加入。

## 移除sentinel的时候等待30s是怎么计算的？
