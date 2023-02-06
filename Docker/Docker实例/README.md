# 目录

- Docker-Redis集群部署
- Docker-打包运行微服务项目
- Docker-运行MySQL

# Redis集群部署

创建一个网卡
 
 ```
docker network create redis --subnet 172.38.0.0/16 
 ```

查看详细信息

```
docker network inspect redis
```

通过脚本创建六个redis配置

```shell
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
```

进入目录发现已经创建好六个节点

![[Pasted image 20230206101306.png]]

启动六个节点

```shell
# 脚本方式
docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \
```

```shell
# 独立创建方式
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf;

docker run -p 6372:6379 -p 16372:16379 --name redis-2 \
-v /mydata/redis/node-2/data:/data \
-v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf;

docker run -p 6373:6379 -p 16373:16379 --name redis-3 \
-v /mydata/redis/node-3/data:/data \
-v /mydata/redis/node-3/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.13 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf;

docker run -p 6374:6379 -p 16374:16379 --name redis-4 \
-v /mydata/redis/node-4/data:/data \
-v /mydata/redis/node-4/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.14 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; 

docker run -p 6375:6379 -p 16375:16379 --name redis-5 \
-v /mydata/redis/node-5/data:/data \
-v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.15 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; 

docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-5/data:/data \
-v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; 
```

创建完毕，查看运行着的六个节点

![[Pasted image 20230206104141.png]]

**开始创建集群**

进入一个容器

```
# 进入节点1
docker exec -it redis-1 /bin/sh
```

使用redis-cli，通过集群方式创建连接

```
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1

>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: 584418519b9a5d5de964be5afe1e668e0a3daaa3 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: bfba025b4ffcfa30089fb7ea90209b41f206fbd5 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: f35a0ae1e0bfe2810a506f34bfccf3998a196d1d 172.38.0.14:6379
   replicates 2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b
S: e2a1b51b693014c986b41a5ba24b57b3babbd8a5 172.38.0.15:6379
   replicates 584418519b9a5d5de964be5afe1e668e0a3daaa3
S: 9207d59d4e0a8cd8569c4a52e964a3275eb887f0 172.38.0.16:6379
   replicates bfba025b4ffcfa30089fb7ea90209b41f206fbd5
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 584418519b9a5d5de964be5afe1e668e0a3daaa3 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: f35a0ae1e0bfe2810a506f34bfccf3998a196d1d 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b
M: 2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: bfba025b4ffcfa30089fb7ea90209b41f206fbd5 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 9207d59d4e0a8cd8569c4a52e964a3275eb887f0 172.38.0.16:6379
   slots: (0 slots) slave
   replicates bfba025b4ffcfa30089fb7ea90209b41f206fbd5
S: e2a1b51b693014c986b41a5ba24b57b3babbd8a5 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 584418519b9a5d5de964be5afe1e668e0a3daaa3
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

集群创建完毕，测试

```
/data # redis-cli -c

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
cluster_stats_messages_ping_sent:333
cluster_stats_messages_pong_sent:340
cluster_stats_messages_sent:673
cluster_stats_messages_ping_received:335
cluster_stats_messages_pong_received:333
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:673

127.0.0.1:6379> cluster nodes

584418519b9a5d5de964be5afe1e668e0a3daaa3 172.38.0.11:6379@16379 myself,master - 0 1675652091000 1 connected 0-5460
f35a0ae1e0bfe2810a506f34bfccf3998a196d1d 172.38.0.14:6379@16379 slave 2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b 0 1675652092901 4 connected
2f292f13f9bf1a422ee30a3708cc7f9cf9b0ca5b 172.38.0.13:6379@16379 master - 0 1675652092000 3 connected 10923-16383
bfba025b4ffcfa30089fb7ea90209b41f206fbd5 172.38.0.12:6379@16379 master - 0 1675652092601 2 connected 5461-10922
9207d59d4e0a8cd8569c4a52e964a3275eb887f0 172.38.0.16:6379@16379 slave bfba025b4ffcfa30089fb7ea90209b41f206fbd5 0 1675652092601 6 connected
e2a1b51b693014c986b41a5ba24b57b3babbd8a5 172.38.0.15:6379@16379 slave 584418519b9a5d5de964be5afe1e668e0a3daaa3 0 1675652092000 5 connected
127.0.0.1:6379> 
```