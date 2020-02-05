 # HBase 配置记录

## 1. 版本选择

java 8 + [hbase 2.1.8](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz) + [hadoop 2.7.7](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz)

## 2. 安装过程

OS：manjaro 18.04

### CASE 1. 单机模式

 该模式下不需要hdfs，只下载hbase即可。

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz
tar zxvf hbase-2.1.8-bin.tar.gz
```

编辑```hbase-2.1.8/conf/hbase-env.sh```，添加以下行

```shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk/
```

编辑```hbase-2.1.8/conf/hbase-site.xml```，编辑如下：

```xml
<configuration>
    <!-- 设置数据库文件存放目录 -->
    <property>
        <name>hbase.rootdir</name>
        <!-- "file://" does not belong to your dir.-->
        <value>file:///tmp/hadoop-wm775825</value>
        <description>The directory shared by region servers.</description>
    </property>
	<!-- false是单机模式，true是分布式模式，默认为false  -->
	<property>
        <name>hbase.cluster.distributed</name>
        <value>false</value>
	</property>
</configuration>
```

至此已经完成了hbase的单机模式配置，可以在单机上操作hbase。

访问```localhost:16010```可以看到hbase中master的详细信息，说明配置成功。

### CASE 2. 伪分布模式

#### 1. 首先执行CASE1

#### 2. 修改环境变量

编辑```/etc/profile```，添加以下行

```
export HBASE_HOME=~/hbase-2.1.8
export PATH=$HBASE_HOME/bin:$PATH
```

使之生效：

```shell
source /etc/profile
```

#### 3. 下载hadoop并编辑配置文件

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz
tar zxvf hadoop-2.7.7.tar.gz
```

编辑```hbase-2.1.8/conf/hbase-env.sh```，添加以下行：

```shell
export HBASE_CLASSPATH=~/hadoop-2.7.7/etc/hadoop
```

编辑```hadoop-2.7.7/etc/hadoop/hadoop-env.sh```，添加以下行：

```shell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk/
```

编辑```hadoop-2.7.7/etc/hadoop/core-site.xml```，添加以下行：

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:8020</value>
    </property>
</configuration>
```

编辑```hadoop-2.7.7/etc/hadoop/hdfs-site.xml```，添加以下行：

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 4. 启动hdfs

运行

```shell
cd ~/hadoop2.7.7
./bin/hdfs namenode -format
```

出现以下类似信息，说明文件系统创建成功。

```
19/12/26 15:35:55 INFO common.Storage: Storage directory /tmp/hadoop-wm775825/dfs/name has been successfully formatted.
19/12/26 15:35:55 INFO namenode.FSImageFormatProtobuf: Saving image file /tmp/hadoop-wm775825/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
19/12/26 15:35:55 INFO namenode.FSImageFormatProtobuf: Image file /tmp/hadoop-wm775825/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 325 bytes saved in 0 seconds.
```

启动sshd服务

```shell
systemctl start sshd.service
```

启动namenode和datanode进程：

```shell
cd ~/hadoop-2.7.7
./sbin/start-dfs.sh
```

输出如下信息：

```
localhost: starting namenode, logging to /home/wm775825/hadoop-2.7.7/logs/hadoop-wm775825-namenode-wm775825-pc.out
localhost: starting datanode, logging to /home/wm775825/hadoop-2.7.7/logs/hadoop-wm775825-datanode-wm775825-pc.out
0.0.0.0: starting secondarynamenode, logging to /home/wm775825/hadoop-2.7.7/logs/hadoop-wm775825-secondarynamenode-wm775825-pc.out
```

启动yarn：

```shell
./sbin/start-yarn.sh
```

输出如下信息：

```
starting resourcemanager, logging to /home/wm775825/hadoop-2.7.7/logs/yarn-wm775825-resourcemanager-wm775825-pc.out
localhost: starting nodemanager, logging to /home/wm775825/hadoop-2.7.7/logs/yarn-wm775825-nodemanager-wm775825-pc.out
```

此时使用```jps```命令，应观察到```DataNode```、```SecondaryNameNode```、``` NameNode```、```Jps```、```NodeManager```、```ResourceManager```六个进程。

还可以通过```localhost:50070```查看hdfs是否启动成功。

#### 5. 启动hbase

首先修改```hbase-2.1.8/conf/hbase-site.xml```，修改如下：

```xml
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
```

然后启动```zookeeper```、```master```、```regionserver```：

```shell
cd ~/hbase-2.1.8
./bin/hbase-daemon.sh start zookeeper
./bin/hbase-daemon.sh start master
./bin/hbase-daemon.sh start regionserver
```

使用```jps```，观察到多了```HQuorumPeer```、```HMaster```、```HRegionServer```三个进程。

配置成功后，可访问```localhost:16030```查看伪分布模式下的hbase中```RegionServer```的详细信息。

#### 6. 停止命令

```shell
~/hadoop-2.7.7/sbin/stop-yarn.sh
~/hadoop-2.7.7/sbin/stop-dfs.sh
~/hbase-2.1.8/bin/stop-hbase.sh
```

### CASE 3. 分布式

在onode3、nnode1、nnode2、nnode3、nnode3五台机器上配置。

#### 1. 免密登录

将上述五台机器的公钥汇总到一起，然后添加到每台机器的```~/.ssh/authorized_keys```文件中。

#### 2. 分发







































