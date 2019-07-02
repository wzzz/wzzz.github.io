# A quick tutorial
# 快速教程

In the next sections of this document, all the details about Sentinel API, configuration and semantics will be covered incrementally. However for people that want to play with the system ASAP, this section is a tutorial that shows how to configure and interact with 3 Sentinel instances.
在本文档的下一个小节，关于sentinel API的细节，配置和语法会被逐个介绍到。对于想立即上手使用的用户，这一节会介绍到使用三个sentinel节点来进行交互和配置。

Here we assume that the instances are executed at port 5000, 5001, 5002. We also assume that you have a running Redis master at port 6379 with a slave running at port 6380. We will use the IPv4 loopback address 127.0.0.1 everywhere during the tutorial, assuming you are running the simulation on your personal computer.
 现在我们假设有三个sentinel实例分别监听在5000,5001,5002端口，同时还有一个master实例监听在6379端口，它的slave监听在6380端口。我们会将ip地址设定为127.0.0.1，假定你是用自己的个人电脑来模拟运行。

The three Sentinel configuration files should look like the following:
三个sentinel实例的配置文件应该如下所示：

port 5000
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1

The other two configuration files will be identical but using 5001 and 5002 as port numbers.
另外两个配置文件与上述内容相同，除了指定端口的参数外是5001和5002

A few things to note about the above configuration:
对于上述配置信息有些需要注意的信息如下：

```
The master set is called mymaster. It identifies the master and its slaves. Since each master set has a different name, Sentinel can monitor different sets of masters and slaves at the same time.
此处的master机器代表起名为mymaster，它将这个master实例和它的所有slave实例联系在一起。因为每个master使用不同的名字，所以sentinel可以同时监控多套包含这master实例和slave实例的系统。
The quorum was set to the value of 2 (last argument of sentinel monitor configuration directive).
该集群的法定配额值设置为2（sentinel monitor配置指令的最后一个参数）
The down-after-milliseconds value is 5000 milliseconds, that is 5 seconds, so masters will be detected as failing as soon as we don't receive any reply from our pings within this amount of time.
down-after-milliseconds被设置为5000毫秒，也就是5秒，当master无法在这段时间内回应，sentinel后会立即标记为故障。
```

Once you start the three Sentinels, you'll see a few messages they log, like:
当你启动这三个Sentinel实例后，你会看到记录下来的日志信息，类似如下：

+monitor master mymaster 127.0.0.1 6379 quorum 2

This is a Sentinel event, and you can receive this kind of events via Pub/Sub if you SUBSCRIBE to the event name as specified later.
这是一个Sentinel事件，可以通过Pub/Sub来订阅和接收消息，如果你使用subscribe命令来订阅一个事件名称（后面将会提到），然后就会收到这种类型的事件消息。

Sentinel generates and logs different events during failure detection and failover.
Sentinel在故障检测和故障转移的时候会生成并记录不同类型的事件。

## Asking Sentinel about the state of a master
## 向Sentinel询问某个master的状态

The most obvious thing to do with Sentinel to get started, is check if the master it is monitoring is doing well:
在我们刚开始使用sentinel时，最常见的操作就是查看它监控的master是否正常工作：

```
$ redis-cli -p 5000
127.0.0.1:5000> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
 7) "runid"
 8) "953ae6a589449c13ddefaee3538d356d287f509b"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "735"
19) "last-ping-reply"
20) "735"
21) "down-after-milliseconds"
22) "5000"
23) "info-refresh"
24) "126"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "532439"
29) "config-epoch"
30) "1"
31) "num-slaves"
32) "1"
33) "num-other-sentinels"
34) "2"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "60000"
39) "parallel-syncs"
40) "1"
```

As you can see, it prints a number of information about the master. There are a few that are of particular interest for us:
如上所示，它输出了大量关于master的信息，其中有一些是我们特别要注意的：

```
    num-other-sentinels is 2, so we know the Sentinel already detected two more Sentinels for this master. If you check the logs you'll see the +sentinel events generated.
    num-other-sentinels的值是2，所以我们可以知道Sentinel实例已经侦测到其他两个Sentinel也在监控当前master，当你去检查日志的时候，你会看到有 +sentinel事件生成。
    flags is just master. If the master was down we could expect to see s_down or o_down flag as well here.
    flags显示为master，如果master挂了，同样会在这里s_down或者o_down
    num-slaves is correctly set to 1, so Sentinel also detected that there is an attached slave to our master.
    num-slaves准确得显示为1，所以可以知道sentinel探测到有一个slave挂载在master上。
```

In order to explore more about this instance, you may want to try the following two commands:
为了对这个实例进行更多的探究，你可以试试以下两个命令：

```
SENTINEL slaves mymaster
SENTINEL sentinels mymaster
```

The first will provide similar information about the slaves connected to the master, and the second about the other Sentinels.
第一个命令会获得连接到master上的slaves的信息，类似master信息，第二个命令会获得其他sentinel的信息。

## Obtaining the address of the current master
## 获取当前master的地址

As we already specified, Sentinel also acts as a configuration provider for clients that want to connect to a set of master and slaves. Because of possible failovers or reconfigurations, clients have no idea about who is the currently active master for a given set of instances, so Sentinel exports an API to ask this question:
我们已经指出，如果客户端想连接某个master和它对应的slave，那么sentinel也能够成为配置提供者。由于随时都有可能发生的故障转移和重新配置，客户端不知道这一堆实例集中哪个是当前可用的master，所以sentinel提供了一个API来解决这个问题：

127.0.0.1:5000> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6379"

## Testing the failover
## 测试故障转移

At this point our toy Sentinel deployment is ready to be tested. We can just kill our master and check if the configuration changes. To do so we can just do:
至此我们的玩具Sentinel部署已经可以被测试了。我们可以把master杀掉然后检查配置是否变化，我们可以像这样来操作：

redis-cli -p 6379 DEBUG sleep 30

This command will make our master no longer reachable, sleeping for 30 seconds. It basically simulates a master hanging for some reason.
这个命令会把让master睡眠30秒，在这段时间内会不可达。这种操作基本上模拟了master由于某些原因被卡住的情况。

If you check the Sentinel logs, you should be able to see a lot of action:
如果你检查sentinel的日志，你会看到一系列的操作：

    Each Sentinel detects the master is down with an +sdown event.
    每个Sentinel会检测到master挂了，同时会有+sdown的事件产生。
    This event is later escalated to +odown, which means that multiple Sentinels agree about the fact the master is not reachable.
    这个时间稍后会升级到+odown, 也就是大多数sentinel都认同此master已经不可达的事实。
    Sentinels vote a Sentinel that will start the first failover attempt.
    这些sentinel会投票出一个Sentinel来开始故障转移的尝试。
    The failover happens.
    故障转移开始。

If you ask again what is the current master address for mymaster, eventually we should get a different reply this time:
如果你再次向Sentinel询问mymaster的当前master是哪个，最终会得到一个不一样的回复：

```
127.0.0.1:5000> SENTINEL get-master-addr-by-name mymaster
1) "127.0.0.1"
2) "6380"
```

So far so good... At this point you may jump to create your Sentinel deployment or can read more to understand all the Sentinel commands and internals.
到目前为止都很顺利，至此你可以跳到部署Sentinel的那一步，或者阅读更多相关内容来理解Sentinel的命令和内部原理的部分了。

# Sentinel API

Sentinel provides an API in order to inspect its state, check the health of monitored masters and slaves, subscribe in order to receive specific notifications, and change the Sentinel configuration at run time.
Sentinel提供了一套API，它们主要是用来探测它的状态，检查监控的master和slave的健康状态，订阅相关的频道来收到特定的通知信息，还可以在运行时改变Sentinel的配置信息。

By default Sentinel runs using TCP port 26379 (note that 6379 is the normal Redis port). Sentinels accept commands using the Redis protocol, so you can use redis-cli or any other unmodified Redis client in order to talk with Sentinel.
默认情况下Sentinel会来监听TCP端口26379（注意6379是正常的redis端口）。Sentinel使用Redis协议接收命令，所以你可以使用redis-cli或者其他未经修改的redis客户端来和Sentinel通信。

