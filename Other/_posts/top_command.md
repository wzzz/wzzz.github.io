top命令的输出项涵义

> top - 15:57:10 up 816 days, 17:44,  3 users,  load average: 1.16, 1.10, 1.13

```
load average表示进程占用CPU的繁忙程度
1.16表示前1分钟平均有0.16个进程等待CPU
1.10表示前5分钟平均有0.1个进程等待CPU
1.13表示前15分钟平均有0.13个进程等待CPU
如果这个数值小于1，表示CPU是有空闲时间的
load average给我们的参考作用不只是提供了CPU的繁忙程度，也提供了进程的等待队列情况，相对于CPU percentage，load average更能显示出当前的系统繁忙程度。
https://unix.stackexchange.com/a/303703/152514
https://www.linuxjournal.com/article/9001
https://www.howtogeek.com/194642/understanding-the-load-average-on-linux-and-other-unix-like-systems/
```

>Tasks: 1467 total,   1 running, 1466 sleeping,   0 stopped,   0 zombie

```
当前的进程总数为1467，正在运行中的进程数为1，正在等待中的进程数为1466，暂停状态的进程数为0，僵尸状态的进程数为0
```

>Cpu(s):  1.8%us,  2.9%sy,  0.0%ni, 95.2%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st

```
用户态的CPU使用率为1.8%, 系统态的CPU使用率为2.9%，low priority user mode为
```

>Mem:  132043924k total, 55606304k used, 76437620k free,   446032k buffers
>Swap: 33554428k total,        0k used, 33554428k free, 37364472k cached

1   PID   PPID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  SWAP CODE DATA WCHAN     COMMAND                                                           
  65324  60951 root      20   0 4699m 4.4g 2560 S  0.0  3.5   1:08.83    0 1500 4.4g pipe_wait /root/.pyenv/versions/2.7.8/bin/python2.7 -m celery worker --loglev
  65342  60951 root      20   0 3424m 3.1g 2592 S  0.0  2.5   0:55.76    0 1500 3.1g pipe_wait /root/.pyenv/versions/2.7.8/bin/python2.7 -m celery worker --loglev


https://www.linux.com/learn/uncover-meaning-tops-statistics