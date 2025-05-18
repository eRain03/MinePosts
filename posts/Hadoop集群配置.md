---
title: "Hadoop集群配置" #标题
date: 2024-03-19T08:45:15+08:00 #创建时间
lastmod: 2024-03-19T08:45:15+08:00 #更新时间
author: ["未淼"] #作者
tags: 
description: "" #描述
weight:          # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: false #是否展示评论
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径

---
# 先前条件

| Node1 | Node2 | Node3 |
| ---- | ---- | ---- |
| 192.168.1.39 | 192.168.1.9 | 192.168.1.40 |

## 关闭防火墙
```

 停止防火墙进程
systemctl stop firewalld.service

禁用防火墙开机启动
systemctl disable firewalld.service

对三个节点分别修改主机名
hostnamectl set-hostname Node{1,2,3}

```

## 安装必要软件环境

```
yum install -y java-1.8.0-openjdk vim 
#安装jdk8和vim等必备软件
```

## 修改Hosts文件定向主机IP

```
vim /etc/hosts

node1 192.168.1.39
node2 192.168.1.9
node3 192.168.1.40


ping node1
ping node2
ping node3
测试连通性

```

## 给Node123添加hadoop用户并赋予sudo权限


```
Node123全部要做

[root@node2 ~]# useradd hadoop
[root@node2 ~]# # 设置密码需要手动输入两次密码，笔者这里也暂时设定密码为hadoop
[root@node2 ~]# passwd hadoop
更改用户 hadoop 的密码 。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
[root@node2 ~]# mkdir -p /data/hadoop
[root@node2 ~]# chown hadoop:hadoop /data/hadoop

chmod u+w /etc/sudoers
vim /etc/sudoers
# 在sudoers文件的root用户一行后面添加下面内容并且保存
hadoop ALL=(ALL) NOPASSWD:ALL 
chmod u-w /etc/sudoers
sudo chmod -R a+w /data/hadoop
```

## 生成RSA密钥SSH免密登录

```
Node123全部要做

su hadoop
ssh-keygen -t rsa
##按几次回车
cd /home/hadoop/.ssh/
cat id_rsa.pub >> authorized_keys
vim authorized_keys
##把其他主机的添加进去
最后每个节点的authorized_keys内容都应该是这样的：
```

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDTwzdM+cmFy1lO4bNpD1T5+EO0FNvBBymf3B/Pdd92/Dq3Z3cTr+XHL9HWrIZPxDoWCm7OQY9Ek3oCJgjhlnxowQbtHp8GUKmmJma19NStBNHrO8T3wpbypOgUB7xJQZclpPpxrkkiMbGEU5MV3wJxFPEA2Sd3b7sdYx2YW/hMU94UabbzV4aJjtsG9GHHx4yq9e9/P31jnidhf013F7t9lIBdCPlgHj7+feliVhjSM1jbUt3CFDsskkFXMKuzVHP6B/ipviFtLt6SA3AgDIvFFD3CYV2KgiCd7JKqR/a+8765ML9auSubf7JegDUBvVXDt+fPOVc1Lm37nCyhGLjX hadoop@node1
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD3arpeN86cemzQPdz86oQV+gKk3A9MTpNAvFNl8QLfiYft8Hscsp3oQkdyWd+vnu/VxUa7URx6iOAfaTluYOlyY18A41wnAkE9xP3861Mak8SmvPPxsj8wsC8KugBk2uGVeOHjHyMQW0Qcd/Qe/2jnqsbDJsuK7ZyxOzEHbMaIb3IRIo6x1WVUv6x4oX9mJkgZw066Wzu0nNAanDDKDmP+Ncz1OcnGoYd+B+YUqeXtPJ7acQtBoNO/2Hp35G5CK5j5VH0Yj5HKUyy1t3jv+PuSc2HVMlHLidx+CxuDYWZd2O/4GRqjnT7xLo88aBE8fAVSr876FyfJDBdYLSC66nyZ hadoop@node2
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7I19R6/js2tvicWcATaNzXlY+BVELMqDx2vhqfnlJAdlfOtyAgsaCduu8hu4El6WZymTckjX1YumeIHr8oGdajNtYugAcOcELijCFCnGUc4xcuVqVWw/Xf4TlT9wg3S9L33+Zj1EYvXp3lq04dyWO2VvZN2Iqq+bhk4tPZdGLttYqo24oXTr+kvyzYy7zeBTQmovVWaDT43ZpYSSzq5M8chjOvR0kPt/ygb/cuV2ZnTAXfNpTF/SHZIWeWqbAIexcdEnczATR/0f5Z/wFrAiG+Awppi1zNVLn1oRoC5sVQ61jqAuvZxAEp5AdgzRdTp3DFtynxVWqsgDmMFTOKNkX hadoop@node3