It is possible to directly query a Sentinel to check what is the state of the monitored Redis instances from its point of view, to see what other Sentinels it knows, and so forth. Alternatively, using Pub/Sub, it is possible to receive push style notifications from Sentinels, every time some event happens, like a failover, or an instance entering an error condition, and so forth.
我们可以向一个Sentinel直接询问它的视角下所监控的Redis实例的状态，查看它所知道的其他的Sentinel实例信息，等等诸如此类的操作。或者我们还可以使用Pub/Sub去接收Sentinel发送的推送类的通知消息，这些通知消息都是在某些事件发生时产生，比如故障转移，或者实例进入错误状态等过程。

## Sentinel commands
## Sentinel命令

The following is a list of accepted commands, not covering commands used in order to modify the Sentinel configuration, which are covered later.
以下是一系列的可用命令，没有包含后面会介绍到的修改Sentinel配置的命令：
```
    PING This command simply returns PONG.
    PING 这个命令仅恢复PONG消息
    SENTINEL masters Show a list of monitored masters and their state.
    SENTINEL masters 展示一系列它所监控的master和他们的状态
    SENTINEL master <master name> Show the state and info of the specified master.
    SENTINEL master <master name>展示指定的master的状态和信息
    SENTINEL slaves <master name> Show a list of slaves for this master, and their state.
    SENTINEL slaves <master name> 展示指定的master的slave信息和状态
    SENTINEL sentinels <master name> Show a list of sentinel instances for this master, and their state.
    SENTINEL sentinels <master name> 展示master的所有监控的sentinel信息以及它们的状态
    SENTINEL get-master-addr-by-name <master name> Return the ip and port number of the master with that name. If a failover is in progress or terminated successfully for this master it returns the address and port of the promoted slave.
    SENTINEL get-master-addr-by-name <master name> 返回指定master名称的ip和端口信息，如果故障转移正在进行中或者已经成功结束，它返回的是即将被提升为master的slave的ip和端口
    SENTINEL reset <pattern> This command will reset all the masters with matching name. The pattern argument is a glob-style pattern. The reset process clears any previous state in a master (including a failover in progress), and removes every slave and sentinel already discovered and associated with the master.
    SENTINEL reset <pattern> 这个命令会重置所有名称与pattern匹配的master的信息。pattern是一个glob风格的模式字符串，reset过程会清除该master之前的所有状态信息（包括正在进行中的故障转移），同时移除该master上所有已经发现的slave和sentinel信息。
    SENTINEL failover <master name> Force a failover as if the master was not reachable, and without asking for agreement to other Sentinels (however a new version of the configuration will be published so that the other Sentinels will update their configurations).
    SENTINEL failover <master name> 强制发起一次故障转移，如同master已经不可达，并且不需要得到其他sentinel的同意（不过这个操作会将新版的配置信息发布出去，其他Sentinel更新它们的配置）
    SENTINEL ckquorum <master name> Check if the current Sentinel configuration is able to reach the quorum needed to failover a master, and the majority needed to authorize the failover. This command should be used in monitoring systems to check if a Sentinel deployment is ok.
    SENTINEL ckquorum <master name> 检查当前的Sentinel配置信息对指定的master是否达到了发起故障转移的配额数量值，以及检查是否达到需要授权进行故障转移的多数派。这个命令可以在监控系统中用来检查Sentinel部署是否正常。
    SENTINEL flushconfig Force Sentinel to rewrite its configuration on disk, including the current Sentinel state. Normally Sentinel rewrites the configuration every time something changes in its state (in the context of the subset of the state which is persisted on disk across restart). However sometimes it is possible that the configuration file is lost because of operation errors, disk failures, package upgrade scripts or configuration managers. In those cases a way to to force Sentinel to rewrite the configuration file is handy. This command works even if the previous configuration file is completely missing.
    SENTINEL flushconfig 强制sentinel重写配置文件以及当前sentinel的状态到磁盘上。正常情况下sentinel会在状态变化的时候重写配置到磁盘(哪些重启后仍然需要保持的状态信息，上文中提到的配置信息的子集)。但是有时候配置文件可能会丢失，比如操作错误，磁盘故障，包升级或者配置管理。这些情况下强制Sentinel重写配置文件是一个很方便的事。这个命令在配置文件缺失的情况下也能正常工作。
```

## Reconfiguring Sentinel at Runtime
## 重配置运行中的Sentinel

Starting with Redis version 2.8.4, Sentinel provides an API in order to add, remove, or change the configuration of a given master. Note that if you have multiple sentinels you should apply the changes to all to your instances for Redis Sentinel to work properly. This means that changing the configuration of a single Sentinel does not automatically propagates the changes to the other Sentinels in the network.
从Redis 2.8.4开始，Sentinel提供了API给指定的master添加，删除或者修改配置项信息。不过需要注意的是，如果你有多个Sentinel实例，你必须对所有的Sentinel实例都修改配置才能正常运行。这就意味着修改单个Sentinel的配置信息并不会自动传播到其他Sentinel上。

The following is a list of SENTINEL sub commands used in order to update the configuration of a Sentinel instance.
以下这些SENTINEL命令的子命令是用来更新Sentinel实例的配置信息。

    SENTINEL MONITOR <name> <ip> <port> <quorum> This command tells the Sentinel to start monitoring a new master with the specified name, ip, port, and quorum. It is identical to the sentinel monitor configuration directive in sentinel.conf configuration file, with the difference that you can't use an hostname in as ip, but you need to provide an IPv4 or IPv6 address.
    SENTINEL MONITOR <name> <ip> <port> <quorum> 这个命令告诉Sentinel开始监控给定master名称，ip, 端口和配额数值的新的master。它和配置文件sentinel.conf中sentinel monitor指令是一样的，有差异的是你不能sentinel命令中使用hostname来作为ip，不过可以用ipv4地址或者ipv6地址作为ip。
    SENTINEL REMOVE <name> is used in order to remove the specified master: the master will no longer be monitored, and will totally be removed from the internal state of the Sentinel, so it will no longer listed by SENTINEL masters and so forth.
    SENTINEL REMOVE <name> 这个命令是用来移除指定名称的master，这个master将不会被监控，并且会被完全得从当前Sentinel实例内部状态中移除，所以也不会从sentinel masters这类命令中被查到。
    SENTINEL SET <name> <option> <value> The SET command is very similar to the CONFIG SET command of Redis, and is used in order to change configuration parameters of a specific master. Multiple option / value pairs can be specified (or none at all). All the configuration parameters that can be configured via sentinel.conf are also configurable using the SET command.
    SENTINEL SET <name> <option> <value> 这个SET命令和redis的CONFIG SET命令非常类似，它是用来修改指定名称的master的配置项，可以在命令中指定多个option/value键值对，或者不指定值也可以。所有可以在sentinel.conf中配置的参数项都可以使用SET命令配置。

The following is an example of SENTINEL SET command in order to modify the down-after-milliseconds configuration of a master called objects-cache:
以下是SENTINEL SET命令的一个示例，修改一个master名为objects-cache的down-after-milliseconds配置参数：

SENTINEL SET objects-cache-master down-after-milliseconds 1000

As already stated, SENTINEL SET can be used to set all the configuration parameters that are settable in the startup configuration file. Moreover it is possible to change just the master quorum configuration without removing and re-adding the master with SENTINEL REMOVE followed by SENTINEL MONITOR, but simply using:
如上所述，SENTINEL SET命令可以设置所有在启动配置文件中指定的配置参数。另外，也可以直接修改配额数量的值，而不需要使用删除和重新添加master的命令来操作（SENTINEL REMOVE和SENTINEL MONITOR），只要执行以下语句：

SENTINEL SET objects-cache-master quorum 5

Note that there is no equivalent GET command since SENTINEL MASTER provides all the configuration parameters in a simple to parse format (as a field/value pairs array).
需要注意的是没有相对应的GET命令查看所有的配置信息，因为SENTINEL MASTER命令可以以一个很简洁的键值对的格式输出给定master的配置项信息。

## Adding or removing Sentinels
## 添加和移除Sentinel节点

