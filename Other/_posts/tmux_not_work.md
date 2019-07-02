# tmux突然假死的问题

上周公司上了一个新的堡垒机系统，统一了登录密码，现在连接线上服务器不用再输入多次不一样的密码，使用更方便了。
但是有个很恼火的问题是tmux突然出问题了，前一天晚上连接很正常，第二天再连接就没有任何响应了，无论怎么敲键盘都没办法响应。
尝试使用tmux ls查看tmux的会话情况，居然也卡住了。无奈只能再另开一个窗口，把所有的tmux进程kill掉。

发生问题时的现象很奇怪，tmux只有我一个人在使用，但是在ps查看进程时却有多个：
```
UID        PID  PPID  C STIME TTY          TIME CMD
root     17881 11491  0 17:06 pts/159  00:00:00 tmux attach -t 0
root     23354 23250  0 17:10 pts/208  00:00:00 tmux attach -t 0
root     23515 23409  0 17:11 pts/209  00:00:00 tmux attach -t 0
```
连接的还是同一个tmux会话。

然后尝试使用strace来定位问题，strace tmux ls，发现是阻塞在epoll_wait函数上:
```

```

由于对tmux的内部运行原理不清楚，strace信息也没有什么头绪。

然后从上面的ps -ef的输出中想到，这些tmux attach的进程是怎么来的？
通过使用父进程id，查到他们都还在sshd进程下运行着：
```
UID        PID  PPID  C STIME TTY          TIME CMD
root     11464 13299  0 17:00 ?        00:00:00 sshd: root@pts/159
root     11491 11464  0 17:00 pts/159  00:00:00 -bash
root     17881 11491  0 17:06 pts/159  00:00:00 tmux attach -t 0
```

那这些sshd进程是哪儿来的呢？突然想到每次关闭线上服务器的连接都是直接关闭客户端做的，每次再连接是通过自动登录脚本来做的。
问题到这儿就差不多清楚了，每次关闭客户端都没有关闭ssh连接，导致很多tmux attach -t 0在运行，也就是多个tmux同时连接同一个会话，这些会话需要使用缓冲区，如果其他窗口异常关闭了，但是连接还在，那么缓冲区满了之后也不会释放。

解决方式：
1. 把tmux attach -t 0所在的sshd进程kill掉，对应的bash进程以及bash进程下的tmux attach -t 0就终止了
2. 或者在每次关闭客户端连接的时候先detach tmux会话，然后使用exit的方式正常退出bash，就不会有多个tmux attach -t 0的进程了。
3. 或者提bug给堡垒机维护方，让他们在客户端异常关闭连接时也能关闭ssh连接