```

全部完事了记着赋予目录权限
`chmod 700 /home/hadoop/.ssh/ && chmod 700 /home/hadoop/ && chmod 600 /home/hadoop/.ssh/authorized_keys`
```
ssh hadoop@192.168.1.9
应该是免密码的

```


# 开始安装hadoop

## 下载并解压

```
cd /data/hadoop
把hadoop安装包复制到各个节点
[hadoop@node1 hadoop]$ scp hadoop-3.3.5.tar.gz hadoop@node2:/data/hadoop/ 
hadoop-3.3.5.tar.gz                                                                                                  100%  674MB  61.2MB/s   00:11    
[hadoop@node1 hadoop]$ scp hadoop-3.3.5.tar.gz hadoop@node3:/data/hadoop/ 
The authenticity of host 'node3 (192.168.1.40)' can't be established.
ECDSA key fingerprint is SHA256:tVUngomohqjvUulSZAx83tsqN6qn+8cMap1QbYjsFZc.
ECDSA key fingerprint is MD5:33:0e:00:28:c6:55:e7:97:85:a9:10:bc:ad:19:6c:b9.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node3,192.168.1.40' (ECDSA) to the list of known hosts.
hadoop-3.3.5.tar.gz                                                                                                  100%  674MB  61.2MB/s   00:11    
[hadoop@node1 hadoop]$ 
解压
tar -zxvf hadoop-3.3.5.tar.gz
使用mv命令重命名为app方便记录
mv hadoop-3.3.5 app


```

## 配置环境变量

```
vim ~/.bashrc

写入
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.402.b06-1.el7_9.x86_64/jre
export PATH=$JAVA_HOME/bin:$PATH
export HADOOP_HOME=/data/hadoop/app
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH


保存
source ~/.bashrc
```


## hadoop安装成功

```
[hadoop@node1 ~]$ hadoop version
Hadoop 3.3.5
Source code repository https://github.com/apache/hadoop.git -r 706d88266abcee09ed78fbaa0ad5f74d818ab0e9
Compiled by stevel on 2023-03-15T15:56Z
Compiled with protoc 3.7.1
From source with checksum 6bbd9afcf4838a0eb12a5f189e9bd7
This command was run using /data/hadoop/app/share/hadoop/common/hadoop-common-3.3.5.jar

```


# ### 4、Hadoop配置

配置`core-site.xml`（具体是`/data/hadoop/app/etc/hadoop/core-site.xml`）：

```xml
<configuration>
    <property>
            <name>fs.defaultFS</name>
            <value>hdfs://node1:9000</value>
    </property>
    <property>
            <name>hadoop.tmp.dir</name>
            <value>/data/hadoop/temp</value>
    </property>
</configuration>
```

- `fs.defaultFS`：`nameNode`的`HDFS`协议的文件系统通信地址
- `hadoop.tmp.dir`：`Hadoop`集群在工作的时候存储的一些临时文件的目录

配置`hdfs-site.xml`（具体是`/data/hadoop/app/etc/hadoop/hdfs-site.xml`）：

```xml
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/data/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.secondary.http.address</name>
        <value>node3:50090</value>
    </property>
    <property>
        <name>dfs.http.address</name>
        <value>192.168.56.200:50070</value>
    </property>
