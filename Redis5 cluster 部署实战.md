# 规划
redis-5.0.3  

127.0.0.1  6379  
127.0.0.1  6380  
127.0.0.1  6381  
127.0.0.1  7379  
127.0.0.1  7380  
127.0.0.1  7381 
# 系统参数优化
##### 最大队列长度，应付突发的大并发连接请求，默认为128
```
 vim /etc/sysctl.conf
 net.core.somaxconn = 20480
 sysctl -p
```
##### 半连接队列长度，此值受限于内存大小，默认为1024
```
 vim /etc/sysctl.conf
net.ipv4.tcp_max_syn_backlog = 20480
sysctl -p
```
##### 不能过量使用内存
```
 vim /etc/sysctl.conf
vm.overcommit_memory = 1
sysctl -p
```
##### 关闭Linux (THP) 透明内存
```
 echo never > /sys/kernel/mm/transparent_hugepage/enabled 
 #追加到文件
vim /etc/rc.local
 echo never > /sys/kernel/mm/transparent_hugepage/enabled 
```
##### 文件描述符
```
vim /etc/security/limits.conf 
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```
# 安装依赖组件
```
yum -y install wget gcc g cc-c++ make t ar openssl openssl-devel  zlib zlib-devel
```
# 下载编译
```
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar zxvf redis-5.0.3.tar.gz
cd redis-5.0.3
make
```

