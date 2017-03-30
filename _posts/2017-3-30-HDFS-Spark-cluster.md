---
layout:     post
title:      "HDFS Spark cluster搭建"
date:       2017-3-30
author:     "Ryan"
header-img: "img/post-bg-2017.jpg"
tags:
    - Hadoop
    - HDFS
    - Spark
---

* Hadoop version 2.7.3
* Spark  version 2.1

# HDFS cluster

## Step1: 创建3虚拟机节点, 1 namenode, 2 datanodes

`/etc/hosts` 配置如下:
```
192.168.220.184 datanode1
192.168.220.185 datanode2
192.168.220.171 master namenode
```
## Step2: SSH NOPASSWD(SSH无密码登录)

在３个虚拟机上安装 openssh, 并进行配置
```
ubuntu:$ sudo apt install openssh-server

namenode:$ ssh-keygen -t rsa -P ''
namenode:$ cat .ssh/id_rsa.pub >> .ssh/authorized_keys
namenode:$ scp .ssh/authorized_keys user@datanode1:/user
namenode:$ scp .ssh/authorized_keys user@datanode2:/user
```

## Step3: 在3个虚拟机上安装 Java 并设置环境变量

```
ubuntu:$ sudo apt install openjdk-8-jdk
```

在三个机器的 .bashrc 中进行如下设置
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=~/hadoop-2.7.3
export CLASSPATH=`${HADOOP_HOME}/bin/hadoop classpath --glob`
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin
```

## Step4: 下载并解压 hadoop 到 ~ (/home/user)

## Step5: 配置 hadoop

(1). $HADOOP_HOME/etc/hadoop/hadoop-env.sh

```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

(2). $HADOOP_HOME/etc/hadoop/slaves

```
datanode1
datanode2
```

(3). $HADOOP_HOME/etc/hadoop/core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master:9000</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>131072</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/user/hadoop-2.7.3/tmp</value>
    </property>
</configuration>
```

(4). $HADOOP_HOME/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/user/hadoop-2.7.3/hdfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/home/user/hadoop-2.7.3/hdfs/data</value>
    </property>
    <property>
        <name></name>
        <value></value>
    </property>
</configuration>
```

(5). 将 master 配置好的 $HADOOP_HOME 目录拷贝到其他 datanode 上

(6). Format namenode

```
namenode:$ hdfs namenode -format
```


(7). 启动 hdfs 的 namenode(master)

```
namenode:$ $HADOOP_HOME/sbin/start-dfs.sh
```

# Spark cluster

## Step 1: Download spark-2.1.0-bin-hadoop2.7.tgz and extract to ~

## Step 2: 安装 scala

```
ubuntu:$ sudo apt install scala
```

## Step 3: 配置 Spark

(1). ~/.bashrc

```
export SPARK_HOME=/home/shiy/spark-2.1.0-bin-hadoop2.7/
export PATH=$PATH:$SPARK_HOME/bin
```

(2). $SPARK_HOME/conf/spark-env.sh

```
export SCALA_HOME=/usr/share/scala-2.11
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export SPARK_MASTER_IP=master
export SPARK_WORKING_MEMORY=300M
export HADOOP_HOME=/home/shiy/hadoop-2.7.3
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

(3). $SPARK_HOME/conf/slaves

```
master
datanode1
datanode2
```

## Step 4: 将 master 配置好的 $SPARK_HOME 目录拷贝到其他 datanode 上

## Step 5: start spark

```
namenode:$ $SPARK_HOME/sbin/start-all.sh
```