</configuration>
```

- `dfs.namenode.name.dir`：`NameNode`的数据存放目录
- `dfs.datanode.data.dir`：`DataNode`的数据存放目录
- `dfs.replication`：`HDFS`的副本数
- `dfs.secondary.http.address`：`SecondaryNameNode`节点的`HTTP`入口地址
- `dfs.http.address`：通过`HTTP`访问`HDFS`的`Web`管理界面的地址

配置`mapred-site.xml`（具体是`/data/hadoop/app/etc/hadoop/mapred-site.xml`）:

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
</configuration>
```

- `mapreduce.framework.name`：选用`yarn`，也就是`MR`框架使用`YARN`进行资源调度。

配置`yarn-site.xml`（具体是`/data/hadoop/app/etc/hadoop/yarn-site.xml`）：

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node3</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

- `yarn.resourcemanager.hostname`：指定`ResourceManager`所在的主机名
- `yarn.nodemanager.aux-services`：指定`YARN`集群为`MapReduce`程序提供`Shuffle`服务

配置`workers`文件（这个文件在旧版本叫`slaves`，因为技术政治化运动被改为`workers`，具体是`/data/hadoop/app/etc/hadoop/workers`：

```shell
node1
node2
node3
```

至此，核心配置基本完成。

```
同步配置到node23
scp -r /data/hadoop/app hadoop@node2:/data/hadoop
scp -r /data/hadoop/app hadoop@node3:/data/hadoop
```

在node1上初始化节点
`hadoop namenode -format`


### 7、启动和停止HDFS[#](https://www.cnblogs.com/throwable/p/14131255.html#7%E5%90%AF%E5%8A%A8%E5%92%8C%E5%81%9C%E6%AD%A2hdfs)

可以在任意一个节点中启动和停止`HDFS`，为了简单起见还是在`hadoop01`节点中操作：

- 启动：`start-dfs.sh`
- 停止：`stop-dfs.sh`

调用启动命令后，控制台输出如下：

```shell
[hadoop@hadoop01 hadoop]$ start-dfs.sh 
Starting namenodes on [hadoop01]
Starting datanodes
Starting secondary namenodes [hadoop03]
```

### 8、启动和停止YARN[#](https://www.cnblogs.com/throwable/p/14131255.html#8%E5%90%AF%E5%8A%A8%E5%92%8C%E5%81%9C%E6%AD%A2yarn)

`YARN`集群的启动命令必须在`ResourceManager`节点中调用，规划中的对应角色的节点为`hadoop03`，在该机器执行`YARN`相关命令：

- 启动：`start-yarn.sh`
- 停止：`stop-yarn.sh`

执行启动命令后，控制台输出如下：

```shell
[hadoop@hadoop03 data]$ start-yarn.sh 
Starting resourcemanager
Starting nodemanagers
```

### 9、查看所有节点的进程状态[#](https://www.cnblogs.com/throwable/p/14131255.html#9%E6%9F%A5%E7%9C%8B%E6%89%80%E6%9C%89%E8%8A%82%E7%82%B9%E7%9A%84%E8%BF%9B%E7%A8%8B%E7%8A%B6%E6%80%81)

分别查看集群中所有节点的进程状态，可以直接使用`jps`工具，具体结果如下：

```shell
[hadoop@hadoop01 hadoop]$ jps
8673 NameNode
8823 DataNode
9383 NodeManager
9498 Jps

[hadoop@hadoop02 hadoop]$ jps
4305 DataNode
4849 Jps
4734 NodeManager

[hadoop@hadoop03 data]$ jps
9888 Jps
9554 NodeManager
5011 DataNode
9427 ResourceManager
5125 SecondaryNameNode
```

可见进程是正常运行的。

![[Snipaste_2024-03-19_10-23-23.png]]
![[Snipaste_2024-03-19_10-24-26.png]]

# 后续软件补充

- Spark2.1.0
- HBase1.1.5
- Scala2.11.8
- MySQL
- Kafka_2.11-0.10.2.0
- Flume1.7.0
- sbt
- Maven3.3.9
- MongoDB3.2.17
- Hive2.1.0
- Scala IDE（包含Eclipse4.7.0和Maven、Scala、sbt插件）

>只是简单记录，可能某些地方会有小错误，如有发现，联系我修改