# 安装redis
```
mkdir -p /usr/local/redis/bin 
mkdir -p /usr/local/redis/6379
mkdir -p /usr/local/redis/6380
mkdir -p /usr/local/redis/6381
mkdir -p /usr/local/redis/7379
mkdir -p /usr/local/redis/7380
mkdir -p /usr/local/redis/7381

cp -a ./redis-5.0.3/src/redis-* /usr/local/redis/bin/
rm -rf /usr/local/redis/bin/redis-*.c
rm -rf /usr/local/redis/bin/redis-*.o
ln -s /usr/local/redis/bin/redis-cli /usr/local/bin/
```
# 配置文件配置
```
cp -a ./redis-5.0.3/redis.conf /usr/local/redis/6379/
mv  /usr/local/redis/6379/redis.conf /usr/local/redis/6379/redis.conf_bak
cat /usr/local/redis/6379/redis.conf_bak |grep -v '^#' |grep -v '^$' > /usr/local/redis/6379/redis.conf

#配置文件里需要修改的内容如下：
bind 127.0.0.1 192.168.0.13
daemonize yes
pidfile "/usr/local/redis/6379/redis.pid"
logfile "/usr/local/redis/6379/redis.log"
dir "/usr/local/redis/6379"
cluster-enabled yes
cluster-config-file "nodes.conf"
cluster-node-timeout 5000


\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/6380/
\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/6381/
\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/7379/
\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/7380/
\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/7381/

sed 's#6379#6380#g' /usr/local/redis/6380/redis.conf -i
sed 's#6379#6381#g' /usr/local/redis/6381/redis.conf -i
sed 's#6379#7379#g' /usr/local/redis/7379/redis.conf -i
sed 's#6379#7380#g' /usr/local/redis/7380/redis.conf -i
sed 's#6379#7381#g' /usr/local/redis/7381/redis.conf -i

```
# 启动redis
```
/usr/local/redis/bin/redis-server  /usr/local/redis/6379/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/6380/redis.conf 
/usr/local/redis/bin/redis-server  /usr/local/redis/6381/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/7379/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/7380/redis.conf 
/usr/local/redis/bin/redis-server  /usr/local/redis/7381/redis.conf
```
# 检查启动结果
```
[root@m redis]# ps -ef |grep redis 
root      9869     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6379 [cluster]
root      9871     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6380 [cluster]
root      9876     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6381 [cluster]
root      9878     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7379 [cluster]
root      9886     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7380 [cluster]
root      9895     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7381 [cluster]
root      9903  6052  0 12:23 pts/0    00:00:00 grep --color=auto redis
```
# 配置集群
redis5.0不再需要ruby
```
[root@m redis]# redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:7379 127.0.0.1:7380 127.0.0.1:7381 --cluster-replicas 1
* --cluster-replicas 1 表示每个master的副本数是1个 * 
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7379 to 127.0.0.1:6379
Adding replica 127.0.0.1:7380 to 127.0.0.1:6380
Adding replica 127.0.0.1:7381 to 127.0.0.1:6381
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 2a29e2b632f0df6659b990a8d6a00019be98c7fc 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
M: 2dbbd0821215a8f77f809e54fd3ae4e71fa8c674 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
M: 621647f3952902c4994c4add1c6641ee250546cc 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
S: 42ba2b269886c709891e4b67687b12ec1850bc9c 127.0.0.1:7379
   replicates 2a29e2b632f0df6659b990a8d6a00019be98c7fc
S: 3ff7e957606c334f388b2f7b64f24bab2a817cee 127.0.0.1:7380
   replicates 2dbbd0821215a8f77f809e54fd3ae4e71fa8c674
S: bb77cfa65f487c57668a2b5992461e22de0d3e39 127.0.0.1:7381
   replicates 621647f3952902c4994c4add1c6641ee250546cc
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: 2a29e2b632f0df6659b990a8d6a00019be98c7fc 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: bb77cfa65f487c57668a2b5992461e22de0d3e39 127.0.0.1:7381
   slots: (0 slots) slave
   replicates 621647f3952902c4994c4add1c6641ee250546cc
S: 42ba2b269886c709891e4b67687b12ec1850bc9c 127.0.0.1:7379
   slots: (0 slots) slave
   replicates 2a29e2b632f0df6659b990a8d6a00019be98c7fc
S: 3ff7e957606c334f388b2f7b64f24bab2a817cee 127.0.0.1:7380
   slots: (0 slots) slave
   replicates 2dbbd0821215a8f77f809e54fd3ae4e71fa8c674
M: 2dbbd0821215a8f77f809e54fd3ae4e71fa8c674 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 621647f3952902c4994c4add1c6641ee250546cc 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```
# 登录查看集群状态
```
[root@m redis]# redis-cli -p 6379 -c
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:221
cluster_stats_messages_pong_sent:230
cluster_stats_messages_sent:451
cluster_stats_messages_ping_received:225
cluster_stats_messages_pong_received:221
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:451

127.0.0.1:6379> cluster nodes
bb77cfa65f487c57668a2b5992461e22de0d3e39 127.0.0.1:7381@17381 slave 621647f3952902c4994c4add1c6641ee250546cc 0 1610771255000 6 connected
2a29e2b632f0df6659b990a8d6a00019be98c7fc 127.0.0.1:6379@16379 myself,master - 0 1610771252000 1 connected 0-5460
42ba2b269886c709891e4b67687b12ec1850bc9c 127.0.0.1:7379@17379 slave 2a29e2b632f0df6659b990a8d6a00019be98c7fc 0 1610771254275 4 connected
3ff7e957606c334f388b2f7b64f24bab2a817cee 127.0.0.1:7380@17380 slave 2dbbd0821215a8f77f809e54fd3ae4e71fa8c674 0 1610771255277 5 connected
2dbbd0821215a8f77f809e54fd3ae4e71fa8c674 127.0.0.1:6380@16380 master - 0 1610771254576 2 connected 5461-10922
621647f3952902c4994c4add1c6641ee250546cc 127.0.0.1:6381@16381 master - 0 1610771255000 3 connected 10923-16383
127.0.0.1:6379> 
```
# 开启权限认证
```
echo 'masterauth "123456"'  >> /usr/local/redis/6379/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/6379/redis.conf 

echo 'masterauth "123456"'  >> /usr/local/redis/6380/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/6380/redis.conf 

echo 'masterauth "123456"'  >> /usr/local/redis/6381/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/6381/redis.conf 

echo 'masterauth "123456"'  >> /usr/local/redis/7379/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/7379/redis.conf 

echo 'masterauth "123456"'  >> /usr/local/redis/7380/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/7380/redis.conf 

echo 'masterauth "123456"'  >> /usr/local/redis/7381/redis.conf 
echo 'requirepass "123456"'  >> /usr/local/redis/7381/redis.conf 
```
# 启动redis
```
/usr/local/redis/bin/redis-server  /usr/local/redis/6379/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/6380/redis.conf 
/usr/local/redis/bin/redis-server  /usr/local/redis/6381/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/7379/redis.conf
/usr/local/redis/bin/redis-server  /usr/local/redis/7380/redis.conf 
/usr/local/redis/bin/redis-server  /usr/local/redis/7381/redis.conf
```
# 检查启动结果
```
[root@m redis]# ps -ef |grep redis 
root      9869     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6379 [cluster]
root      9871     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6380 [cluster]
root      9876     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6381 [cluster]
root      9878     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7379 [cluster]
root      9886     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7380 [cluster]
root      9895     1  0 12:23 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7381 [cluster]
root      9903  6052  0 12:23 pts/0    00:00:00 grep --color=auto redis
```
# 登录验证
```
[root@m redis]# redis-cli -p 6380 -c
127.0.0.1:6380> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6380> auth 123456
OK
127.0.0.1:6380> keys *
1) "name"
127.0.0.1:6380> exit
```
# 破坏性测试


