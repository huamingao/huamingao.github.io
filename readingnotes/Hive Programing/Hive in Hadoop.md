## Hive on Hadoop 环境搭建

Hive依赖于Hadoop。Hadoop依赖于Java。

#### Hadoop环境搭建
- 本地模式
- 伪分布模式
- 分布模式

下面以伪分布模式为例

#### 准备工作

1. 设置机器名和host文件
```
vi /etc/sysconfig/network
vi /etc/hosts
```
2. 关闭防火墙
```
chkconfig iptables off
```
3. 打开下面文件，设置SELINUX=disabled
```
vi /etc/selinux/config
```
4. 安装JDK，[猛戳这里](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html)下载安装包

```
tar -zxf jdk-7u55-linux-x64.tar.gz
mv jdk1.7.0_55/ /app/lib
ll /app/lib
```

5. 配置JDK路径，打开/etc/profile添加环境变量
```
export JAVA_HOME=/app/lib/jdk1.7.0_55
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

6. SSH无密码验证配置

打开vi /etc/ssh/sshd_config，开放如下三个配置
```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```
开放后重启sshd服务

```
service sshd restart
```
生成公钥和私钥
```
ssh-keygen -r rsa
```
进入~/.ssh目录把公钥命名为authorized_keys，设置authorized_keys读写权限
```
cp id_rsa.pub authorized_keys
chmod 400 authorized_keys
```

#### Hadoop环境搭建
1. 根据[官网指引](http://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz)，下载haddop二进制包,解压放到安装目录

```
 wget ...
 tar -xzf hadoop-1.1.2-bin.tar.gz
 rm -rf /app/hadoop-1.1.2
 mv hadoop-1.1.2 /app
```
2. 创建hdfs文件系统目录

```
 mkdir tmp
 mkdir hdfs
 mkdir hdfs/name
 mkdir hdfs/data
```
3.  使用chmod -R 755 data命令把hdfs/data设置为755，否则DataNode会启动失败
4.  在hadoop-env.sh中添加jdk路径、hadoop/bin路径
```
export JAVA_HOME=/app/lib/jdk1.7.0_55
export PATH=$PATH:/app/hadoop-1.1.2/bin
```
5.  配置core-site.xml

```
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://hadoop:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/app/hadoop-1.1.2/tmp</value>
  </property>
</configuration>
```
6. 配置hdfs-site.xml

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <name>dfs.name.dir</name>
    <value>/app/hadoop-1.1.2/hdfs/name</value>
  </property>
  <property>
    <name>dfs.data.dir</name>
    <value>/app/hadoop-1.1.2/hdfs/data</value>
  </property>
</configuration>
```
7. 配置mapred-site.xml

```
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>hadoop:9001</value>
  </property>
</configuration
```
8. 配置masters和slaves文件，写入主机名hadoop

```
vi masters
vi slaves
```
9. 格式化namenode

```
./hadoop namenode -format
```
10. 启动hadoop

```
./start-all.sh
```

11. 使用jps命令检查各后台进程是否成功启动

#### Hive环境搭建
1. 安装mysql (略)
2. 在mysql中创建hive用户并授权
```
mysql>create user 'hive' identified by 'hive';
mysql>grant all on *.* TO 'hive'@'%' identified by 'hive' with grant option;
mysql>grant all on *.* TO 'hive'@'localhost' identified by 'hive' with grant option;
mysql>flush privileges;
```
3. 使用hive用户登录，创建hive数据库

```
mysql -uhive -phive -h hadoop
mysql>create database hive; # 若存在则无需再创建
mysql>show databases;
```
4. 安装hive二进制包
5. 配置/etc/profile环境变量，添加内容如下
```
export HIVE_HOME=/app/hive-0.12.0
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:$HIVE_HOME/bin
```
6. 设置hive-env.sh配置文件，分别设置HADOOP_HOME和HIVE_CONF_DIR两个值

```
export HADOOP_HOME=/app/hadoop-1.1.2
export HIVE_CONF_DIR=/app/hive-0.12.0/conf
```
7. 设置hive-site.xml配置文件
- 默认metastore在本地，添加配置改为非本地
- hive默认为derby数据库，需要把相关信息调整为mysql数据库
- 把hive.metastore.schema.verification配置项值修改为false

```
<property>
  <name>hive.metastore.local</name>
  <value>false</value>
</property>
<property>
   <name>hive.metastore.uris</name>
   <value>thrift://hadoop:9083</value>
   <description>Thrift URI for the remote metastore. ...</description>
 </property>
 <property>
   <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://hadoop:3306/hive?=createDatabaseIfNotExist=true</value>
   <description>JDBC connect string for a JDBC metastore</description>
 </property>
 <property>
   <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
   <description>Driver class name for a JDBC metastore</description>
 </property>
 <property>
   <name>javax.jdo.option.ConnectionUserName</name>
   <value>hive</value>
   <description>username to use against metastore database</description>
 </property>
 <property>
   <name>javax.jdo.option.ConnectionPassword</name>
   <value>hive</value>
   <description>password to use against metastore database</description>
 </property>
 <property>
   <name>hive.metastore.schema.verification</name>
   <value>false</value>
    <desc....>
 </property>
 <property>
 <name>hive.server2.thrift.sasl.qop</name>
   <value>auth</value>
   <des.....
 </property>
```

8. 验证部署
启动metastore和hiveserver

```
hive --service metastore &
hive --service hiveserver &
```
使用jps命令查看是否新增了两个RunJar进程

