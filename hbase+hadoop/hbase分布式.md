# Hbase分布式

## 1. 机器配置

版本：java 8 + [hbase 2.1.8](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz) + [hadoop 2.7.7](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz)

节点：nnode1、nnode2、nnode3、nnode3四台机器。

## 2. 免密登录

将上述五台机器的公钥汇总到一起，然后添加到每台机器的```~/.ssh/authorized_keys```文件中。

## 3. hadoop

- ```shell
  wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz
  tar zxvf hadoop-2.7.7.tar.gz
  ```

- ```hadoop-env.sh```：

  ```shell
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
  ```

- ```core-site.xml```：

  ```xml
  <configuration>
      ...
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://master:9000</value>
      </property>
      ...
  </configuration>
  ```

- ``hdfs-site.xml``：

  ```xml
  <configuration>
      ...
      <property>
          <name>dfs.replication</name>
          <value>1</value>
      </property>
      <property>
          <name>dfs.name.dir</name>
          <value>/home/dsl/hdfs/name</value>
          <description>namenode data dir</description>
      </property>
      <property>
      	<name>dfs.data.dir</name>
          <value>/home/dsl/hdfs/data</value>
          <description>datanode data dir</description>
      </property>
      ...
  </configuration>
  ```

- ```slaves```：

  ```
  nnode1
  nnode2
  nnode3
  nnode4
  ```

## 4. hbase

- ```shell
  wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz
  tar zxvf hbase-2.1.8-bin.tar.gz
  ```

- ```hbase-env.sh```：

  ```shell
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/
  export HBASE_CLASSPATH=~/hadoop-2.7.7/etc/hadoop
  ```

- ```hbase-site.xml```：

  ```xml
  <configuration>
      <!-- 设置数据库文件存放目录 -->
      <property>
          <name>hbase.rootdir</name>
          <value>hdfs://master:9000/hbase</value>
          <description>The directory shared by region servers.</description>
      </property>
  	<property>
          <name>hbase.cluster.distributed</name>
          <value>true</value>
  	</property>
      <property>
      	<name>hbase.zookeeper.quorum</name>
          <!--一开始还加上了slave3，但是zookeeper服务器最好不要偶数个，可能会出现不一致。 -->
  		<value>master,slave1,slave2</value>
      </property>
  </configuration>
  ```

- ```regionservers```：

  ```
  nnode1
  nnode2
  nnode3
  nnode4
  ```

## 5. 环境变量

每台机器都要做如下配置

- ```/etc/profile```：

  ```shell
  export HBASE_HOME=/home/dsl/hbase-2.1.8
  export HBASE_CONF_DIR=$HBASE_HOME/conf
  export PATH=$HBASE_HOME/bin:$PATH
  
  export HADOOP_HOME=/home/dsl/hadoop-2.7.7
  export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
  export PATH=$HADOOP_HOME/bin:$PATH
  ```

  ```shell
  source /etc/profile
  ```

- ```/etc/hosts```：

  ```
  # 首先把 127.0.1.1　xxxxx　这一行给注释掉
  # 不然通信的时候不知道为什么会出现xxxxx，导致无法解析ip，会出现许多莫名的错误，巨坑。
  192.168.1.54 master
  192.168.1.55 slave1
  192.168.1.56 slave2
  192.168.1.57 slave3
  ```

## 6. 部署

将hbase和hadoop两个文件夹发送到所有节点上。

## 7. 启动

- 第一次启动需要格式化hdfs文件系统

    ```shell
    hadoop namenode -format
    ```

	格式化成功会出现以下信息：

    ```
    20/02/09 17:03:41 INFO common.Storage: Storage directory 		/home/dsl/hdfs/name has been successfully formatted.
    ```

- 启动hadoop

  ```shell
  ~/hadoop-2.7.7/sbin/start-all.sh
  ```

  使用jps命令，可在master节点观察到```SecondaryNameNode```、``` NameNode```、```ResourceManager```三个进程，在slave节点观察到```DataNode```、```NodeManager```两个进程。(均不包括jps进程)

- 启动hadoop之后，启动hbase：

    ```shell
    ~/hbase-2.1.8/bin/start-hbase.sh
    ```

    使用jps命令，可在master节点观察到```HMaster```、``` HQuormpeer```两个进程，在slave节点观察到```HRegionServer```进程。(均不包括jps进程及hadoop相应进程)

- 关闭hbase

  ```shell
  ~/hbase-2.1.8/bin/stop-hbase.sh
  ```

- 关闭hadoop

  ```shell
  ~/hadoop-2.7.7/sbin/stop-all.sh
  ```


## 8. ntp同步时间

参考链接：https://blog.csdn.net/weixin_39469127/article/details/90116498

- master节点的```/etc/ntp.conf```：

  ```
  # Clients from this (example!) subnet have unlimited access, but only if
  # cryptographically authenticated.
  # restrict 192.168.123.0 mask 255.255.255.0 nomodify 
  将上面这一段第三行改为
  restrict 192.168.1.0 mask 255.255.255.0 [notrust nomodify]
  中括号中的删掉最好
  ```

  ```
  将pool开头的五行注释掉，添加以下内容：
  # aliyun clock
  server 203.107.6.88
  restrict 203.107.6.88
  ```

  编辑完之后，手动同步一次时间(master和aliyun)

  - 首先关闭ntp

    ```
    sudo systemctl stop ntp.service
    ```

  - 手动更新

    ```
    sudo ntpdate [-d] 203.107.6.88
    -d 为开启debug
    ```

  - 再开启ntp

    ```
    sudo systemctl start ntp.service
    ```

  - 一般来说，在重新开启ntp之后，15分钟内才开始同步。这期间可以不断用```ntpstat```命令来查看同步进度。

    ```
    没开始时：
    unsynchronised
       polling server every 8 s
    开始之后：
    synchronised to NTP server (203.107.6.88) at stratum 3 
       time correct to within 463 ms
       polling server every 64 s
    ```

  - 查看master节点和aliyun服务器的状态

    ```
    ntpq -p
    ```

    输出如下：

    ```shell
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    *203.107.6.88    10.137.38.86     2 u   56   64  177   36.311    0.045   0.464
    ```

- slave节点的```/etc/ntp.conf```：

  ```
  将pool开头的五行注释掉，添加以下内容：
  # master clock
  server 192.168.1.54
  restrict 192.168.1.54
  ```

  编辑后执行：

  ```
  sudo systemctl stop ntp.service
  sudo ntpdate 192.168.1.54
  sudo systemctl start ntp.service
  ```

  ntpdate可能出现```no server suitable for synchronization found ```错误。

  debug发现原因有二：

  - Server dropped: no data

    搜检ntp的版本，若是你应用的是ntp4.2（包含4.2）之后的版本，在restrict的定义中应用了notrust的话，会导致以上错误。因此删除掉notrust。

  - Server dropped: Strata too high 

    在master从阿里云启动ntp服务后，master自身或与其server的同步的须要一个时候段，这个过程可能是5分钟，在这个时候之客户端运行ntpdate会产生no server suitable for synchronization found的错误。等了一会之后确实问题消失。 