经测试允许死3个slave或者死两个master，必须保证集群里有3个master是正常的；生产环境会分机器部署，应该测试一台服务器宕机后，集群是否可以正常读写。

以下为测试杀死两个master进程的结果
##### python3连接redis cluster的代码
```
#/usr/local/bin/pip3 install redis-py-cluster

from rediscluster import RedisCluster
import sys
redis_nodes = [{'host': '127.0.0.1', 'port': 6379},
                    {'host': '127.0.0.1', 'port': 6380},
                    {'host': '127.0.0.1', 'port': 6381},
                    {'host': '127.0.0.1', 'port': 7379},
                    {'host': '127.0.0.1', 'port': 7380},
                    {'host': '127.0.0.1', 'port': 7381},
                    ]
pwd='123456'
try:
    rc=RedisCluster(startup_nodes=redis_nodes,decode_responses=True, password=pwd)
    rc.set("foo", "bar")
    print('connect successful!! value is %s' %(rc.get("foo")))  	
except Exception  as e:
    print("redis cluster 连接失败%s！！" %(e))

```
##### 正常情况下代码运行结果
```
[root@m redis]# /usr/local/bin/python3 conn.py
connect successful!! value is bar
```
##### 正常情况下redis cluster的状态
```
[root@m redis]# redis-cli -p 6379 -c
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> cluster nodes
37d215da83d1556a88fc5ec7c3ac14487b8f74b0 127.0.0.1:7379@17379 slave 912d65f779d1222533f7a4267cecaa4fc12eba78 0 1610774083000 4 connected
936d755c147661c3e320b561637c33675ddcbca5 127.0.0.1:6381@16381 master - 0 1610774083835 3 connected 10923-16383
0519961878940d733befb7161b5ec4e959984344 127.0.0.1:6380@16380 slave bdc67b82cc279e5edfa8e6def5376f1cf5523628 0 1610774084836 8 connected
b45a8d4927daf78d60fa8f9fd01c7248572b3b9d 127.0.0.1:7381@17381 slave 936d755c147661c3e320b561637c33675ddcbca5 0 1610774083534 6 connected
bdc67b82cc279e5edfa8e6def5376f1cf5523628 127.0.0.1:7380@17380 master - 0 1610774084000 8 connected 5461-10922
912d65f779d1222533f7a4267cecaa4fc12eba78 127.0.0.1:6379@16379 myself,master - 0 1610774082000 1 connected 0-5460
127.0.0.1:6379> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:8
cluster_my_epoch:1
cluster_stats_messages_ping_sent:70
cluster_stats_messages_pong_sent:54
cluster_stats_messages_sent:124
cluster_stats_messages_ping_received:54
cluster_stats_messages_pong_received:56
cluster_stats_messages_received:110
```
##### 杀死7380(master)的进程
```
[root@m redis]# ps -ef |grep 7380
root     12895     1  0 13:14 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:7380 [cluster]
root     12948  6052  0 13:15 pts/0    00:00:00 grep --color=auto 7380
[root@m redis]# kill -9 12895
[root@m redis]# ps -ef |grep 7380
root     12956  6052  0 13:15 pts/0    00:00:00 grep --color=auto 7380

[root@m redis]# redis-cli -p 6379 -c
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> cluster info
cluster_state:ok #集群状态依然是OK
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:9
cluster_my_epoch:1
cluster_stats_messages_ping_sent:222
cluster_stats_messages_pong_sent:115
cluster_stats_messages_fail_sent:4
cluster_stats_messages_auth-ack_sent:1
cluster_stats_messages_sent:342
cluster_stats_messages_ping_received:115
cluster_stats_messages_pong_received:113
cluster_stats_messages_fail_received:1
cluster_stats_messages_auth-req_received:1
cluster_stats_messages_received:230
#master仍然有三个
127.0.0.1:6379> cluster nodes
37d215da83d1556a88fc5ec7c3ac14487b8f74b0 127.0.0.1:7379@17379 slave 912d65f779d1222533f7a4267cecaa4fc12eba78 0 1610774125510 4 connected
936d755c147661c3e320b561637c33675ddcbca5 127.0.0.1:6381@16381 master - 0 1610774125000 3 connected 10923-16383
0519961878940d733befb7161b5ec4e959984344 127.0.0.1:6380@16380 master - 0 1610774125911 9 connected 5461-10922
b45a8d4927daf78d60fa8f9fd01c7248572b3b9d 127.0.0.1:7381@17381 slave 936d755c147661c3e320b561637c33675ddcbca5 0 1610774125000 6 connected
bdc67b82cc279e5edfa8e6def5376f1cf5523628 127.0.0.1:7380@17380 master,fail - 1610774112983 1610774111000 8 disconnected
912d65f779d1222533f7a4267cecaa4fc12eba78 127.0.0.1:6379@16379 myself,master - 0 1610774124000 1 connected 0-5460
127.0.0.1:6379> exit
# 脚本依然可以执行
[root@m redis]# /usr/local/bin/python3 conn.py
connect successful!! value is bar
```
##### 杀死6381(master)进程
```
[root@m redis]# ps -ef |grep 6381
root     12888     1  0 13:14 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6381 [cluster]
root     12993  6052  0 13:15 pts/0    00:00:00 grep --color=auto 6381
[root@m redis]# kill -9 12888
[root@m redis]# ps -ef |grep 6381
root     13002  6052  0 13:16 pts/0    00:00:00 grep --color=auto 6381

[root@m redis]# redis-cli -p 6379 -c
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> cluster info
cluster_state:ok #集群状态依然是ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:10
cluster_my_epoch:1
cluster_stats_messages_ping_sent:872
cluster_stats_messages_pong_sent:181
cluster_stats_messages_fail_sent:7
cluster_stats_messages_auth-ack_sent:2
cluster_stats_messages_sent:1062
cluster_stats_messages_ping_received:181
cluster_stats_messages_pong_received:189
cluster_stats_messages_fail_received:2
cluster_stats_messages_auth-req_received:2
cluster_stats_messages_received:374
# 仍然有3个master
127.0.0.1:6379> cluster nodes
37d215da83d1556a88fc5ec7c3ac14487b8f74b0 127.0.0.1:7379@17379 slave 912d65f779d1222533f7a4267cecaa4fc12eba78 0 1610774171005 4 connected
936d755c147661c3e320b561637c33675ddcbca5 127.0.0.1:6381@16381 master,fail - 1610774159481 1610774158580 3 disconnected
0519961878940d733befb7161b5ec4e959984344 127.0.0.1:6380@16380 master - 0 1610774172008 9 connected 5461-10922
b45a8d4927daf78d60fa8f9fd01c7248572b3b9d 127.0.0.1:7381@17381 master - 0 1610774172509 10 connected 10923-16383
bdc67b82cc279e5edfa8e6def5376f1cf5523628 127.0.0.1:7380@17380 master,fail - 1610774112983 1610774111000 8 disconnected
912d65f779d1222533f7a4267cecaa4fc12eba78 127.0.0.1:6379@16379 myself,master - 0 1610774170000 1 connected 0-5460
127.0.0.1:6379> exit
# 脚本依然可以执行
[root@m redis]# /usr/local/bin/python3 conn.py
connect successful!! value is bar
```
##### 杀死6380(master)进程
```
[root@m redis]# ps -ef |grep 6380
root     12883     1  0 13:14 ?        00:00:00 /usr/local/redis/bin/redis-server 127.0.0.1:6380 [cluster]
root     13044  6052  0 13:16 pts/0    00:00:00 grep --color=auto 6380
[root@m redis]# kill -9 12883
[root@m redis]# ps -ef |grep 6380
root     13052  6052  0 13:16 pts/0    00:00:00 grep --color=auto 6380
[root@m redis]# redis-cli -p 6379 -c
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> cluster info
cluster_state:fail #集群状态失败
cluster_slots_assigned:16384
cluster_slots_ok:10922
cluster_slots_pfail:0
cluster_slots_fail:5462
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:10
cluster_my_epoch:1
cluster_stats_messages_ping_sent:2251
cluster_stats_messages_pong_sent:260
cluster_stats_messages_fail_sent:9
cluster_stats_messages_auth-ack_sent:2
cluster_stats_messages_sent:2522
cluster_stats_messages_ping_received:260
cluster_stats_messages_pong_received:264
cluster_stats_messages_fail_received:3
cluster_stats_messages_auth-req_received:2
cluster_stats_messages_received:529
# 只剩两个master
127.0.0.1:6379> cluster nodes
37d215da83d1556a88fc5ec7c3ac14487b8f74b0 127.0.0.1:7379@17379 slave 912d65f779d1222533f7a4267cecaa4fc12eba78 0 1610774222130 4 connected
936d755c147661c3e320b561637c33675ddcbca5 127.0.0.1:6381@16381 master,fail - 1610774159481 1610774158580 3 disconnected
0519961878940d733befb7161b5ec4e959984344 127.0.0.1:6380@16380 master,fail - 1610774214007 1610774213105 9 disconnected 5461-10922
b45a8d4927daf78d60fa8f9fd01c7248572b3b9d 127.0.0.1:7381@17381 master - 0 1610774223133 10 connected 10923-16383
bdc67b82cc279e5edfa8e6def5376f1cf5523628 127.0.0.1:7380@17380 master,fail - 1610774112983 1610774111000 8 disconnected
912d65f779d1222533f7a4267cecaa4fc12eba78 127.0.0.1:6379@16379 myself,master - 0 1610774221000 1 connected 0-5460

#脚本无法连接集群
[root@m redis]# /usr/local/bin/python3 conn.py
redis cluster 连接失败CLUSTERDOWN error. Unable to rebuild the cluster！！
```
# 集群扩容
扩容一个主节点端口为8000到集群，并给8000这个主节点添加一个从节点端口为9000
##### 部署redis
```
mkdir -p /usr/local/redis/8000
mkdir -p /usr/local/redis/9000

\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/8000/
\cp -a /usr/local/redis/6379/redis.conf /usr/local/redis/9000/


sed 's#6379#8000#g' /usr/local/redis/8000/redis.conf -i
sed 's#6379#9000#g' /usr/local/redis/9000/redis.conf -i

/usr/local/redis/bin/redis-server /usr/local/redis/8000/redis.conf
/usr/local/redis/bin/redis-server /usr/local/redis/9000/redis.conf
```
##### 添加端口为8000的redis进集群
```
[root@m redis]# redis-cli -a 123456   --cluster add-node 127.0.0.1:8000 127.0.0.1:6379
>>> Adding node 127.0.0.1:8000 to cluster 127.0.0.1:6379
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: e4bc25db2cf943e7672c993aec26e1a4dfad3379 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 88b34f2a6b6d489c54b695826f9a40c4f358159c 127.0.0.1:7381
   slots: (0 slots) slave
   replicates ef886b4f51409f7c16cbff8997fa1ff7a083a961
S: ef80024f0897fac54d0fed515002ce94cf88c0ae 127.0.0.1:7380
   slots: (0 slots) slave
   replicates e4bc25db2cf943e7672c993aec26e1a4dfad3379
M: fde151e2a515b8b78d5cd51b72cf2fbc34b6299f 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: ef886b4f51409f7c16cbff8997fa1ff7a083a961 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: ebfb65a3408a1b3d3b76a0d96f8f6ce11e0e5f49 127.0.0.1:7379
   slots: (0 slots) slave
   replicates fde151e2a515b8b78d5cd51b72cf2fbc34b6299f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:8000 to make it join the cluster.
[OK] New node added correctly.
```
目前端口为8000的redis上的slots还是0，尚未分配slots
##### 集群重新分配slots
```
[root@m redis]# redis-cli --cluster reshard 127.0.0.1:6379 -a 123456
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: e4bc25db2cf943e7672c993aec26e1a4dfad3379 127.0.0.1:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 88b34f2a6b6d489c54b695826f9a40c4f358159c 127.0.0.1:7381
   slots: (0 slots) slave
   replicates ef886b4f51409f7c16cbff8997fa1ff7a083a961
S: ef80024f0897fac54d0fed515002ce94cf88c0ae 127.0.0.1:7380
   slots: (0 slots) slave
   replicates e4bc25db2cf943e7672c993aec26e1a4dfad3379
M: fde151e2a515b8b78d5cd51b72cf2fbc34b6299f 127.0.0.1:6381
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: a7e17d72b338082f0530be3466546930864f4051 127.0.0.1:8000
   slots: (0 slots) master
M: ef886b4f51409f7c16cbff8997fa1ff7a083a961 127.0.0.1:6380
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: ebfb65a3408a1b3d3b76a0d96f8f6ce11e0e5f49 127.0.0.1:7379
   slots: (0 slots) slave
   replicates fde151e2a515b8b78d5cd51b72cf2fbc34b6299f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 4096 ##加上8000这个redis，目前集群里就有4个master，slots一共为16384，均分的话每个master上应该有4096个solts
What is the receiving node ID? a7e17d72b338082f0530be3466546930864f4051
#这里填写新加入的redis 8000的id
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1: all  #这里选择all
...
    Moving slot 12286 from fde151e2a515b8b78d5cd51b72cf2fbc34b6299f
    Moving slot 12287 from fde151e2a515b8b78d5cd51b72cf2fbc34b6299f
Do you want to proceed with the proposed reshard plan (yes/no)? yes
...
Moving slot 12285 from 127.0.0.1:6381 to 127.0.0.1:8000: 
Moving slot 12286 from 127.0.0.1:6381 to 127.0.0.1:8000: 
Moving slot 12287 from 127.0.0.1:6381 to 127.0.0.1:8000:
```
##### 将端口为9000的redis加入集群
```
[root@m 9000]# redis-cli -a 123456   --cluster add-node 127.0.0.1:9000 127.0.0.1:6379
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Adding node 127.0.0.1:9000 to cluster 127.0.0.1:6379
>>> Performing Cluster Check (using node 127.0.0.1:6379)
M: e4bc25db2cf943e7672c993aec26e1a4dfad3379 127.0.0.1:6379
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 88b34f2a6b6d489c54b695826f9a40c4f358159c 127.0.0.1:7381
   slots: (0 slots) slave
   replicates ef886b4f51409f7c16cbff8997fa1ff7a083a961
S: ef80024f0897fac54d0fed515002ce94cf88c0ae 127.0.0.1:7380
   slots: (0 slots) slave
   replicates e4bc25db2cf943e7672c993aec26e1a4dfad3379
M: fde151e2a515b8b78d5cd51b72cf2fbc34b6299f 127.0.0.1:6381
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
M: a7e17d72b338082f0530be3466546930864f4051 127.0.0.1:8000
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
M: ef886b4f51409f7c16cbff8997fa1ff7a083a961 127.0.0.1:6380
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: ebfb65a3408a1b3d3b76a0d96f8f6ce11e0e5f49 127.0.0.1:7379
   slots: (0 slots) slave
   replicates fde151e2a515b8b78d5cd51b72cf2fbc34b6299f
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 127.0.0.1:9000 to make it join the cluster.
[OK] New node added correctly.

```
##### 登录9000redis设置自己的主节点为8000
```
[root@m 9000]# redis-cli -p 9000 -a 123456
127.0.0.1:9000> cluster replicate a7e17d72b338082f0530be3466546930864f4051
OK 
#节点id是8000的节点id，主节点id
127.0.0.1:9000> cluster nodes
fde151e2a515b8b78d5cd51b72cf2fbc34b6299f 127.0.0.1:6381@16381 master - 0 1611027327820 3 connected 12288-16383
e4bc25db2cf943e7672c993aec26e1a4dfad3379 127.0.0.1:6379@16379 master - 0 1611027326000 1 connected 1365-5460
ebfb65a3408a1b3d3b76a0d96f8f6ce11e0e5f49 127.0.0.1:7379@17379 slave fde151e2a515b8b78d5cd51b72cf2fbc34b6299f 0 1611027327000 3 connected
a7e17d72b338082f0530be3466546930864f4051 127.0.0.1:8000@18000 master - 0 1611027326000 7 connected 0-1364 5461-6826 10923-12287
ef80024f0897fac54d0fed515002ce94cf88c0ae 127.0.0.1:7380@17380 slave e4bc25db2cf943e7672c993aec26e1a4dfad3379 0 1611027327620 1 connected
ef886b4f51409f7c16cbff8997fa1ff7a083a961 127.0.0.1:6380@16380 master - 0 1611027328121 2 connected 6827-10922
88b34f2a6b6d489c54b695826f9a40c4f358159c 127.0.0.1:7381@17381 slave ef886b4f51409f7c16cbff8997fa1ff7a083a961 0 1611027327019 2 connected
96b31b06037c3541cba3f5e90504ffdd56e71b92 127.0.0.1:9000@19000 myself,slave a7e17d72b338082f0530be3466546930864f4051 0 1611027326000 0 connected
127.0.0.1:9000> 
```