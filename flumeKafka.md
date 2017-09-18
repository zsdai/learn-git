# flume
## 修改网卡，设置IP地址  
vim /etc/sysconfig/network-scripts/ifcfg-ens33    
`IPADDR="192.168.200.24"`  
`NETMASK="255.255.255.0"`  
`GATEWAY="192.168.200.1"`    
## 设置住名称
hostnamectl set-hostname flumeKafka    
## 设置代理上网  
vim /etc/profile  
`http_proxy=http://172.17.18.80:8080`  
`export http_proxy`  
source /etc/profile
测试:
yum install tree  
## 安装JDK
下载或拷贝JDK  
scp jdk-8u144-linux-x64.tar.gz root@192.168.200.24:/usr/local/src/  
tar zxvf /usr/local/src/jdk-8u144-linux-x64.tar.gz -C /usr/local/  
ln -s /usr/local/jdk1.8.0_144 /usr/local/jdk  
=##################环境变量配置#############  
`export JAVA_HOME=/usr/local/jdk`  
`export JRE_HOME=$JAVA_HOME/jre`  
`export CLASSPATH=$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH`  
`export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH`  
source /etc/profile  
java -version  
## 下载flume
wget http://mirrors.hust.edu.cn/apache/flume/1.6.0/apache-flume-1.6.0-bin.tar.gz  
tar zxf /usr/local/src/apache-flume-1.6.0-bin.tar.gz -C /usr/local/  
ln -s /usr/local/apache-flume-1.6.0-bin/ /usr/local/flume  
拷贝配置模板  
cp /usr/local/flume/conf/flume-conf.properties.template /usr/local/flume/conf/flume-conf.properties  
vim /usr/local/flume/conf/flume-conf.properties  
`#此配置文件需要定义sources、channels和sinks`  
`#这种定义了sources、channels和sinks的被称为一个agent，每一个agent都会定义这3个属性`  
`agent.sources = r1`  
`agent.channels = c1`  
`agent.sinks = s1`  
`# For each one of the sources, the type is defined`  
`#对于sources中的每一个，都定义了类型。`  
`agent.sources.r1.type = netcat`  
`agent.sources.r1.bind = 192.168.200.24`  
`agent.sources.r1.port = 8888`  
`# The channel can be defined as follows.`  
`#channel可以按如下定义`  
`agent.sources.r1.channels = c1`  
`# Each sink's type must be defined`  
`# 每一个sink的类型必须被定义`  
`agent.sinks.s1.type = file_roll`  
`agent.sinks.s1.sink.directory = /tmp/log/flume`  
`#Specify the channel the sink should use`  
`#指定sink使用的channel`  
`agent.sinks.s1.channel = c1`  
`# Each channel's type is defined.`  
`#每一个channel的类型被定义`  
`agent.channels.c1.type = memory`  
`# Other config values specific to each type of channel(sink or source)`  
`# can be defined as well`  
`# In this case, it specifies the capacity of the memory channel`  
`#也可以指定sink或source的其他配置属性的值`  
`agent.channels.c1.capacity = 100`  
## 启动测试
1. 建立输出目录  
mkdir -p /tmp/log/flume  
2. 启动服务  
cd /usr/local/flume/  
bin/flume-ng agent --conf conf -f conf/flume-conf.properties -n agent&  
3. 发送数据  
telnet 192.168.200.24 8888 
输入  
hello world!  
hello Flume!  
4. 查看数据文件 查看 /tmp/log/flume 目录文件:
5. 文件名不同，需要根据实际情况修改和查看  
cat /tmp/log/flume/1447671188760-2  
hello world!  
hello Flume!  
# kafka
## 安装kafka
wget http://mirrors.cnnic.cn/apache/kafka/0.10.0.1/kafka_2.10-0.10.0.1.tgz  
tar zxf /usr/local/src/kafka_2.10-0.10.0.1.tgz -C /usr/local/  
ln -s /usr/local/kafka_2.10-0.10.0.1 /usr/local/kafka  
## 启动

