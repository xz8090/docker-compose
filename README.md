# Bigdata-Docker构建大数据学习开发环境


### 介绍

##### 1、镜像环境

* 系统：centos 8
* Java ：java1.8
* Zookeeper: 3.4.6
* Hadoop: 2.7.1
* Hive: 1.2.1
* Spark: 1.6.2
* Hbase: 1.1.2

##### 2、镜像介绍


* centos-java：openssh、java8，基础镜像
* docker-zk:  基于centos-java构建，zookeeper，用于启动zk集群
* docker-hadoop：基于centos-java构建, hadoop，用于启动hadoop集群
* docker-hive：基于docker-hadoop镜像构建，包含hadoop、hive，用于启动hadoop、hive集群
* docker-spark：基于docker-hive镜像构建，包含hadoop、hive、spark，用于启动hadoo、hive、spark集群
* docker-hbase：基于docker-spark镜像构建，包含hadoop、hive、spark、hbase，用于启动hadoop、hive、spark、hbase集群



### Quick Start

#### 1、装载镜像,并重命名
##### 离线版
```
$ docker load < xxx.tar
$ docker tag [镜像ID] [repository]:[tag] 
```

可以根据需求注释掉不需要的镜像

##### 在线版，点击运行build.sh

#### 2、创建大数据集群网络

```
$ docker network create zoo
```

#### 3、启动zk集群

```
$ docker-compose -f docker-compose-zk.yml up -d
```

根据需要可在compose膜拜中增减集群数量，注意同时要增减myid配置

#### 4、大数据集群

##### a）启动Hadoop集群

```
$ docker-compose -f docker-compose-hadoop.yml up -d
```

启动集群，格式化namenode

```
$ docker exec -it hadoop-master bash
$ cd /usr/local/hadoop/bin
$ hdfs namenode -format
```

然后启动hdfs和yarn

```
$ cd /usr/local/hadoop/sbin
$ ./start-all.sh
```

访问http://localhost:50070，看集群是否启动成功
有可能50070端口无法使用，原因是hyper-v端口占用问题，需要修改hyper-v保留端口或者修改端口

##### b）启动Hive集群

需要依赖mysql容器

```
$ docker-compose -f docker-compose-hive.yml up -d
```

 启动hadoo集群的操作和上面启动hadoop集群一样

##### c）启动Spark集群

需要依赖mysql容器

```
$ docker-compose -f docker-compose-spark.yml up -d
```

 启动hadoop集群同a。

启动spark集群

```
$ sh /usr/local/spark/sbin/start-all.sh
```

使用 spark 自带样例中的计算 Pi 的应用来验证一下

```
/usr/local/spark/bin/spark-submit --master spark://hadoop-master:7077 --class org.apache.spark.examples.SparkPi /usr/local/spark/lib/spark-examples-1.6.2-hadoop2.2.0.jar 1000
```

计算结果输出如下

```
starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark--org.apache.spark.deploy.master.Master-1-1bdfd98bccc7.out
hadoop-slave2: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-9dd7e2ebbf13.out
hadoop-slave3: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-97a87730dd03.out
hadoop-slave1: starting org.apache.spark.deploy.worker.Worker, logging to /usr/local/spark/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-adb07707f15b.out
<k/bin/spark-submit --master spark://hadoop-master:7077 --class org.apache.spark.examples.SparkPi /usr/local/spark/li
lib/      licenses/
<.examples.SparkPi /usr/local/spark/lib/spark-examples-1.6.2-hadoop2.2.0.jar 1000
16/11/07 08:19:46 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Pi is roughly 3.1417756
```



##### d）启动Hbase集群

```
$ docker-compose -f docker-compose-hbase.yml up -d
```

启动hadoop、spark集群同c

启动hbase集群

```
$ sh /usr/local/hbase/bin/start-hbase.sh
```





注意docker-compose-hadoop.yml、docker-compose-hive.yml、docker-compose-spark.yml和docker-compose-hbase.yml不要一起启动，后面模板中是包含了前一个的所有配置