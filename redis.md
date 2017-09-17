yum install gcc-c++
tar zxf /usr/local/src/redis-3.0.0.tar.gz
cd /usr/local/src/redis-3.0.0
make
make install PREFIX=/usr/local/redis-cluster/1-node/master-6379
复制配置文件
cp /usr/local/src/redis-3.0.0/redis.conf /usr/local/redis-cluster/1-node/master-6379/
vim /usr/local/redis-cluster/1-node/master-6379/redis.conf
7 daemonize yes		#设置可后台启动
187 dir /data/redis/persist/ #修改RDB和AOF持久化文件位置(可选)
[root@redis master-6379]# chown -R redis.redis /data/redis
[root@redis master-6379]# chown -R redis.redis /data/redis/*
----开启持久化--
177 dbfilename dump.rdb #设置dbfilename指定rdb快照文件的名称
504 appendonly yes	#开启AOF持久化
508 appendfilename appendonly.aof	#默认的文件名是appendonly.aof，可以通过appendfilename参数修改
---开启可以集群----
632 cluster-enabled yes

==========创建主从复制=====================
创建1个slave
cp -ar /usr/local/redis-cluster/1-node/master-6379 /usr/local/redis-cluster/1-node/slave-6382

修改配置
vim /usr/local/redis-cluster/1-node/slave-6382/redis.conf
port 6382
slaveof 192.168.200.23 6379

编写启动脚本
vim /usr/local/redis-cluster/redis-up.sh
------
#!/bin/sh
cd /usr/local/redis-cluster/
./master-6379/bin/redis-server ./master-6379/redis.conf
./slave-6380/bin/redis-server ./slave-6382/redis.conf
注意：
主机一旦发生增删改操作，那么从机会将数据同步到从机中,从机不能执行写操作
===========创建集群==========================
再复制4台redis
mkdir /usr/local/redis-cluster/2-node
mkdir /usr/local/redis-cluster/3-node

cp -ar /usr/local/redis-cluster/1-node/master-6379 /usr/local/redis-cluster/2-node/master-6380
cp -ar /usr/local/redis-cluster/1-node/master-6379 /usr/local/redis-cluster/3-node/master-6381
cp -ar /usr/local/redis-cluster/1-node/slave-6382 /usr/local/redis-cluster/2-node/slave-6383
cp -ar /usr/local/redis-cluster/1-node/slave-6382 /usr/local/redis-cluster/3-node/slave-6384
主    	从
6379	6382
6380	6383
6381	6384
vim /usr/local/redis-cluster/2-node/master-6380/redis.conf
port 6380
vim /usr/local/redis-cluster/2-node/slave-6383/redis.conf
port 6383
slaveof 192.168.200.23 6380
vim /usr/local/redis-cluster/3-node/master-6381/redis.conf
port 6381
vim /usr/local/redis-cluster/3-node/slave-6384/redis.conf
port 6384
slaveof 192.168.200.23 6381

修改启动脚本
cd /usr/local/redis-cluster/1-node/master-6379 &&./bin/redis-server redis.conf
cd /usr/local/redis-cluster/2-node/master-6380 &&./bin/redis-server redis.conf
cd /usr/local/redis-cluster/3-node/master-6381 &&./bin/redis-server redis.conf
cd /usr/local/redis-cluster/1-node/slave-6382 &&./bin/redis-server redis.conf
cd /usr/local/redis-cluster/2-node/slave-6383 &&./bin/redis-server redis.conf
cd /usr/local/redis-cluster/3-node/slave-6384 &&./bin/redis-server redis.conf
拷贝集群配置
cp /usr/local/src/redis-3.0.0/src/redis-trib.rb /usr/local/redis-cluster/

yum install ruby
yum install rubygems
手动下载https://rubygems.global.ssl.fastly.net/gems/redis-3.2.1.gem
gem install redis-3.2.1.gem

./redis-trib.rb create --replicas 1 192.168.200.23:6379 192.168.200.23:6380 192.168.200.23:6381 192.168.200.23:6382 192.168.200.23:6383 192.168.200.23:6384

启动时发现
>>> 'slaveof 192.168.200.23 6379'
slaveof directive not allowed in cluster mode
slaveof不允许在cluster中使用

关闭:
/usr/local/redis-cluster/master-6379/bin/redis-cli shutdown
