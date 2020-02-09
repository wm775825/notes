# Hbase分布式

## 1. 机器配置

版本：java 8 + [hbase 2.1.8](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz) + [hadoop 2.7.7](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz)

节点：onode3、nnode1、nnode2、nnode3、nnode3五台机器。(onode3暂时不用)

## 2. 免密登录

将上述五台机器的公钥汇总到一起，然后添加到每台机器的```~/.ssh/authorized_keys```文件中。

## 3. hadoop

- ```shell
  wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.7.7/hadoop-2.7.7.tar.gz
  tar zxvf hadoop-2.7.7.tar.gz
  ```

- ```hadoop-env.sh```：

  ```shell
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk/
  ```

- ```hbase-env.sh```：

  ```shell
  export HBASE_CLASSPATH=~/hadoop-2.7.7/etc/hadoop
  ```

- ```core-site.xml```：

  ```xml
  <configuration>
      ...
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://nnode1:8020</value>
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
      ...
  </configuration>
  ```

## 4. hbase

- ```shell
  wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/2.1.8/hbase-2.1.8-bin.tar.gz
  tar zxvf hbase-2.1.8-bin.tar.gz
  ```

- ```hbase-env.sh```：

  ```shell
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk/
  ```

- ```hbase-site.xml```：

  ```xml
  <configuration>
      <!-- 设置数据库文件存放目录 -->
      <property>
          <name>hbase.rootdir</name>
          <!-- "file://" does not belong to your dir.-->
          <value>file://nnode1:8020/~/hbase</value>
          <description>The directory shared by region servers.</description>
      </property>
  	<!-- false是单机模式，true是分布式模式，默认为false  -->
  	<property>
          <name>hbase.cluster.distributed</name>
          <value>true</value>
  	</property>
  </configuration>
  ```

## 5. 环境变量

- ```/etc/profile```：

  ```shell
  export HBASE_HOME=~/hbase-2.1.8
  export PATH=$HBASE_HOME/bin:$PATH
  ```

  ```shell
  source /etc/profile
  ```

  