Adding a new Sentinel to your deployment is a simple process because of the auto-discover mechanism implemented by Sentinel. All you need to do is to start the new Sentinel configured to monitor the currently active master. Within 10 seconds the Sentinel will acquire the list of other Sentinels and the set of slaves attached to the master.
添加一个新的Sentinel到部署系统中是一个很简单的过程，因为Sentinel实现了自动发现的功能。你需要做的所有事情就是：启动新的Sentinel实例，并将它配置为监控当前的活跃master。在10秒内sentinle就会得到其他sentinel的列表以及当前监控的master的其他slave信息。

If you need to add multiple Sentinels at once, it is suggested to add it one after the other, waiting for all the other Sentinels to already know about the first one before adding the next. This is useful in order to still guarantee that majority can be achieved only in one side of a partition, in the chance failures should happen in the process of adding new Sentinels.
如果你需要同时添加多个sentinel，建议逐个地添加，在添加新的之前确保之前添加的sentinel已经被当前系统的其他sentinel知晓。这可以保证在添加新的Sentinel的时候如果突发故障，多数派只会出现在其中分区的一边。

This can be easily achieved by adding every new Sentinel with a 30 seconds delay, and during absence of network partitions.
在没有网络分区的情况下，每30s间隔添加一个新的sentinel可以保证多数派只出现在一个分区的一边。

At the end of the process it is possible to use the command SENTINEL MASTER mastername in order to check if all the Sentinels agree about the total number of Sentinels monitoring the master.
在添加完所有sentinel之后，可以使用SENTINEL MASTER mastername命令来查看是否所有的sentinel都对当前监控的master的sentinel数量达成一致。

Removing a Sentinel is a bit more complex: Sentinels never forget already seen Sentinels, even if they are not reachable for a long time, since we don't want to dynamically change the majority needed to authorize a failover and the creation of a new configuration number. So in order to remove a Sentinel the following steps should be performed in absence of network partitions:
移除一个Sentinel有一点复杂，sentinels从不会忘记已经记住的其他Sentinel，即使它们已经长时间不可达了，因为我们不想让系统动态调整多数派的值（这个值会与授权发起故障转移和创建一个新的配置号有关系）。所以为了移除一个sentinel，在没有网络分区的情况下，我们需要按照以下操作来一步步进行：

    Stop the Sentinel process of the Sentinel you want to remove.
    停掉需要移除的sentinel进程
    Send a SENTINEL RESET * command to all the other Sentinel instances (instead of * you can use the exact master name if you want to reset just a single master). One after the other, waiting at least 30 seconds between instances.
    发送一个 SENTINEL RESET *命令给其他所有的sentinel实例（如果你只想移除某一个指定的master的监控可以用master名称替代*）, 在给sentinel发送这个命令的时候，操作间隔需要等待至少30s
    Check that all the Sentinels agree about the number of Sentinels currently active, by inspecting the output of SENTINEL MASTER mastername of every Sentinel.
    向所有其他的sentinel发送SENTINEL MASTER mastername命令，检查他们是否对当前活跃的sentinel数量达成一致。

## Removing the old master or unreachable slaves
## 移除旧的master或者不可达的slaves

Sentinels never forget about slaves of a given master, even when they are unreachable for a long time. This is useful, because Sentinels should be able to correctly reconfigure a returning slave after a network partition or a failure event.
Sentinels从不会忘记它之前监测到的某个master的slave，即使它已经长时间不可达了。在网络分区或者故障事件恢复后，这是很有用的，sentinel可以正确得配置它们。

Moreover, after a failover, the failed over master is virtually added as a slave of the new master, this way it will be reconfigured to replicate with the new master as soon as it will be available again.
另外，在发生故障转移后，被转移的master实质上会以slave的角色被添加到新的master上，这样在老master可用后可以立即被配置为从新master同步数据。

However sometimes you want to remove a slave (that may be the old master) forever from the list of slaves monitored by Sentinels.
但是有时你也会想从一个sentinel系统中移除某个被监控的slave。

In order to do this, you need to send a SENTINEL RESET mastername command to all the Sentinels: they'll refresh the list of slaves within the next 10 seconds, only adding the ones listed as correctly replicating from the current master INFO output.
为了达到这个目的，你需要向所有的sentinel发送SENTINEL RESET mastername命令，它们会在接下来的10秒内刷新slave列表，只会添加从master得到的INFO命令中列举的正常复制的实例信息。

## Pub/Sub Messages
## 订阅和发布消息

