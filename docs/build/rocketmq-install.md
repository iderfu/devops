---
category: 常见服务的搭建
tag:
  - rocketmq
---



# rocketmq集群的搭建

环境

* JDK1.8
* maven3.6.0
* centos7.6主机两台

## 下载解压

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/rocketmq/4.6.0/rocketmq-all-4.6.0-bin-release.zip
unzip rocketmq-all-4.6.0-bin-release.zip
mv rocketmq-all-4.6.0-bin-release /usr/local/rocketmq
```

## 修改配置文件

rocketmq默认给出了三种建议配置模式
2m-2s-async(主从异步),----本文采用这种
2m-2s-sync(主从同步)
2m-noslave(仅master)

1. 对主机一（192.168.163.196）进行操作

```shell
cd /usr/local/rocketmq/conf/2m-2s-async
vim broker-a.properties
# 向配置文件中追加如下内容
namesrvAddr=192.168.163.196:9876;192.168.163.197:9876
listenPort=10911
storePathRootDir=/usr/local/rocketmq/master/store
storePathCommitLog=/usr/local/rocketmq/master/store/commitlog
storePathConsumeQueue=/usr/local/rocketmq/master/store/consumequeue
storePathIndex=/usr/local/rocketmq/master/store/index
storeCheckpoint=/usr/local/rocketmq/master/store/checkpoint
```

```shell
vim broker-b-s.properties
# 向配置文件中追加如下内容
namesrvAddr=192.168.163.196:9876;192.168.163.197:9876
listenPort=20911
storePathRootDir=/usr/local/rocketmq/slave/store
storePathCommitLog=/usr/local/rocketmq/slave/store/commitlog
storePathConsumeQueue=/usr/local/rocketmq/slave/store/consumequeue
storePathIndex=/usr/local/rocketmq/slave/store/index
storeCheckpoint=/usr/local/rocketmq/slave/store/checkpoint
```

2. 对主机二（192.168.163.197）进行操作

```shell
cd /usr/local/rocketmq/conf/2m-2s-async
vim broker-b.properties
# 向配置文件中追加如下内容
namesrvAddr=192.168.163.196:9876;192.168.163.197:9876
listenPort=10911
storePathRootDir=/usr/local/rocketmq/master/store
storePathCommitLog=/usr/local/rocketmq/master/store/commitlog
storePathConsumeQueue=/usr/local/rocketmq/master/store/consumequeue
storePathIndex=/usr/local/rocketmq/master/store/index
storeCheckpoint=/usr/local/rocketmq/master/store/checkpoint
```

```shell
vim broker-a-s.properties
# 向配置文件中追加如下内容
namesrvAddr=192.168.163.196:9876;192.168.163.197:9876
listenPort=20911
storePathRootDir=/usr/local/rocketmq/slave/store
storePathCommitLog=/usr/local/rocketmq/slave/store/commitlog
storePathConsumeQueue=/usr/local/rocketmq/slave/store/consumequeue
storePathIndex=/usr/local/rocketmq/slave/store/index
storeCheckpoint=/usr/local/rocketmq/slave/store/checkpoint
```

3. 设置Rocketmq运行时的JVM内存

```shell
cd /usr/local/rocketmq/bin
vim runbroker.sh
#JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
#改成：
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m"
```

```shell
vim runserver.sh 
# JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
# 改成
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

## 启动Rocketmq

1. 主机一上操作

```
nohup sh mqnamesrv &
nohup sh mqbroker -c ../conf/2m-2s-async/broker-a.properties &
nohup sh mqbroker -c ../conf/2m-2s-async/broker-b-s.properties &
```

2. 主机二上操作

```shell
nohup sh mqnamesrv &
nohup sh mqbroker -c ../conf/2m-2s-async/broker-a-s.properties &
nohup  sh mqbroker -c ../conf/2m-2s-async/broker-b.properties &
```

## 搭建Console可视化控制台

```shell
git clone https://github.com/apache/rocketmq-externals.git
cd rocketmq-externals/rocketmq-console/
vim src/main/resources/application.properties
# 添加两个namesvr的主机ip
rocketmq.config.namesrvAddr=192.168.163.196:9876;192.168.163.197:9876
```

```shell
# 打包
 mvn clean package -Dmaven.test.skip=true
 scp target/rocketmq-console-ng-1.0.1.jar 192.168.163.196:/home/ncar/service/webapps
```

```shell
# 启动jar包，在主机一（192.168.163.196）上
 nohup java -jar rocketmq-console-ng-1.0.1.jar &
```

打开控制台查看集群启动状态

```
http://192.168.163.196:8080/#/cluster
```

## 关闭命令

```
sh bin/mqshutdown broker
sh bin/mqshutdown namesrv
```