A client can use a Sentinel as it was a Redis compatible Pub/Sub server (but you can't use PUBLISH) in order to SUBSCRIBE or PSUBSCRIBE to channels and get notified about specific events.
客户端可以使用Sentinel来订阅和发布信息到频道上，从频道内得到特定的事件信息，因为sentinel实例也是一个兼容Pub/Sub功能的Redis服务器（不过不能使用PUBLISH命令）。

The channel name is the same as the name of the event. For instance the channel named +sdown will receive all the notifications related to instances entering an SDOWN (SDOWN means the instance is no longer reachable from the point of view of the Sentinel you are querying) condition.
频道名称和事件名称一致，例如一个名称为+sdown的频道会接收到进入SDOWN状态实例的相关的通知信息。SDOWN意味着在Sentinel的视角下该实例已经不可达。

To get all the messages simply subscribe using PSUBSCRIBE *.
为了获得所有的信息，你可以简单得使用PSUBSCRIBE *

The following is a list of channels and message formats you can receive using this API. The first word is the channel / event name, the rest is the format of the data.
以下内容是你可以使用这个API接收到的频道和信息格式，消息的第一个单词部分是频道或者事件名称，其他的部分是数据。

Note: where instance details is specified it means that the following arguments are provided to identify the target instance:
注：当instance details被指定了就意味着以下参数会显示出来以唯一确定是哪个实例：

<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>

The part identifying the master (from the @ argument to the end) is optional and is only specified if the instance is not a master itself.
标记来源master的消息部分（从@到消息末尾）是可选的，在发送消息的实例不是master的时候会被显示出来。

    +reset-master <instance details> -- The master was reset.
    +reset-master <instance details> --master被重置
    +slave <instance details> -- A new slave was detected and attached.
    +slave <instance details> -- 一个新的slave被检查到并添加到系统中
    +failover-state-reconf-slaves <instance details> -- Failover state changed to reconf-slaves state.
    +failover-state-reconf-slaves <instance details> -- 故障转移切换到reconf-slaves状态
    +failover-detected <instance details> -- A failover started by another Sentinel or any other external entity was detected (An attached slave turned into a master).
    +failover-detected <instance details> -- 其他sentinel发起一次故障转移，或者其他外部实体故障转移
    +slave-reconf-sent <instance details> -- The leader sentinel sent the SLAVEOF command to this instance in order to reconfigure it for the new slave.
    +slave-reconf-sent <instance details> -- leader sentinel发送SLAVEOF命令给对应的实例将它配置为新的slave
    +slave-reconf-inprog <instance details> -- The slave being reconfigured showed to be a slave of the new master ip:port pair, but the synchronization process is not yet complete.
    +slave-reconf-inprog <instance details> -- 新的slave已经被挂到新master上，但是同步过程还没有结束
    +slave-reconf-done <instance details> -- The slave is now synchronized with the new master.
    +slave-reconf-done <instance details> -- 新的slave完成了从新master同步数据的过程
    -dup-sentinel <instance details> -- One or more sentinels for the specified master were removed as duplicated (this happens for instance when a Sentinel instance is restarted).
    -dup-sentinel <instance details> -- 由于该sentinel实例与当前已有sentinel实例重复（当sentinel实例重启的时候会发生），将重复的一个或多个sentinel移除掉。
    +sentinel <instance details> -- A new sentinel for this master was detected and attached.
    +sentinel <instance details> -- 监控master的新的sentinel被检测到并上线
    +sdown <instance details> -- The specified instance is now in Subjectively Down state.
    +sdown <instance details> -- 指明实例当前是主观下线状态
    -sdown <instance details> -- The specified instance is no longer in Subjectively Down state.
    -sdown <instance details> -- 指明的实例不再是主观下线状态
    +odown <instance details> -- The specified instance is now in Objectively Down state.
    +odown <instance details> -- 指明的实例现在是客观下线状态
    -odown <instance details> -- The specified instance is no longer in Objectively Down state.
    -odown <instance details> -- 指明的实例不再是客观下线状态
    +new-epoch <instance details> -- The current epoch was updated.
    +new-epoch <instance details> -- 当前的配置纪元更新了
    +try-failover <instance details> -- New failover in progress, waiting to be elected by the majority.
    +try-failover <instance details> -- 新的故障转移开始，等待多数派选举出leader
    +elected-leader <instance details> -- Won the election for the specified epoch, can do the failover.
    +elected-leader <instance details> -- 在指定的配置纪元内赢得选举，可以开始故障转移
    +failover-state-select-slave <instance details> -- New failover state is select-slave: we are trying to find a suitable slave for promotion.
    +failover-state-select-slave <instance details> -- 故障转移状态切换到select-slave状态，表示正在选出一个合适的slave来提升为master
    no-good-slave <instance details> -- There is no good slave to promote. Currently we'll try after some time, but probably this will change and the state machine will abort the failover at all in this case.
    no-good-slave <instance details> -- 没有好的slave可以提升，稍后会重试，但是可能会导致状态变更，同时状态机放弃故障转移。
    selected-slave <instance details> -- We found the specified good slave to promote.
    selected-slave <instance details> -- 发现好的slave可以提升
    failover-state-send-slaveof-noone <instance details> -- We are trying to reconfigure the promoted slave as master, waiting for it to switch.
    failover-state-send-slaveof-noone <instance details> -- 故障转移状态到send-slaveof-noone状态，表示正在尝试重新配置新的可提升的slave为master，等待它切换为master
    failover-end-for-timeout <instance details> -- The failover terminated for timeout, slaves will eventually be configured to replicate with the new master anyway.
    failover-end-for-timeout <instance details> -- 故障转移因为超时终止，不过slaves最终会被配置为从新的master进行复制同步。
    failover-end <instance details> -- The failover terminated with success. All the slaves appears to be reconfigured to replicate with the new master.
    failover-end <instance details> -- 故障转移过程成功完成，所有的slave应该都正确得从新的master进行复制同步。
    switch-master <master name> <oldip> <oldport> <newip> <newport> -- The master new IP and address is the specified one after a configuration change. This is the message most external users are interested in.
    switch-master <master name> <oldip> <oldport> <newip> <newport> -- 新的master的IP和地址经过重新配置已经生效，这个信息是大多数外部用户关注的东西。
    +tilt -- Tilt mode entered.
    +tilt -- 进入Tilt模式
    -tilt -- Tilt mode exited.
    -tilt -- 退出Tilt模式

## Handling of -BUSY state
## 处理-BUSY状态

The -BUSY error is returned by a Redis instance when a Lua script is running for more time than the configured Lua script time limit. When this happens before triggering a fail over Redis Sentinel will try to send a SCRIPT KILL command, that will only succeed if the script was read-only.
-BUSY错误是在Redis实例运行Lua脚本时间超过配置中指定的lua脚本运行时间限制参数后会返回的信息，当这种情况发生在sentinel发起故障转移之前，sentinel会尝试发送SCRIPT KILL命令，不过只有当脚本是只读的时候才会成功。

If the instance will still be in an error condition after this try, it will eventually be failed over.
如果redis实例在尝试KILL之后还是一直处于这种错误状态，sentinel还是会发起故障转移。

## Slaves priority
## Slave 优先级

Redis instances have a configuration parameter called slave-priority. This information is exposed by Redis slave instances in their INFO output, and Sentinel uses it in order to pick a slave among the ones that can be used in order to failover a master:
Redis实例有一个配置参数称为slave-priority. 这个信息会在slave实例的INFO输出中展示出来，sentinel用它作为参考值来从slave中选择一个可以作为提升master：

    If the slave priority is set to 0, the slave is never promoted to master.
    如果slave-priority设置为0，那么slave就永远不会被提升为master
    Slaves with a lower priority number are preferred by Sentinel.
    slave-priority值越低的slave，越倾向于会被选为master

For example if there is a slave S1 in the same data center of the current master, and another slave S2 in another data center, it is possible to set S1 with a priority of 10 and S2 with a priority of 100, so that if the master fails and both S1 and S2 are available, S1 will be preferred.
例如有一个slave S1和当前的master在同一个数据中心，同时另一个slave S2在另一个数据中心，那么可以将S1的priority设置为10，S2的priority设置为100，这样当master宕机后S1和S2都可用的情况下，S1会被选择为master

For more information about the way slaves are selected, please check the slave selection and priority section of this documentation.
如果想了解更多关于选取slave作为新master的信息，请查看本文档中slave选择和优先级选择那一节。

## Sentinel and Redis authentication
## Sentinel和Redis授权

When the master is configured to require a password from clients, as a security measure, slaves need to also be aware of this password in order to authenticate with the master and create the master-slave connection used for the asynchronous replication protocol.
当master为了安全考虑，要求客户端必须使用密码登录后，slave也需要知道这个密码以获得授权来重新创建master-slave复制的连接。

This is achieved using the following configuration directives:
这个可以通过使用以下指令来达到目的：

    requirepass in the master, in order to set the authentication password, and to make sure the instance will not process requests for non authenticated clients.
    在master中的requirepass指令指定了客户端的登录密码，以确保未授权的请求不会被处理。
    masterauth in the slaves in order for the slaves to authenticate with the master in order to correctly replicate data from it.
    在slave上的masterauth指令指定了连接它进行复制同步的master的登录密码，以便于在创建master-slave复制的连接时使用。

When Sentinel is used, there is not a single master, since after a failover slaves may play the role of masters, and old masters can be reconfigured in order to act as slaves, so what you want to do is to set the above directives in all your instances, both masters and slaves.
当使用了Sentinel的时候，master实例不会只限于当前唯一的master实例，因为一次故障转移后有slave可能会取代原来的master成为新的master，原来的master就会变成新的slave，所以我们需要在所有的实例上都执行上述指令，包括master和slave。

This is also usually a sane setup since you don't want to protect data only in the master, having the same data accessible in the slaves.
这通常是一个明智的选择,因为你不会仅仅只想保护master的数据而让slave的数据任意访问。

However, in the uncommon case where you need a slave that is accessible without authentication, you can still do it by setting up a slave priority of zero, to prevent this slave from being promoted to master, and configuring in this slave only the masterauth directive, without using the requirepass directive, so that data will be readable by unauthenticated clients.
但是, 有些不寻常的情况下,你需要一个不需要授权访问的slave, 你可以给这个slave设置priority为0以禁止它被提升为master, 同时仅配置这个slave的masterauth指令,不指定requirepass指令,这样它就可以接受未授权的客户端请求.

In order for sentinels to connect to Redis server instances when they are configured with requirepass, the Sentinel configuration must include the sentinel auth-pass directive, in the format:
为了使sentinel可以连接配置了requirepass的Redis server,sentinel实例的配置需要包括sentinel auth-pass指令, 格式如下:

sentinel auth-pass <master-group-name> <pass>

## Configuring Sentinel instances with authentication
## 配置Sentinel实例的登录授权

You can also configure the Sentinel instance itself in order to require client authentication via the AUTH command, however this feature is only available starting with Redis 5.0.1.
你也可以给Sentinel实例本身配置登录授权的密码, 但是这个特性只在5.0.1之后的版本可用.

In order to do so, just add the following configuration directive to all your Sentinel instances:
你可以在sentinel的配置文件中加入以下指令信息:

requirepass "your_password_here"

When configured this way, Sentinels will do two things:
在配置了这个指令信息后,sentinel会做两件事:

    A password will be required from clients in order to send commands to Sentinels. This is obvious since this is how such configuration directive works in Redis in general.
    客户端需要使用密码来登录sentinel,这是显然的,因为这个指令的目的就是如此.
    Moreover the same password configured to access the local Sentinel, will be used by this Sentinel instance in order to authenticate to all the other Sentinel instances it connects to.
    并且该密码会同时用来登录到其他它所连接的Sentinel实例。

This means that you will have to configure the same requirepass password in all the Sentinel instances. This way every Sentinel can talk with every other Sentinel without any need to configure for each Sentinel the password to access all the other Sentinels, that would be very impractical.
这就意味着你需要将所有的sentinel实例的requirepass参数设置为相同的密码。这样每个sentinel都能相互交流，不必为每个sentinel配置登录其他各个sentinel的密码，因为这种方式是很不切实际的。

Before using this configuration make sure your client library is able to send the AUTH command to Sentinel instances.
在使用这个配置密码的功能前你需要确保客户端动态库支持向Sentinel发送AUTH命令的功能。

## Sentinel clients implementation
## Sentinel客户端的实现

Sentinel requires explicit client support, unless the system is configured to execute a script that performs a transparent redirection of all the requests to the new master instance (virtual IP or other similar systems). The topic of client libraries implementation is covered in the document Sentinel clients guidelines.
Sentinel client功能需要显式支持，除非sentinel系统配置了透明重定向请求到新master实例的脚本。相关的主题可以在Sentinel clients guidelines中查看。

# More advanced concepts

In the following sections we'll cover a few details about how Sentinel work, without to resorting to implementation details and algorithms that will be covered in the final part of this document.
在下列章节中我们会讲到一些sentinel运行原理的详细内容，不过不会太深入到一些具体的实现细节和算法，这些会在本文档最后一个章节中介绍到。

## SDOWN and ODOWN failure state
## SDOWN和ODOWN故障状态

Redis Sentinel has two different concepts of being down, one is called a Subjectively Down condition (SDOWN) and is a down condition that is local to a given Sentinel instance. Another is called Objectively Down condition (ODOWN) and is reached when enough Sentinels (at least the number configured as the quorum parameter of the monitored master) have an SDOWN condition, and get feedback from other Sentinels using the SENTINEL is-master-down-by-addr command.
Redis Sentinel对实例的下线判断有两个不同的概念，一个是称为主观下线的状态(SDOWN)，这是指定的Sentinel实例对其他实例的在线状态的本地判断。另一个是称为客观下线的状态(ODOWN)，这种状态是在足够数量的Sentinel认为该实例已经是主观下线，并且当前的Sentinel已经从其他的Sentinel通过SENTINEL is-master-down-by-addr命令获得下线的反馈。

From the point of view of a Sentinel an SDOWN condition is reached when it does not receive a valid reply to PING requests for the number of seconds specified in the configuration as is-master-down-after-milliseconds parameter.
从某一个Sentinel的视角来看，只有当它在is-master-down-after-milliseconds参数所配置的时间内没有收到对某个实例的PING请求的有效回复，这个Sentinel就会把该实例标记为SDOWN状态。

An acceptable reply to PING is one of the following:
以下PING消息的回复是可以接受的：

    PING replied with +PONG.
    PING 返回+PONG
    PING replied with -LOADING error.
    PING返回-LOADING错误
    PING replied with -MASTERDOWN error.
    PING返回-MASTERDOWN错误

Any other reply (or no reply at all) is considered non valid. However note that a logical master that advertises itself as a slave in the INFO output is considered to be down.
其他的PING回复或者没有回复都会被认为是不可用的，不过还要注意逻辑上为master角色的实例然而实际上是slave(该实例的INFO广播实际为slave)时，也会被标记为down

Note that SDOWN requires that no acceptable reply is received for the whole interval configured, so for instance if the interval is 30000 milliseconds (30 seconds) and we receive an acceptable ping reply every 29 seconds, the instance is considered to be working.
注意SDOWN要求在配置中指定的时间段内没有任何有效的回复，所以如果这个时间段是30s的时候，我们观察到每29s能收到一个有效ping回复，则这个实例就可以认为是仍然正常运行的。

SDOWN is not enough to trigger a failover: it only means a single Sentinel believes a Redis instance is not available. To trigger a failover, the ODOWN state must be reached.
达到SDOWN状态不足以触发一个故障转移，它只意味着一个单独的sentinel认为另一个单独的redis实例不可用。为了触发这个故障转移，Sentinel必须达到ODOWN状态。

To switch from SDOWN to ODOWN no strong consensus algorithm is used, but just a form of gossip: if a given Sentinel gets reports that a master is not working from enough Sentinels in a given time range, the SDOWN is promoted to ODOWN. If this acknowledge is later missing, the flag is cleared.
为了从SDOWN状态切换到ODOWN状态，sentinel没有使用强一致性算法，而是一种流言协议：如果一个给定的sentinel得到报告某个master在给定的时间段内已经无法被足够数量的sentinel确定为可用状态，那么这个master就会被sentinel从SDOWN状态提升为ODOWN状态。如果之后又有足够数量的sentinel确定为可用状态，ODOWN状态标记就会被清除。

A more strict authorization that uses an actual majority is required in order to really start the failover, but no failover can be triggered without reaching the ODOWN state.
在发起故障转移之前，我们需要使用一个更严格的授权机制来判断是否有真实的多数派授权，但是仍然需要这个实例是在达到ODOWN状态的情况下才会发生。

The ODOWN condition only applies to masters. For other kind of instances Sentinel doesn't require to act, so the ODOWN state is never reached for slaves and other sentinels, but only SDOWN is.
ODOWN状态只会应用在master实例上，因为别的类型的实例不需要进行故障转移，所以slaves和sentinels实例不会进入ODOWN状态，只可能会进入SDOWN状态。

However SDOWN has also semantic implications. For example a slave in SDOWN state is not selected to be promoted by a Sentinel performing a failover.
然而SDOWN状态也有隐含的作用，比如一个SDOWN状态的slave不会被Sentinel在发起故障转移时选择作为提升为master的候选实例。

## Sentinels and Slaves auto discovery
## Sentinels和Slaves的自动发现

Sentinels stay connected with other Sentinels in order to reciprocally check the availability of each other, and to exchange messages. However you don't need to configure a list of other Sentinel addresses in every Sentinel instance you run, as Sentinel uses the Redis instances Pub/Sub capabilities in order to discover the other Sentinels that are monitoring the same masters and slaves.
每个Sentinels会保持与其他的Sentinels的连接，以便于相互检查可用性，并交换信息。然而你不需要为每个运行的Sentinel配置一个其他Sentinel的列表，因为Sentinel会使用Redis实例的Pub/Sub功能来发现监控相同masters和slaves的其他Sentinel实例。

This feature is implemented by sending hello messages into the channel named __sentinel__:hello.
这个特性的是通过向名称为__sentinel__:hello的频道发送hello信息实现的。

Similarly you don't need to configure what is the list of the slaves attached to a master, as Sentinel will auto discover this list querying Redis.
类似地，你不需要配置连接到master的slave列表，因为Sentinel会向Redis发送查询来自动发现。

    Every Sentinel publishes a message to every monitored master and slave Pub/Sub channel __sentinel__:hello, every two seconds, announcing its presence with ip, port, runid.
    每个Sentinel会向它监控的每个master和slave上的__sentinel__:hello频道发布信息，发送间隔为2s，其中包含ip，port, runid信息，以此声明自己的存在。
    Every Sentinel is subscribed to the Pub/Sub channel __sentinel__:hello of every master and slave, looking for unknown sentinels. When new sentinels are detected, they are added as sentinels of this master.
    每个Sentinel会从它监控的每个master和slave上的__sentinel__:hello频道订阅信息，查找未知的sentinels实例，当有新的sentinel被探测到，它们会被加入到当前此master的sentinel监控列表中。
    Hello messages also include the full current configuration of the master. If the receiving Sentinel has a configuration for a given master which is older than the one received, it updates to the new configuration immediately.
    Hello信息也会包含master的完整的当前配置信息。如果收到该信息的Sentinel有一个比此信息更旧的配置信息，它就会立即更新到新的配置信息。
    Before adding a new sentinel to a master a Sentinel always checks if there is already a sentinel with the same runid or the same address (ip and port pair). In that case all the matching sentinels are removed, and the new added.
    在为一个master添加一个新的sentinel之前，sentinel总是会检查是否有相同的runid或者相同地址的sentinel已经存在，这种情况下所有满足条件的sentinel都会被移除，然后将新的添加进来。

## Sentinel reconfiguration of instances outside the failover procedure
## 在故障转移之外的Sentinel重配置机制

Even when no failover is in progress, Sentinels will always try to set the current configuration on monitored instances. Specifically:
即使没有故障转移正在进行，Sentinels也会经常尝试为监控的实例设置当前的配置。特别是以下的情景：

    Slaves (according to the current configuration) that claim to be masters, will be configured as slaves to replicate with the current master.
    申明自己为master的Slaves会被配置为slave，将会从当前的master复制同步数据。
    Slaves connected to a wrong master, will be reconfigured to replicate with the right master.
    连接到错误的master的Slaves将会被重新配置为从正确的master上复制同步数据。

For Sentinels to reconfigure slaves, the wrong configuration must be observed for some time, that is greater than the period used to broadcast new configurations.
使用sentinel来重配置slave是因为错误的配置信息必须要被检测到，这比定期去广播新的配置信息要好。

This prevents Sentinels with a stale configuration (for example because they just rejoined from a partition) will try to change the slaves configuration before receiving an update.
这样可以防止一个有旧配置信息的sentinel在收到更新前尝试去改变slave的配置信息。

Also note how the semantics of always trying to impose the current configuration makes the failover more resistant to partitions:
同时你会发现这种不断尝试强行应用当前配置信息的做法使故障转移对网络隔离更有抵抗力：

    Masters failed over are reconfigured as slaves when they return available.
    发生故障的masters一旦可用之后就就会被重新配置为slave。
    Slaves partitioned away during a partition are reconfigured once reachable.
    被网络隔离的slaves一旦脱离网络分区，就会被重新配置。

The important lesson to remember about this section is: Sentinel is a system where each process will always try to impose the last logical configuration to the set of monitored instances.
这一节最重要的信息是，sentinel系统会一直尝试将逻辑上最新的配置信息应用到所有监控的实例对象。

## Slave selection and priority
## Slave的选主和优先级

When a Sentinel instance is ready to perform a failover, since the master is in ODOWN state and the Sentinel received the authorization to failover from the majority of the Sentinel instances known, a suitable slave needs to be selected.
当Sentinel实例开始执行一次故障转移，当且仅当master被标记为ODOWN状态，并且Sentinel实例收到了来自多数派其他Sentinel实例授权来发起故障转移，这时一个合适的slave就需要被选出来。

The slave selection process evaluates the following information about slaves:
slave选择流程需要评估每个slave的以下信息：

    Disconnection time from the master.
    与master的断连时间
    Slave priority.
    slave的优先级
    Replication offset processed.
    复制偏移
    Run ID.
    RunID

A slave that is found to be disconnected from the master for more than ten times the configured master timeout (down-after-milliseconds option), plus the time the master is also not available from the point of view of the Sentinel doing the failover, is considered to be not suitable for the failover and is skipped.
当slave与master的失联时间超过10倍的down-after-milliseconds时间加上该sentinel计算的master断连时间，此时就可以认为slave不能与master连接，不能作为可选的故障转移实例，直接跳过。

In more rigorous terms, a slave whose the INFO output suggests to be disconnected from the master for more than:
可以使用更严格的术语来描述，当一个slave的INFO信息中所包含的与master失联的时间超过：

(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_state

Is considered to be unreliable and is disregarded entirely.
就会被认为不可达并且完全不会被考虑

The slave selection only considers the slaves that passed the above test, and sorts it based on the above criteria, in the following order.
选主只会考虑通过以上测试的slave实例，然后使用上述标准按照如下规则来排序：

    The slaves are sorted by slave-priority as configured in the redis.conf file of the Redis instance. A lower priority will be preferred.
    slaves会被按照在redis.conf中定义的slave-priority的值排序，priority值越小优先级越高。
    If the priority is the same, the replication offset processed by the slave is checked, and the slave that received more data from the master is selected.
    如果优先级一样，那么slave的复制偏移就会成为考虑因素，从master收到更多数据的slave会优先被选择。
    If multiple slaves have the same priority and processed the same data from the master, a further check is performed, selecting the slave with the lexicographically smaller run ID. Having a lower run ID is not a real advantage for a slave, but is useful in order to make the process of slave selection more deterministic, instead of resorting to select a random slave.
    如果多个slave有相同的优先级，并且拥有同样多的数据，那么就再进行进一步的检查，选择runid按字典序排在前面的slave。使用runnid的字典序靠前的slave并不是因为有什么实在的优势，只是为了在选择slave的时候有确定性，而不是随机性得选择一个slave

Redis masters (that may be turned into slaves after a failover), and slaves, all must be configured with a slave-priority if there are machines to be strongly preferred. Otherwise all the instances can run with the default run ID (which is the suggested setup, since it is far more interesting to select the slave by replication offset).
如果有一些机器更倾向于成为master实例所在机器，那么这些机器上的Redis master（可能是在故障转移后变成了slave的master）和slaves就必须要配置一个slave-priority值。否则这些实例就会使用默认的runid运行，这是推荐的做法，因为这样就会更大可能使用复制偏移量作为参数来决定哪一个会成为master。

A Redis instance can be configured with a special slave-priority of zero in order to be never selected by Sentinels as the new master. However a slave configured in this way will still be reconfigured by Sentinels in order to replicate with the new master after a failover, the only difference is that it will never become a master itself.
我们可以将一个Redis实例的slave-priority配置为0作为特殊的slave，这是为了避免它被Sentinel选为master。然而当一个slave的slave-priority被配置为0后，如果发生故障转移后，它还会被Sentinel重新配置为从新的master继续复制数据，不同的是这个实例永远不会成为master。

# Algorithms and internals
# 算法和内部实现

In the following sections we will explore the details of Sentinel behavior. It is not strictly needed for users to be aware of all the details, but a deep understanding of Sentinel may help to deploy and operate Sentinel in a more effective way.
在以下章节中，我们会浏览Sentinel的行为细节，不需要用户完全掌握这些细节，但是对sentinel有一个深刻的理解会对部署和操作sentinel会有很大的帮助。

## Quorum
## 数量配额值

The previous sections showed that every master monitored by Sentinel is associated to a configured quorum. It specifies the number of Sentinel processes that need to agree about the unreachability or error condition of the master in order to trigger a failover.
前一个章节展示了sentinel监控的每个master都会与一个配额关联，这个配额指定了发起故障转移所需要收到的同意故障转移的的sentinel进程数量。

However, after the failover is triggered, in order for the failover to actually be performed, at least a majority of Sentinels must authorize the Sentinel to failover. Sentinel never performs a failover in the partition where a minority of Sentinels exist.
然而，在故障转移发起后，要真正去执行故障转移，还得至少多数派的sentinels来授权才能去执行。当集群中只有少数派的sentinel存在时，sentinel永远不会去执行故障转移。

Let's try to make things a bit more clear:
我们可以把这个过程解释得更清晰一些：

    Quorum: the number of Sentinel processes that need to detect an error condition in order for a master to be flagged as ODOWN.
    配额：由于检测到master错误从而将master标记为ODOWN的sentinel的数量
    The failover is triggered by the ODOWN state.
    这个故障转移是由ODOWN状态触发的。
    Once the failover is triggered, the Sentinel trying to failover is required to ask for authorization to a majority of Sentinels (or more than the majority if the quorum is set to a number greater than the majority).
    一旦故障转移被触发，尝试发起故障转移的Sentinel就会去请求多数派的sentinel授权（如果配额值高于多数派的数量那么这里的sentinel数量也会超过多数派）

The difference may seem subtle but is actually quite simple to understand and use. For example if you have 5 Sentinel instances, and the quorum is set to 2, a failover will be triggered as soon as 2 Sentinels believe that the master is not reachable, however one of the two Sentinels will be able to failover only if it gets authorization at least from 3 Sentinels.
配额值和多数派值的不同之处可能比较晦涩，但是也很容易理解并应用。例如，在5个sentinel实例的情况下，配额值设置为2，那么一次故障转移在2个sentinel认为master不可达的情况下就会被立即触发，但是这两个sentinel其中之一只会在得到至少3个sentinel实例的授权情况下才会执行故障转移。

If instead the quorum is configured to 5, all the Sentinels must agree about the master error condition, and the authorization from all Sentinels is required in order to failover.
如果配额设置的值是5，那么必须所有的sentinel对master的不可达错误达成一致，并且要得到所有的sentinel授权后才能执行故障转移。

This means that the quorum can be used to tune Sentinel in two ways:
这就意味着配额可以使用两种方式调整sentinel的行为：

    If a the quorum is set to a value smaller than the majority of Sentinels we deploy, we are basically making Sentinel more sensible to master failures, triggering a failover as soon as even just a minority of Sentinels is no longer able to talk with the master.
    如果配额的值设置为比多数派数量小的值时，我们大致会使sentinel对master的错误更加敏感，只要小部分的sentinel发现无法与master通信就会立即发起一次故障转移。
    If a quorum is set to a value greater than the majority of Sentinels, we are making Sentinel able to failover only when there are a very large number (larger than majority) of well connected Sentinels which agree about the master being down.
    如果配置的值设置得大于多数派的值，那么在大量的（超过多数派数量）正常连通的sentinel实例都认同master故障的时候，我们才会让sentinel发起故障转移。

## Configuration epochs
## 配置信息epoch

Sentinels require to get authorizations from a majority in order to start a failover for a few important reasons:
sentinels需要从多数派获得授权后才能执行故障转移，是由于以下一些重要的原因：

When a Sentinel is authorized, it gets a unique configuration epoch for the master it is failing over. This is a number that will be used to version the new configuration after the failover is completed. Because a majority agreed that a given version was assigned to a given Sentinel, no other Sentinel will be able to use it. This means that every configuration of every failover is versioned with a unique version. We'll see why this is so important.
当一次sentinel被授权时，它会得到一个唯一的配置epoch值与当前正在故障转移的master相对应，这个数字在完成故障转移后会作为新配置信息的版本号。由于多数派的sentinel已经同意将该版本号赋予了给定的sentinel，其他的sentinel就无法再去拥有它。这就意味着每次故障转移都会有一个唯一的版本号来被标记到配置信息，我们将会看到为什么这会如此重要。

Moreover Sentinels have a rule: if a Sentinel voted another Sentinel for the failover of a given master, it will wait some time to try to failover the same master again. This delay is the failover-timeout you can configure in sentinel.conf. This means that Sentinels will not try to failover the same master at the same time, the first to ask to be authorized will try, if it fails another will try after some time, and so forth.
此外sentinel有一项规则，如果一个sentinel为另一个发起故障转移投票的sentinel投过票了，那么它会等待一段时间后才会再为同一个master尝试发起故障转移，这个等待延迟时间是在sentinel.conf文件中配置的failover-timeout参数。这就意味着sentinel不会在同一时间为相同的master尝试发起故障转移，第一个请求授权的sentinel会首先尝试，如果它失败了，那么过一段时间后才会有其他sentinel继续发起故障转移。

Redis Sentinel guarantees the liveness property that if a majority of Sentinels are able to talk, eventually one will be authorized to failover if the master is down.
Redis sentinel的保活性保证了在多数派的sentinel都能够连通的时候，如果有master发生故障，最终肯定会有一个sentinel会被授权执行故障转移。

Redis Sentinel also guarantees the safety property that every Sentinel will failover the same master using a different configuration epoch.
redis sentinel的安全性保证了每个sentinel在对同一个master进行故障转移时能使用不同的配置epoch。

## Configuration propagation
## 配置信息的传播

Once a Sentinel is able to failover a master successfully, it will start to broadcast the new configuration so that the other Sentinels will update their information about a given master.
一旦sentinel能够成功完成一个master的故障转移，它就会广播新的配置信息,以便于监控同一个master的其他的Sentinel更新它们的信息。

For a failover to be considered successful, it requires that the Sentinel was able to send the SLAVEOF NO ONE command to the selected slave, and that the switch to master was later observed in the INFO output of the master.
判断一次故障转移是否成功，需要sentinel至少能够发送SLAVEOF NO ONE命令给选定的slave，这个操作之后在master的INFO输出中就能观察到切换到master的情况。

At this point, even if the reconfiguration of the slaves is in progress, the failover is considered to be successful, and all the Sentinels are required to start reporting the new configuration.
此时，即使slave的重配置还在进行中，故障转移已经可以认为是成功了，然后所有的sentinel就需要开始上报新的配置信息。

The way a new configuration is propagated is the reason why we need that every Sentinel failover is authorized with a different version number (configuration epoch).
之前我们为每个发起故障转移的sentinel分配一个不一样的版本号的原因就是为了使用这种方式来传播新的配置信息。

Every Sentinel continuously broadcast its version of the configuration of a master using Redis Pub/Sub messages, both in the master and all the slaves. At the same time all the Sentinels wait for messages to see what is the configuration advertised by the other Sentinels.
每个sentinel使用redis pub/sub消息将它自己的配置信息版本持续得广播到master和它所有的slave上，与此同时，所有的sentinel也都会等待其他sentinel广播的配置信息。

Configurations are broadcast in the __sentinel__:hello Pub/Sub channel.
配置信息是在__sentinel__:hello频道进行广播和订阅的。

Because every configuration has a different version number, the greater version always wins over smaller versions.
因为每个配置信息都会有不一样的版本号，更大的版本号的配置信息会优先于小的版本号被采用。

So for example the configuration for the master mymaster start with all the Sentinels believing the master is at 192.168.1.50:6379. This configuration has version 1. After some time a Sentinel is authorized to failover with version 2. If the failover is successful, it will start to broadcast a new configuration, let's say 192.168.1.50:9000, with version 2. All the other instances will see this configuration and will update their configuration accordingly, since the new configuration has a greater version.
比如当前的名称为mymaster的集群组对应的配置信息中标记master为192.168.1.50:6379，配置的版本号为1，当某个sentinel被授权执行故障转移后，赋予的版本号为2.如果故障转移成功，它就会广播新的配置信息，比如新master为192.168.1.50:9000，对应的版本号为2. 所有其他实例会看到这个配置信息，然后会根据与当前版本号比较发现新版本更大后更新配置信息。

This means that Sentinel guarantees a second liveness property: a set of Sentinels that are able to communicate will all converge to the same configuration with the higher version number.
这就意味着sentinel有第二种保活性，sentinel集合中各实例之间可以相互间通信来将更高的配置信息覆盖到所有实例。

Basically if the net is partitioned, every partition will converge to the higher local configuration. In the special case of no partitions, there is a single partition and every Sentinel will agree about the configuration.
基本上如果网络是被隔离的，每个分区都会被当前已知的最新配置覆盖。在一种特殊的无分区的情况下，也就是只有一个分区的时候，每个sentinel都会认同当前的配置。

## Consistency under partitions
## 网络分区故障下的一致性

Redis Sentinel configurations are eventually consistent, so every partition will converge to the higher configuration available. However in a real-world system using Sentinel there are three different players:
Redis sentinel配置信息最终都会达到一致，所以每个分区都会更新到当前可用的更高版本的配置信息。然而在一个使用sentinel的真实系统中，有三种不同的角色：

    Redis instances.
    redis实例
    Sentinel instances.
    sentinel实例
    Clients.
    客户端

In order to define the behavior of the system we have to consider all three.
为了定义系统的行为我们必须要考虑到这所有三种。
The following is a simple network where there are 3 nodes, each running a Redis instance, and a Sentinel instance:
以下是一个有三个节点的简单网络，每个节点都运行着一个redis实例和一个sentinel实例：

```
            +-------------+
            | Sentinel 1  |----- Client A
            | Redis 1 (M) |
            +-------------+
                    |
                    |
+-------------+     |          +------------+
| Sentinel 2  |-----+-- // ----| Sentinel 3 |----- Client B
| Redis 2 (S) |                | Redis 3 (M)|
+-------------+                +------------+
```

In this system the original state was that Redis 3 was the master, while Redis 1 and 2 were slaves. A partition occurred isolating the old master. Sentinels 1 and 2 started a failover promoting Sentinel 1 as the new master.
在这个系统的原始状态中，redis3 是master，redis 1和redis 2是slave。老的master被分区隔离在外，sentinel 1和sentinel 2开始发起故障转移来提升sentinel 1成为新的master

The Sentinel properties guarantee that Sentinel 1 and 2 now have the new configuration for the master. However Sentinel 3 has still the old configuration since it lives in a different partition.
sentinel 的特点保证了sentinel 1和sentinel 2现在有新的master的配置信息，然而sentinel 3还是只有旧的配置信息，因为它还在另一个不同的分区中。

We know that Sentinel 3 will get its configuration updated when the network partition will heal, however what happens during the partition if there are clients partitioned with the old master?
我们知道sentinel 3在网络分区愈合时将会更新它的配置信息，但是如果在这个分区中仍然还有客户端连接老的master将会发生什么？

Clients will be still able to write to Redis 3, the old master. When the partition will rejoin, Redis 3 will be turned into a slave of Redis 1, and all the data written during the partition will be lost.
客户端会仍然向redis 3写入，也就是老的master。当分区融合后，redis 3就会成为redis 1的slave，所有在网络分区阶段已经写入的数据都会丢失。

Depending on your configuration you may want or not that this scenario happens:
根据你的配置场景，你可能不在意也可能在意这种情况的发生：

    If you are using Redis as a cache, it could be handy that Client B is still able to write to the old master, even if its data will be lost.
    如果你把redis当做缓存使用，那么即使client B还在向老master写入数据然后数据丢了，这个仍然还是很好用的
    If you are using Redis as a store, this is not good and you need to configure the system in order to partially prevent this problem.
    如果你把redis当做存储使用，这样就会有问题，你需要对这个进行一些配置以防止这种问题的发生。

Since Redis is asynchronously replicated, there is no way to totally prevent data loss in this scenario, however you can bound the divergence between Redis 3 and Redis 1 using the following Redis configuration option:
由于redis是异步复制的，所以在这种情况下没有办法完全得防止数据的丢失，不过你可以限制Redis 3和Redis 1之间数据不一致的程度，你可以采用如下的配置项：

min-slaves-to-write 1
min-slaves-max-lag 10

With the above configuration (please see the self-commented redis.conf example in the Redis distribution for more information) a Redis instance, when acting as a master, will stop accepting writes if it can't write to at least 1 slave. Since replication is asynchronous not being able to write actually means that the slave is either disconnected, or is not sending us asynchronous acknowledges for more than the specified max-lag number of seconds.
在上述的配置下（请参照redis发行版中redis.conf中自注释的例子查看更过信息），作为master的redis实例如果没有slave就会停止接受写数据。因为复制是异步的，当发生这种情况使master不可写时，也就意味着slave要么已经断连，要么无法将min-slaves-max-lag以内的数据接收的确认消息发送出来。

Using this configuration the Redis 3 in the above example will become unavailable after 10 seconds. When the partition heals, the Sentinel 3 configuration will converge to the new one, and Client B will be able to fetch a valid configuration and continue.
在上述例子中使用旧配置信息的Redis 3将会在10s后不可用，当分区愈合后，sentinel 3的配置会被新的配置覆盖，client B将会获得一个新的配置继续运行。

In general Redis + Sentinel as a whole are a an eventually consistent system where the merge function is last failover wins, and the data from old masters are discarded to replicate the data of the current master, so there is always a window for losing acknowledged writes. This is due to Redis asynchronous replication and the discarding nature of the "virtual" merge function of the system. Note that this is not a limitation of Sentinel itself, and if you orchestrate the failover with a strongly consistent replicated state machine, the same properties will still apply. There are only two ways to avoid losing acknowledged writes:
在一般的redis+sentinel的这样一个最终一致性的整个系统中，合并功能会使最后一次故障转移成功，老master的数据都会被清空以便于从当前的新master复制，所以总是会有一个丢失已经确认写完成的数据的窗口期。这是由于redis的异步复制和系统的虚拟合并功能所导致的丢弃所引发的。注意这不是redis本身的局限导致的，如果你为故障转移功能使用一个精心设计的强一致性状态机，同样的问题还会发生。只有两种方法可以防止丢失已经确认写成功的数据：

    Use synchronous replication (and a proper consensus algorithm to run a replicated state machine).
    使用同步复制（和一个合适的一致性算法实现的复制状态机）
    Use an eventually consistent system where different versions of the same object can be merged.
    使用一个可以合并同一份数据的不同版本的强一致性系统。

Redis currently is not able to use any of the above systems, and is currently outside the development goals. However there are proxies implementing solution "2" on top of Redis stores such as SoundCloud Roshi, or Netflix Dynomite.
redis现在还不能使用上述的系统，而且也不在开发的目标中。然而也有一些中间代理系统在redis上层实现了方案2，比如soundCloud Roshi, Netflix Dynomite

## Sentinel persistent state
## Sentinel的持久化状态

Sentinel state is persisted in the sentinel configuration file. For example every time a new configuration is received, or created (leader Sentinels), for a master, the configuration is persisted on disk together with the configuration epoch. This means that it is safe to stop and restart Sentinel processes.
Sentinel状态是持久化在sentinel的配置文件中的，比如每隔一段时间收到或创建一个新的配置信息，配置信息就会和配置epoch值一起持久化到磁盘上.这就意味着我们可以安全得关闭和重启Sentinel进程了。

## TILT mode
## TILT模式

Redis Sentinel is heavily dependent on the computer time: for instance in order to understand if an instance is available it remembers the time of the latest successful reply to the PING command, and compares it with the current time to understand how old it is.
Redis Sentinel重度依赖于计算机时钟，比如为了了解一个实例是否是可用的，它需要记住最近一次成功得到PING命令的回复时间，然后和当前时间比较从而得知这个实例的新旧程度。

However if the computer time changes in an unexpected way, or if the computer is very busy, or the process blocked for some reason, Sentinel may start to behave in an unexpected way.
但是然而如果计算机时钟在一种无法预知的情况下被改变了，或者计算机非常繁忙，又或者进程由于某些原因被阻塞了，sentinel可能会以一种不可预知的方式运行下去。

The TILT mode is a special "protection" mode that a Sentinel can enter when something odd is detected that can lower the reliability of the system. The Sentinel timer interrupt is normally called 10 times per second, so we expect that more or less 100 milliseconds will elapse between two calls to the timer interrupt.
TILT模式是Sentinel的一种特殊的保护模式，它表明Sentinel遇到了一些奇怪的问题，可能会导致系统的可靠性降低。Sentinel的定时器中断在正常情况下每秒会被调起10次，所以我们认为在每两次时钟中断的函数调用之间差不多是100ms。

What a Sentinel does is to register the previous time the timer interrupt was called, and compare it with the current call: if the time difference is negative or unexpectedly big (2 seconds or more) the TILT mode is entered (or if it was already entered the exit from the TILT mode postponed).
Sentinel
sentinel所做的是将上次时钟中断调用的时间记录下来，并和当前的调用时间比较，如果时间间隔是负数或者是一个超出预料的很长的间隔（超过2s）,sentinel就会进入TILT模式（或者它已经在TILT模式了，就将退出期延后）
When in TILT mode the Sentinel will continue to monitor everything, but:
当Sentinel进入TILT模式时，Sentinel将会持续监控所有的东西，但是：

    It stops acting at all.
    它会停止所有的操作
    It starts to reply negatively to SENTINEL is-master-down-by-addr requests as the ability to detect a failure is no longer trusted.
    开始向SENTINEL is-master-down-by-addr 请求回复消极的消息，以此作为侦测不可信失败情况的能力。

If everything appears to be normal for 30 second, the TILT mode is exited.
如果所有的请求能够保持正常恢复超过30s后，就从TILT模式退出。

Note that in some way TILT mode could be replaced using the monotonic clock API that many kernels offer. However it is not still clear if this is a good solution since the current system avoids issues in case the process is just suspended or not executed by the scheduler for a long time.
注意在某些情况下TILT模式可以被单调时钟的API替代，很多内核都提供了这样的API。然而这是否是一个好的解决方案还不清楚，因为当前系统的实现所避免的问题只是进程挂起或者长时间没有被调度器调起的问题。

This website is open source software. See all credits. 