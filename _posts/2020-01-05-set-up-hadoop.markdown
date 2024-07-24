---
layout: post
title:  "搭建Hadoop集群"
date:   2020-01-05 02:00:00 -1500
categories: hadoop 
---

手把手教你搭建Hadoop集群

---

# 前言

&emsp;目前很多企业比如银行的BI系统通常是建立在传统数据仓库上，实现载体主要是Oracle、Teradata、GreenPlum等关系型数据库。

&emsp;但是在如今信息大爆炸的时代，数据都是实时更新且数据量非常大，传统数据仓库已经不能满足BI系统的业务需求了。总的来说，传统数据仓库劣势主要有以下三点：

1. 不能满足海量数据存储需求

2. 不能处理不同类型的数据

3. 计算与处理能力差

&emsp;随着大数据技术趋于成熟，越来越多企业尝试将大数据技术应用于BI系统中。但并不是每个企业都需要打造自己的大数据平台，量力而行吧，可以自研，比如BAT（百度Apollo、阿里飞天系统、腾讯Angel），也可以采购，比如传统大企业，也可以租用，比如用阿里云和AWS。

&emsp;在众多大数据分析工具中，以开源组件Hadoop为主流。一个典型的基于Hadoop大数据平台的数据仓库架构如下所示：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">基于Hadoop平台的数据仓库架构</div>
</center>

&emsp;这是一个四层架构：OLTP层、数据仓库层（含ODS）、数据集市层、应用层。新型的数据仓库架构与传统数据仓库架构的唯一区别就是数据仓库层的不同。新型的数据仓库架构除了使用关系型数据库，同时将大数据技术应用于数据仓库中。

&emsp;从图中可以看到，搭建Hadoop大数据平台需要的组件包括Zookeeper、Kafka、Hadoop、Hbase、Hive、Spark、Storm、Flink以及MySQL关系型数据库等：Zookeeper作为管理中心；Kafka分布式发布订阅消息；Hadoop分布式批处理；Storm和Flink分布式流处理；而Spark既可批处理，亦可作流处理；Hive基于hdfs存储的关系型数据库；HBase基于hdfs存储的非关系型数据库。实际上，我们可以将其与Lambda大数据构架对应，Hadoop和Spark负责批处理层（Batch Layer），Storm负责加速层（Speed Layer），而服务层（Serving Layer）则对应由数据集市和前端应用。

&emsp;一个简单的Hadoop集群至少应该有三台机器，在每台机器都安装Zookeeper、Kafka、Hadoop、Hbase、Hive、Spark、Storm、Flink，在其中一台机器上安装MySQL。好啦，话不多说，下面我们开始搭建Hadoop集群。

---

&emsp;

# 概览

1. [准备工作](#anchor1)

   + [准备虚拟机](#anchor1_1)

   + [网络配置](#anchor1_2)

      - [网卡配置](#anchor1_2_1)

      - [关闭防火墙](#anchor1_2_2)

      - [修改hosts](#anchor1_2_3)

   + [下载基础工具](#anchor1_3)

   + [创建hadoop用户](#anchor1_4)

   + [分发密钥](#anchor1_5)

   + [安装ntp时钟同步服务](#anchor1_6)

2. [安装jdk](#anchor2)

3. [在cluster2上安装MySQL](#anchor3)

4. [安装Zookeeper](#anchor4)

5. [安装Kafka](#anchor5)

6. [安装Hadoop](#anchor6)

7. [安装HBase](#anchor7)

8. [安装Hive](#anchor8)

9. [安装Spark](#anchor9)

   + [安装Scala](#anchor9_1)

   + [安装Spark](#anchor9_2)

10. [安装Storm](#anchor10)

11. [安装Flink](#anchor11)

&emsp;

---

<span id = "anchor1">&emsp;</span>

# 准备工作

因为要安装的工具比较多，所以先列出工具清单。

<table width="90%" border="1">
    <thead>
        <tr>
            <th width="60%" align="center">软件包文件名</th>
            <th width="15%" align="center">软件名称</th>
            <th width="15%" align="center">软件版本</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>jdk-8u201-linux-x64.tar.gz</td>
            <td align="center">JDK</td>
            <td align="center">1.8</td>
        </tr>
        <tr>
            <td>mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz</td>
            <td align="center">MySQL</td>
            <td align="center">8.0.15</td>
        </tr>
        <tr>
            <td>zookeeper-3.4.13.tar.gz</td>
            <td align="center">Zookeeper</td>
            <td align="center">3.4.13</td>
        </tr>
        <tr>
            <td>kafka_2.12-2.1.1.tgz</td>
            <td align="center">Kafka</td>
            <td align="center">2.1.1</td>
        </tr>
        <tr>
            <td>hadoop-3.1.2.tar.gz</td>
            <td align="center">Hadoop</td>
            <td align="center">3.1.2</td>
        </tr>
        <tr>
            <td>hbase-2.1.3-bin.tar.gz</td>
            <td align="center">HBase</td>
            <td align="center">2.1.3</td>
        </tr>
        <tr>
            <td>apache-hive-2.3.4-bin.tar.gz</td>
            <td align="center">Hive</td>
            <td align="center">2.3.4</td>
        </tr>
        <tr>
            <td>scala-2.12.8.tgz</td>
            <td align="center">Scala</td>
            <td align="center">2.12.8</td>
        </tr>
        <tr>
            <td>spark-2.4.0-bin-hadoop2.7.tgz</td>
            <td align="center">Spark</td>
            <td align="center">2.4.0</td>
        </tr>
        <tr>
            <td>apache-storm-1.2.2.tar.gz</td>
            <td align="center">Storm</td>
            <td align="center">1.2.2</td>
        </tr>
        <tr>
            <td>mysql-connector-java-8.0.15.jar</td>
            <td align="center">MySQL链接器</td>
            <td align="center">8.0.15</td>
        </tr>
        <tr>
            <td>CentOS-7-x86_64-Minimal-1708.iso</td>
            <td align="center">CentOS7</td>
            <td align="center">最小安装版镜像</td>
        </tr>
        <tr>
            <td>MobaXterm_Portable_v12.1.zip</td>
            <td align="center">MobaXterm</td>
            <td align="center">12.1</td>
        </tr>
    </tbody>
</table>

<span id = "anchor1_1">&emsp;</span>

## 准备虚拟机

&emsp;虚拟机可以在电脑上自行安装，常用的工具有VMware和VirtualBox。也可以租3台云服务器来搭建集群。

&emsp;如果在电脑上自己安装 3 台 linux 虚拟机，推荐使用CentOS 7系统镜像，文件名“CentOS-7-x86_64-Minimal-1708.iso”。

&emsp;如果购买云服务器，比如阿里云服务器，需要注意一点，因为端口是默认关闭的，为使集群之间正常通信，要将服务器的部分端口开放。

<span id = "anchor1_2">&emsp;</span>

## 网络配置

<span id = "anchor1_2_1"></span>

### 虚拟机网卡配置

&emsp;以Vmware安装虚拟机为例，为使虚拟机能与主机通信，网卡应按如下设置：

&emsp;&emsp;1) 虚拟机的网络设置选择“NAT”模式，也就是使用VMnet8虚拟网卡。

<center>
    <img style="width:50%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\vmware.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">设置虚拟机网络适配器</div>
</center>

&emsp;&emsp;2) 配置VMnet8虚拟网卡的IP地址，如下图所示。比如我这里设置地址为“192.168.61.0”，那么接下来三台虚拟机的IP应在网段“192.168.61.128-192.168.61.254”内选择。

<center>
    <img style="width:50%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\vmware2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">设置VMnet8虚拟网卡IP地址</div>
</center>

&emsp;&emsp;3) 开启虚拟机，输入“ip addr”查看网卡名称。比如我的 cluster1 虚拟机网卡名称为 ens33，那么输入“vi /etc/sysconfig/network-scripts/ifcfg-ens33”编辑网络配置文件。

<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\ipaddr2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">查看网卡信息</div>
</center>

<center>
    <img style="width:36%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\ens332.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">配置虚拟机网卡</div>
</center>

&emsp;&emsp;&emsp;编辑网卡配置文件。设置BOOTPROTO为 static 或 none，IP地址范围为“192.168.61.128-192.168.61.254”，VMnet8网关地址为“192.168.61.2”，掩码可设可不设。设置开机启动网卡“ONBOOT=yes”。
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33

# 设置BOOTPROTO为 static 或 none
BOOTPROTO=static
ONBOOT=yes
# 地址范围“192.168.61.128-192.168.61.254”
IPADDR=192.168.61.130
# 网关选择VMnet8网关
GATEWAY=192.168.61.2
# 掩码可不设，默认为255.255.255.0
NETMASK=255.255.255.0
```
&emsp;&emsp;&emsp;然后重启网络：
```
service network restart
```

&emsp;&emsp;4) 因为静态地址无法上网，需要添加DNS服务器才能上网，常用的DNS服务器有Google的“8.8.8.8” 服务器和“8.8.4.4” 服务器。
```
vi /etc/resolv.conf
nameserver 8.8.8.8
```

<span id = "anchor1_2_2">&emsp;</span>

### 关闭防火墙

```
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#禁用selinux
vi /etc/selinux/config
SELINUX=disabled

#查看selinux状态，显示“Disabled”表示已被禁用
getenforce
```

<span id = "anchor1_2_2">&emsp;</span>

### 修改hosts

&emsp;在hosts文件中添加三台机器的ip地址与域名的映射关系。
```
192.168.61.130 cluster1
192.168.61.131 cluster2
192.168.61.132 cluster3
```
&emsp;如果是阿里云服务器或其他云服务器，要注意云服务器的外网和内网的关系，外网地址是云服务器厂商提供给用户使用的地址，在万维网中可访问；内网地址实际上是厂商的私有局域网中的网址，用于云服务器之间的内部通信。所以使用云服务器搭建集群的时候，配置hosts需要将本机映射到内网地址，其他机器映射到外网地址。下面给出示范：

&emsp;&emsp;cluster1的外网地址为47.98.176.164，内网地址为172.16.136.37

&emsp;&emsp;cluster2的外网地址为47.98.47.81，内网地址为172.16.206.43

&emsp;&emsp;cluster2的外网地址为116.62.119.79，内网地址为172.16.29.7

&emsp;&emsp;那么，三台机器的hosts配置应该是这样的：

&emsp;&emsp;cluster1
```
172.16.136.37 cluster1
47.98.47.81 cluster2
116.62.119.79 cluster3
```
&emsp;&emsp;cluster2
```
47.98.176.164 cluster1
172.16.206.43 cluster2
116.62.119.79 cluster3
```
&emsp;&emsp;cluster3
```
47.98.176.164 cluster1
47.98.47.81 cluster2
172.16.29.7 cluster3
```

<span id = "anchor1_3">&emsp;</span>

## 下载基础工具
```
yum install perl*
yum install ntpdate
yum install libaio
yum install screen
```

<span id = "anchor1_4">&emsp;</span>

## 创建hadoop用户
```
groupadd hadoop
useradd -s /bin/bash -g hadoop -d /home/hadoop -m hadoop
```

<span id = "anchor1_5">&emsp;</span>

## 分发密钥

&emsp;每个节点生成私钥和公钥，然后将公钥分发给其他节点，这样能避免机器之间的一些互相信任问题和访问权限问题。
```
# 切换到 hadoop 用户，生成ssh密钥（私钥和公钥）
ssh-keygen -t rsa

# 将ssh公钥拷贝给其他节点，一路回车
ssh-copy-id cluster1
ssh-copy-id cluster2
ssh-copy-id cluster3
```
+ 如果分发给远程机器的公钥过期，比如cluster1分发给cluster2公钥时报错“The ECDSA host key for cluster2 has changed”，那就在本地删除关于远程机器的授权信息，然后重新授权：
```
ssh-keygen -R cluster2
```
<span id = "anchor1_6">&emsp;</span>

## 安装ntp时钟同步服务

&emsp;分布式并行操作往往会在集群时间不同步时出现问题，所以这里需要安装ntp时钟同步服务。
```
# ntpdate服务器每个节点都要安装：
yum install ntpdate
```
&emsp;以下操作只在cluster1上进行：
```
yum install ntp

# 配置ntp
vi /etc/ntp.conf
#注释掉
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
#在最后面添加
restrict default ignore
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap
server 127.127.1.0

#重启cluster1的ntp服务器
service ntpd restart
chkconfig ntpd on
```
&emsp;以下操作在cluster2、cluster3上进行：
```
# 设定每天00:00向服务器同步时间，并写入日志
# “crontab -e”命令会自动打开一个空文件，然后添加内容“0 0 * * * /usr/sbin/ntpdate cluster1>> /root/ntpd.log”
crontab -e
0 0 * * * /usr/sbin/ntpdate cluster1>> /root/ntpd.log

# 手动同步时间。在cluster2、cluster3上，输入“ntpdate cluster1”命令，将时间与cluster1同步。
ntpdate cluster1
```

<span id = "anchor2">&emsp;</span>

## 安装jdk

&emsp;上传文件“jdk-8u201-linux-x64.tar.gz”至/usr/local路径下，并解压：
```
tar -zxvf jdk-8u201-linux-x64.tar.gz
```
&emsp;添加到环境变量：
```
vi /etc/profile
export JAVA_HOME=/usr/local/jdk1.8.0_201/
export JRE_HOME=/usr/local/jdk1.8.0_201/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH
source /etc/profile
```
&emsp;拷贝到其他节点：
```
scp -r /usr/local/jdk1.8.0_201/ cluster2:/usr/local/
scp -r /usr/local/jdk1.8.0_201/ cluster3:/usr/local/
```

<span id = "anchor3">&emsp;</span>

## 在cluster2上安装MySQL

&emsp;只在cluster2上安装MySQL，其他节点不安装。

&emsp;查看是否已经安装mysql：
```
rpm -qa|grep -i mysql
#如已安装，卸载删除
rpm -ev perl-DBD-MySQL-4.023-6.el7.x86_64 --nodeps
```
&emsp;上传文件“mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz”至/usr/local路径下，并解压：
```
xz -dv mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz
tar -xvf mysql-8.0.15-linux-glibc2.12-x86_64.tar
#重命名文件夹
mv mysql-8.0.15-linux-glibc2.12-x86_64 mysql
```
&emsp;&emsp;1) 编辑配置文件my.cnf。mysql的配置文件my.cnf放在/etc/目录下，完整路径为“/etc/my.cnf”，如果没有，则新建一个。my.cnf内容如下：
```
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
port=3306
user=mysql

[mysqld_safe]
log-error=error.log
pid-file=/usr/local/mysql/data/mysql.pid
```
&emsp;&emsp;2) 添加用户mysql（这是由于mysql的安全机制所规定的。基于安全考虑，mysql运行的时候使用一个独立的账号，如果mysql被黑了那么开始拿到的权限就是那个创建的账号而不是默认的root），并为其配置权限：
```
#添加用户。参数“-s /bin/false”表示用户不能登录，并且不会有任何提示。实际上，mysql用户只提供给mysql服务器内部使用
groupadd mysql
useradd -g mysql -s /bin/false mysql

#配置权限
cd /usr/local/mysql
chown -R mysql:mysql .
chown -R mysql:mysql /usr/local/mysql
# chmod -R 755 /usr/local/mysql/data
```
&emsp;&emsp;3) 初始化数据库，初始化过程中会生成mysql随机密码：
```
cd /usr/local/mysql/bin/
/usr/local/mysql/bin/mysqld --initialize
```
完整的初始化命令为：
```
mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/mysql.pid
```
&emsp;&emsp;4) 添加到环境变量：
```
vi /etc/profile
export MYSQL_HOME=/usr/local/mysql
export PATH=$PATH:$MYSQL_HOME/lib:$MYSQL_HOME/bin
source /etc/profile
```
&emsp;&emsp;5) 添加mysql到服务列表中，用service命令可执行/etc/init.d/目录中相应服务的脚本：
```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
# 添加可执行权限
chmod +x /etc/init.d/mysql
```
&emsp;&emsp;6) 设置mysql服务器开机自启动
```
# 设置开机启动
chkconfig --add mysql
# 查看是否设置成功
chkconfig --list
# 取消开机启动
chkconfig --del mysql
```
&emsp;&emsp;7) 启动服务器
```
service mysql start
```
&emsp;&emsp;&emsp;也可以使用mysqld_safe脚本启动mysql。参数“--user=user_name”表示以用户名user_name或数字用户ID即user_id运行mysqld服务器。这里的用户指系统登录账户，而不是授权表中的MySQL用户，但是我尝试过使用不存在的系统用户也可以启动服务器，所以感觉这个参数没有实际用处。
```
/usr/local/mysql/bin/mysqld_safe --user=xijiawei &
```
&emsp;&emsp;8) 第一次登录mysql，需要修改密码。使用初始化时生成的随机密码登录并修改密码：
+ 登录mysql
```
mysql -uroot -p
```
+ 选择mysql数据库
```
mysql> use mysql;
```
+ 修改密码为“fionasit61”
```
mysql> alter user user() identified by 'fionasit61';
或
mysql> alter user 'root'@'localhost' identified by 'fionasit61';
```
&emsp;&emsp;9) 如果密码忘记，想要重置密码，可以按如下步骤操作。
+ 免密登陆模式启动服务器。有两种方式：
   1. 在my.cnf中添加“skip-grant-tables”保存退出。修改完后记得将其注释或删除
   2. 直接用命令“mysqld_safe --skip-grant-tables”启动mysql
+ 登陆数据库，提示输入密码时直接敲回车，选择mysql数据库，然后将密码置空：
```
mysql> update user set authentication_string = '' where user = 'root';
```
+ 常规模式重启服务器：
```
service mysql restart
```
+ 再次登录数据库，因为密码设置为空，所以直接回车可进入数据库，然后修改密码：
```
mysql> alter user user() identified by 'fionasit61';
或
mysql> alter user 'root'@'localhost' identified by 'fionasit61';
```
&emsp;&emsp;10) 创建远程连接账号'root'并允许远程连接，否则无法远程连接登录mysql：
```
# 选择mysql数据库
mysql> use mysql;
# 创建远程账户'root'，%表示允许所有远程的地址，远程连接密码为“fionasit61”
mysql> create user root@'%' identified by 'fionasit61';
# 赋予远程账户权限
mysql> grant all privileges on *.* to root@'%' with grant option;
# 刷新权限
mysql> flush privileges;
```

<span id = "anchor4">&emsp;</span>

## 安装Zookeeper

&emsp;三台机器都安装，下面以在cluster1上安装为例。

&emsp;上传文件“zookeeper-3.4.13.tar.gz”至/usr/local路径下，并解压：
```
tar -xzf zookeeper-3.4.13.tar.gz
```
&emsp;1) 新建zoo.cfg配置文件：
```
vi /usr/local/zookeeper-3.4.13/conf/zoo.cfg
```
向zoo.cfg文件中添加以下内容：
```
# 客户端心跳时间(毫秒) 
tickTime=2000 
# 允许心跳间隔的最大时间 
initLimit=10 
# 同步时限 
syncLimit=5 
# 数据存储目录 
dataDir=/home/hadoop_files/hadoop_data/zookeeper
# 数据日志存储目录 
dataLogDir=/home/hadoop_files/hadoop_logs/zookeeper/dataLog 
# 端口号 
clientPort=2181 
# 集群节点和服务端口配置 
server.1=cluster1:2888:3888 
server.2=cluster2:2888:3888 
server.3=cluster3:2888:3888 
```
&emsp;2) 编辑zkEnv.sh脚本文件：
```
vi /usr/local/zookeeper-3.4.13/bin/zkEnv.sh
```
找到文件中的ZOOCFGDIR和ZOO_LOG4J_PROP两项，修改为如下内容：
```
if [ "x${ZOO_LOG_DIR}" = "x" ]
then
    ZOO_LOG_DIR="/home/hadoop_files/hadoop_logs/zookeeper/logs" 
fi 
if [ "x${ZOO_LOG4J_PROP}" = "x" ] 
then
    ZOO_LOG4J_PROP="INFO,ROLLINGFILE" 
fi
```
&emsp;3) 编辑log4j.properties文件：
```
vi /usr/local/zookeeper-3.4.13/conf/log4j.properties
```
找到文件中的zookeeper.root.logger和log4j.appender.ROLLINGFILE两项，修改为如下内容：
```
zookeeper.root.logger=INFO,ROLLINGFILE
log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
```
&emsp;4) 添加到环境变量：
```
vi /etc/profile
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.4.13
export PATH=$ZOOKEEPER_HOME/bin:$PATH
source /etc/profile
```
&emsp;5) 为了方便起见，将已配置好的整个zookeeper安装文件夹拷贝给其他结点：
```
scp -r /usr/local/zookeeper-3.4.13/ cluster2:/usr/local/
scp -r /usr/local/zookeeper-3.4.13/ cluster3:/usr/local/
```
&emsp;5) 在三台机器上新建数据和日志的存放目录：
```
mkdir -p /home/hadoop_files/hadoop_data/zookeeper
mkdir -p /home/hadoop_files/hadoop_logs/zookeeper/dataLog
mkdir -p /home/hadoop_files/hadoop_logs/zookeeper/logs
# 赋予目录的读写权限给hadoop用户
chown -R hadoop:hadoop /home/hadoop_files
chown -R hadoop:hadoop /usr/local/zookeeper-3.4.13
```
&emsp;5) 在三台机器上新建一个myid文件，分别对应写入1、2、3：
```
# cluster1的myid写入1
echo "1" >> /home/hadoop_files/hadoop_data/zookeeper/myid
# cluster2的myid写入2
echo "2" >> /home/hadoop_files/hadoop_data/zookeeper/myid
# cluster3的myid写入3
echo "3" >> /home/hadoop_files/hadoop_data/zookeeper/myid
```
&emsp;6) 启动zookeeper服务器，按myid的顺序启动：
```
zkServer.sh start
```
jps命令查看zookeeper进程，如果看到“QuorumPeerMain”说明zookeeper启动成功：

<center>
    <img style="width:26%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\zookeeper.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">zookeeper进程</div>
</center>

查看zookeeper占用端口2181的情况：
```
netstat -tnlp | grep :2181
```
当三台全部都启动后，可以查看zookeeper状态：
```
zkServer.sh status
```
有一个是leader，其余是follow，说明zookeeper集群启动成功：
<center>
    <img style="width:54%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\zookeeper_status.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">zookeeper leader</div>
</center>

<center>
    <img style="width:54%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\zookeeper_status2.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">zookeeper follow</div>
</center>

关闭zookeeper：
```
zkServer.sh stop
```
&emsp;7) 如果启动或运行过程中出现问题，要养成从日志分析问题的能力。查看zookeeper日志：
```
vi /home/hadoop_files/hadoop_logs/zookeeper/logs/zookeeper.log
```

<span id = "anchor5">&emsp;</span>

## 安装Kafka

&emsp;三台机器都安装，下面以在cluster1上安装为例。

&emsp;上传文件“kafka_2.12-2.1.1.tgz”至/usr/local路径下，并解压：
```
tar -zxvf kafka_2.12-2.1.1.tgz
```
&emsp;1) 修改kafka配置文件server.properties：
```
vi /usr/local/kafka_2.12-2.1.1/config/server.properties
```
找到文件中的broker.id、log.dirs、zookeeper.connect、listeners和advertised.listeners几项，修改为如下内容：
```
broker.id=1
log.dirs=/home/hadoop_files/hadoop_logs/kafka
zookeeper.connect=cluster1:2181,cluster2:2181,cluster3:2181
listeners=PLAINTEXT://cluster1:9092
advertised.listeners=PLAINTEXT://cluster1:9092
```
&emsp;注：修改listeners和advertised.listeners两项是因为kafka会默认使用localhost监听端口，如果使用云服务器搭建集群，由于云服务器的localhost与本机地址并不完全相同，kafka通信很可能会因此出现问题。

&emsp;2) 添加到环境变量：
```
vi /etc/profile
export KAFKA_HOME=/usr/local/kafka_2.12-2.1.1
export PATH=$KAFKA_HOME/bin:$PATH
source /etc/profile
```
&emsp;2) 拷贝至cluster2和cluster3
```
scp -r /usr/local/kafka_2.12-2.1.1/ cluster2:/usr/local/
scp -r /usr/local/kafka_2.12-2.1.1/ cluster3:/usr/local/
```
&emsp;2) 分别修改cluster2和cluster3配置文件server.properties中的broker.id：
```
# cluster2的broker.id
broker.id=2

# cluster3的broker.id
broker.id=3
```

&emsp;2) 创建kafka数据存放目录，并赋予目录权限给hadoop用户：
```
# 创建kafka数据存放目录，log.dirs是kafka数据存放目录，而非日志存放目录
mkdir -p /home/hadoop_files/hadoop_logs/kafka

# 赋予目录权限
chown -R hadoop:hadoop /home/hadoop_files
chown -R hadoop:hadoop /home/hadoop_files/hadoop_logs/kafka
chown -R hadoop:hadoop /usr/local/kafka_2.12-2.1.1
```

&emsp;2) 启动kafka（启动kafka之前要先启动zookeeper），启动后用jps命令查看进程，如果看到“Kafka”说明kafka启动成功了。
```
zkServer.sh start
kafka-server-start.sh /usr/local/kafka_2.12-2.1.1/config/server.properties &
```
&emsp;至此，kafka安装完毕。

&emsp;2) 下面开始使用kafka。

创建主题：
```
kafka-topics.sh --create --zookeeper cluster1:2181,cluster2:2181,cluster3:2181 --replication-factor 3 --partitions 1 --topic mykafka
```
&emsp;注：参数“--replication-factor 3”和“--partitions 1”表示主题有3个副本，1个分区：
+ 副本位于集群中不同的broker上，也就是说副本的数量不能超过broker的数量，否则创建主题时会失败。副本数默认为1。
+ 生产者可以将消息放到指定分区，消费者则会从指定分区读取消息，这样的机制有效地提升了kafka的吞吐量。内部使用轮询或hash算法实现分区。分区数默认为1。

查看主题：
```
kafka-topics.sh --list --zookeeper cluster1:2181,cluster2:2181,cluster3:2181
```
查看详细信息：
```
kafka-topics.sh --describe --zookeeper cluster1:2181,cluster2:2181,cluster3:2181
```
在cluster1上启动一个生产者：
```
kafka-console-producer.sh --broker-list cluster1:9092 --topic mykafka
```
在cluster2启动一个消费者。参数“--bootstrap-server”指定要监听的地址及端口，可以监听多台机器：
```
kafka-console-consumer.sh --bootstrap-server cluster2:9092 --topic mykafka --from-beginning
kafka-console-consumer.sh --bootstrap-server cluster1:9092,cluster2:9092,cluster3:9092 --topic mykafka --from-beginning
```
cluster1的生产者发送消息，在cluster2的消费者端可以接收到消息：
<center>
    <img style="width:100%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\kafka_producer.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">kafka生产者</div>
</center>

<center>
    <img style="width:100%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\kafka_consumer.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">kafka消费者</div>
</center>

关闭kafka：
```
kafka-server-stop.sh
```
删除主题：
+ 先在配置文件server.properties中添加一行：
```
delete.topic.enable=true
```
+ 删除指定主题，这里只是将主题标记为删除，主题数据实际没有删除：
```
kafka-topics.sh --delete --zookeeper cluster1:2181,cluster2:2181,cluster3:2181 --topic mykafka
```
+ 删除主题对应的数据存放目录。如果路径下不存在主题相关的文件夹，说明创建主题时没有在该broker上创建副本：
```
kafka-server-stop.sh
rm -rf * /home/hadoop_files/hadoop_logs/kafka/mykafka-0
```
+ 登录zookeeper客户端，删除zookeeper中余留的主题相关信息：
```
# 登录zookeeper客户端
zkCli.sh
```
```
# 显示主题信息
ls /brokers/topics
# 删除主题相关信息，同时主题被标记为删除
rmr /brokers/topics/mykafka
# 显示被标记为删除的主题
ls /admin/delete_topics
# 删除标记
rmr /admin/delete_topics/mykafka
```

<span id = "anchor6">&emsp;</span>

## 安装Hadoop

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;上传文件“hadoop-3.1.2.tar.gz”至/usr/local路径下，并解压：
```
tar -zxvf hadoop-3.1.2.tar.gz
```
&emsp;1) 修改脚本文件hadoop-env.sh：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/hadoop-env.sh
```
找到hadoop-env.sh文件中的JAVA_HOME和HADOOP_PID_DIR两项，修改为如下内容：
```
export JAVA_HOME=/usr/local/jdk1.8.0_201
export HADOOP_PID_DIR=/home/hadoop_files
```
&emsp;2) 修改脚本文件mapred-env.sh：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/mapred-env.sh
```
mapred-env.sh文件中配置环境变量HADOOP_MAPRED_PID_DIR，在mapred-env.sh文件中添加一行：
```
export HADOOP_MAPRED_PID_DIR=/home/hadoop_files
```
&emsp;3) 修改配置文件core-site.xml：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/core-site.xml
```
找到文件core-site.xml中的configuration标签，添加以下内容：
```
<configuration>
  <!-- 指定hdfs的nameservices名称为mycluster，与hdfs-site.xml的HA配置相同 -->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://cluster1:9000</value>
  </property>
	
  <!-- 指定缓存文件存储的路径 -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hadoop_files/hadoop_tmp/hadoop/data/tmp</value>
  </property>

  <!-- 配置hdfs文件被永久删除前保留的时间（单位：分钟），默认值为0表明垃圾回收站功能关闭 -->
  <property>
    <name>fs.trash.interval</name>
    <value>1440</value>
  </property>
  
  <!-- 指定zookeeper地址，配置HA时需要 -->
  <property>
    <name>ha.zookeeper.quorum</name>
    <value>cluster1:2181,cluster2:2181,cluster3:2181</value>
  </property>
</configuration>
```
&emsp;4) 修改配置文件hdfs-site.xml：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/hdfs-site.xml
```
找到文件hdfs-site.xml中的configuration标签，添加以下内容：
```
<configuration>
  <!-- 指定hdfs元数据存储的路径 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/home/hadoop_files/hadoop_data/hadoop/namenode</value>
  </property>
  <!-- 指定hdfs数据存储的路径 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/hadoop_files/hadoop_data/hadoop/datanode</value>
  </property>

  <property>
    <name>dfs.secondary.http.address</name>
    <value>cluster1:50090</value>
  </property>

  <!-- 数据备份的个数 -->
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>

  <!-- 关闭权限验证 -->
  <property>
    <name>dfs.permissions.enabled</name>
    <value>false</value>
  </property>

  <!-- 开启WebHDFS功能（基于REST的接口服务） -->
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
</configuration>
```
&emsp;5) 修改配置文件mapred-site.xml：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/mapred-site.xml
```
找到文件mapred-site.xml中的configuration标签，添加以下内容：
```
<configuration>
  <!-- 指定MapReduce计算框架使用YARN -->
  <property>
    <name>mapreduce.framework.name</name>

    <value>yarn</value>
  </property>

  <!-- 指定jobhistory server的rpc地址 -->
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>cluster1:10020</value>
  </property>

  <!-- 指定jobhistory server的http地址 -->
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>cluster1:19888</value>
  </property>
</configuration>
```
&emsp;6) 修改配置文件yarn-site.xml：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/yarn-site.xml
```
找到文件yarn-site.xml中的configuration标签，添加以下内容：
```
<configuration>
  <!-- NodeManager上运行的附属服务，需配置成mapreduce_shuffle才可运行MapReduce程序 -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>

  <!-- 配置Web Application Proxy安全代理（防止yarn被攻击） -->
  <property>
    <name>yarn.web-proxy.address</name>
    <value>cluster2:8888</value>
  </property>
  
  <!-- 开启日志 -->
  <property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
  </property>

  <!-- 配置日志删除时间为7天，-1为禁用，单位为秒 -->
  <property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
  </property>

  <!-- 修改日志目录 -->
  <property>
    <name>yarn.nodemanager.remote-app-log-dir</name>
    <value>/home/hadoop_files/hadoop_logs/yarn</value>
  </property>
  <property>
   <name>yarn.resourcemanager.address</name>
    <value>cluster1:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>cluster1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>cluster1:8031</value>
  </property>
</configuration>
```
&emsp;7) 修改文件workers（旧版本hadoop的文件名是slaves）：
```
vi /usr/local/hadoop-3.1.2/etc/hadoop/workers
```
将localhost替换为：
```
cluster1
cluster2
cluster3
```
&emsp;8) 创建目录，并赋予权限：
```
mkdir -p /home/hadoop_files/hadoop_data/hadoop/namenode
mkdir -p /home/hadoop_files/hadoop_data/hadoop/datanode
mkdir -p /home/hadoop_files/hadoop_tmp/hadoop/data/tmp
mkdir -p /home/hadoop_files/hadoop_logs/yarn
mkdir -p /usr/local/hadoop-3.1.2/logs

# 赋予权限
chown -R hadoop:hadoop /home/hadoop_files/
chown -R hadoop:hadoop /usr/local/hadoop-3.1.2/
```
&emsp;9) 拷贝hadoop工作目录到其它节点
```
scp -r /usr/local/hadoop-3.1.2 cluster2:/usr/local/
scp -r /usr/local/hadoop-3.1.2 cluster3:/usr/local/
```
&emsp;10) 添加到环境变量
```
vi /etc/profile
export HADOOP_HOME=/usr/local/hadoop-3.1.2
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
source /etc/profile
```
&emsp;11) 格式化hdfs
```
su hadoop
zkServer.sh start

#启动journalnode，三台机器都执行
hdfs --daemon start journalnode
#格式化hdfs，只在cluster1上执行
hdfs namenode -format
#关闭journalnode，三台机器都执行
hdfs --daemon stop journalnode
```
&emsp;12) 启动hadoop（启动hadoop之前应先启动zookeeper），只在cluster1上执行：
```
#启动hdfs
start-dfs.sh
#启动yarn
start-yarn.sh
```
使用jps命令查看进程，如果三台机器的进程情况和下面一样，说明hadoop启动成功（NodeManager和ResourceManager是yarn的工作进程）：
```
#cluster1的进程
NodeManager
SecondaryNameNode
ResourceManager
DataNode
NameNode
#cluster2的进程
WebAppProxyServer
DataNode
NodeManager
#cluster3的进程
DataNode
NodeManager
```
启动成功后，在本地主机浏览器中输入网址“192.168.61.130:50090”，打开secondary页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop_secondary_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">secondary页面</div>
</center>
输入网址“192.168.61.130:9870”，打开hdfs管理页面（旧版hadoop的hdfs管理页面端口为50070）：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop_hdfs_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">hdfs管理页面</div>
</center>
输入网址“192.168.61.130:8088”，打开yarn任务管理页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop_yarn_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">yarn任务管理页面</div>
</center>

&emsp;13) 关闭hadoop
```
#关闭yarn
stop-yarn.sh
#关闭hdfs
stop-dfs.sh
```
&emsp;14) 下面介绍hadoop简单操作hdfs数据。

&emsp;hadoop命令有两类：hadoop fs和hdfs dfs：
1. hadoop fs是通用的文件系统命令，针对任何系统，比如本地文件、HDFS文件、HFTP文件、S3文件系统等
2. hdfs dfs是只针对hdfs文件系统相关操作的命令，功能参数基本和hadoop fs一致。用于替代旧版的hadoop dfs命令，新版中hadoop dfs命令已经不推荐使用
+ 查看根目录下的所有文件和文件夹：
```
hadoop fs -ls /
```
+ 新建文件夹：
```
hadoop fs -mkdir /test
hadoop fs -mkdir /test/input
```
+ 上传本地文件到hdfs：
```
hadoop fs -put /usr/local/hadoop-3.1.2/LICENSE.txt /test
```
+ 下载hdfs文件到本地：
```
hadoop fs -get /test/LICENSE.txt /usr/local/src
```
+ 查看文件：
```
hadoop fs -cat /test/LICENSE.txt
```
+ 复制文件：
```
hadoop fs -cp /test/LICENSE.txt /test/input
```
+ 移动文件（也可用于重命名文件）：
```
hadoop fs -mv /test/LICENSE.txt /test/LICENSE.txt.bk
```
+ 删除文件：
```
hadoop fs -rm /test/input/LICENSE.txt
```
+ 删除目录（hadoop fs -rmr已经不推荐使用）：
```
hadoop fs -rm -r /test
```
+ 查看hdfs空间使用情况：
```
hdfs dfsadmin -report
```
&emsp;另外，还有一个命令hadoop fsck用于检查hdfs文件系统的健康状况：
```
#检查整个文件系统的健康状况。不带参数表示只打印检查结果报告
hadoop fsck /
#打印被检查文件或目录的名称、块信息、位置信息、机架信息
hadoop fsck / -files -blocks -locations -racks
#删除损坏的文件
hadoop fsck / -delete
#的去损坏的文件到/lost+found目录下
hadoop fsck / -move
```

&emsp;15) 运行hadoop自带示例wordcount：

&emsp;数据准备：
```
echo "my name is xi jiawei and 27 years old. " > /usr/local/src/input.txt
hadoop fs -mkdir /test
hadoop fs -mkdir /test/input
hadoop fs -put /usr/local/src/input.txt /test/input
```
&emsp;提交到集群执行，输出结果保存到hdfs://cluster1:9000/test/output路径下：
```
hadoop jar /usr/local/hadoop-3.1.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.2.jar wordcount /test/input /test/output
```
&emsp;打开yarn任务管理页面，可以看到刚执行的任务，最终状态显示“SUCCEEDED”表示执行成功：
<center>
    <img style="width:80%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop_yarn_web2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">yarn任务执行情况</div>
</center>
&emsp;查看输出结果：
```
hadoop fs -ls /test/output/
hadoop fs -cat /test/output/part-r-00000
```
&emsp;
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop_yarn_output.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">hadoop自带例子wordcount输出结果截图</div>
</center>
&emsp;16) 最后，这里我记录下一些常见的hadoop问题和解决办法。
+ 如果hdfs写入数据时报错
```
org.apache.hadoop.ipc.RemoteException(java.io.IOException): File xx._COPYING_ could only be written to 0 of the 1 minReplication nodes. There are 0 datanode(s) running and 0 node(s) are excluded in this operation.
```
解决方案：

&emsp;&emsp;&emsp;这个问题一般是由于使用“hdfs namenode -format”格式化多次，导致spaceID不一致造成的。

&emsp;&emsp;&emsp;解决办法就是先将“hadoop.tmp.dir”、“dfs.namenode.name.dir”和“dfs.datanode.data.dir”对应的目录清空
```
rm -rf /home/hadoop_files/hadoop_tmp/hadoop/data/tmp/*
rm -rf /home/hadoop_files/hadoop_data/hadoop/namenode/*
rm -rf /home/hadoop_files/hadoop_data/hadoop/datanode/*
```
然后再重新格式化
```
#启动journalnode，三台机器都执行
hdfs --daemon start journalnode
#格式化hdfs，只在cluster1上执行
hdfs namenode -format
#关闭journalnode，三台机器都执行
hdfs --daemon stop journalnode
```
<span id = "anchor7">&emsp;</span>

## 安装HBase

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;上传文件“zookeeper-3.4.13.tar.gz”至/usr/local路径下，并解压：
```
tar -zxvf hbase-2.1.3-bin.tar.gz
```
&emsp;1) 修改脚本文件hbase-env.sh：
```
vi /usr/local/hbase-2.1.3/conf/hbase-env.sh
```
&emsp;&emsp;在hadoop-env.sh文件中配置环境变量，修改hadoop-env.sh文件中的下面几项，如果没有则新添加进去（HADOOP_HOME是新添的项）：
```
export JAVA_HOME=/usr/local/jdk1.8.0_201
export HADOOP_HOME=/usr/local/hadoop-3.1.2
export HBASE_LOG_DIR=/home/hadoop_files/hadoop_logs/hbase/logs
export HBASE_MANAGES_ZK=false
export HBASE_PID_DIR=/home/hadoop_files
```
&emsp;2) 修改配置文件hbase-site.xml：
```
vi /usr/local/hbase-2.1.3/conf/hbase-site.xml
```
&emsp;&emsp;找到文件hbase-site.xml中的configuration标签，添加以下内容：
```
<configuration>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://cluster1:9000/hbase</value>
        </property>
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
        <property>
            <name>hbase.master</name>
            <value>60000</value>
        </property>
        <property>
            <name>hbase.tmp.dir</name>
            <value>/home/hadoop_files/hadoop_tmp/hbase/tmp</value>
        </property>
        <property>
            <name>hbase.zookeeper.quorum</name>
            <value>cluster1,cluster2,cluster3</value>
        </property>
        <property>
            <name>hbase.zookeeper.property.dataDir</name>
            <value>/home/hadoop_files/hadoop_data/zookeeper</value>
        </property>
        <property>
            <name>hbase.zookeeper.property.clientPort</name>
            <value>2181</value>
        </property>
        <property>
            <name>zookeeper.session.timeout</name>
            <value>120000</value>
        </property>
        <property>
            <name>hbase.regionserver.restart.on.zk.expire</name>
            <value>true</value>
        </property>
        <property>
            <name>hbase.master.info.port</name>
            <value>60010</value>
        </property>
</configuration>
```
&emsp;3) 修改文件regionservers：
```
vi /usr/local/hbase-2.1.3/conf/regionservers
```
将localhost替换为：
```
cluster1
cluster2
cluster3
```
&emsp;4) 删除或重命令slf4j-log4j12-1.7.25.jar文件，避免冲突
```
cd /usr/local/hbase-2.1.3/lib/client-facing-thirdparty
#重命名
mv slf4j-log4j12-1.7.25.jar slf4j-log4j12-1.7.25.jar.bk
```
&emsp;5) 拷贝至其他节点
```
scp -r /usr/local/hbase-2.1.3/ cluster2:/usr/local/
scp -r /usr/local/hbase-2.1.3/ cluster3:/usr/local/
```
&emsp;6) 新建目录，并赋予权限
```
mkdir -p /home/hadoop_files/hadoop_tmp/hbase/tmp
mkdir -p /home/hadoop_files/hadoop_logs/hbase/logs

#赋予权限
chown -R hadoop:hadoop /usr/local/hbase-2.1.3
chown -R hadoop:hadoop /home/hadoop_files
```
&emsp;7) 添加到环境变量：
```
vi /etc/profile
export HBASE_HOME=/usr/local/hbase-2.1.3
export PATH=$HBASE_HOME/bin:$PATH
source /etc/profile
```
&emsp;8) 启动hbase（启动hbase之前应先启动hadoop），只在cluster1上执行：
```
zkServer.sh start
start-dfs.sh
start-yarn.sh

start-hbase.sh
```
&emsp;&emsp;使用jps命令查看进程，如果看到cluster1的进程中有“HMaster”和“HRegionServer”，cluster2和cluster的进程中有“HRegionServer”，说明hbase启动成功。

&emsp;&emsp;启动成功后，在本地主机浏览器中输入网址“192.168.61.130:60010”，打开hbase管理页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hbase_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">hbase管理页面</div>
</center>
&emsp;9) 关闭hbase：
```
stop-hbase.sh

stop-yarn.sh
stop-dfs.sh
```
&emsp;&emsp;如果关闭hbase的过程很慢，一直点点点，那就各节点单独关闭hbase：
```
#单节点关闭RegionServer
/usr/local/hbase-2.1.3/bin/hbase-daemon.sh stop regionserver RegionServer
#在cluster1上关闭hbase主节点
/usr/local/hbase-2.1.3/bin/hbase-daemon.sh stop master
```
&emsp;10) 下面介绍hbase的简单操作。

+ 进入hbase shell：
```
hbase shell
```
+ 查看所有表
```
list
```
+ 创建表，create <table>, {NAME => <family>, VERSIONS => <VERSIONS>}
```
create 'users','basicInfo'
```
+ 添加列簇'educationInfo'
```
alter 'users',NAME => 'educationInfo'
```
+ 添加多个列簇'educationInfo'和'insterestInfo'（注意添加多个要加花括号，添加单个可以不加）
```
alter 'users',{NAME => 'educationInfo'},{NAME => 'insterestInfo'}
```
+ 删除列簇
```
alter 'users', NAME => 'insterestInfo', METHOD => 'delete'
或
alter 'users', 'delete' => 'insterestInfo'
```
+ 插入记录，put <table>,<rowkey>,<family:column>,<value>
```
put 'users','18140039','basicInfo:name','xjw'
put 'users','18140039','basicInfo:birthday','0921'
put 'users','18140039','educationInfo:school','bjtu'
```
+删除记录，delete <table>,<rowkey>,<family:column>
```
delete 'users','18140039','educationInfo:school'
```
+ 查询记录，get <table>,<rowkey>,[<family:column>,....]
```
get 'users','18140039'
```
+ 浏览全表
```
scan 'users'
```
+ 范围查询
```
scan 'users' , {STARTROW => ‘18140039’}
```
+ 查看表结构
```
describe 'users'
```
+ 删除表
```
disable 'users'
drop 'users'
```
hbase操作数据库示例：
<center>
    <img style="width:90%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hbase.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">hbase操作数据库示例截图</div>
</center>
&emsp;11) 最后，这里我记录下一些常见的hbase问题和解决办法。
+ 如果操作数据库时报错
```
ERROR: org.apache.hadoop.hbase.ipc.ServerNotRunningYetException: Server is not running yet
```
解决方案：

&emsp;&emsp;&emsp;查看hbase的master日志，发现启动hbase master时一直提示：
```
INFO  [master/cluster1:16000:becomeActiveMaster] util.FSUtils: Waiting for dfs to exit safe mode...
```
&emsp;&emsp;&emsp;问题就出在这！因为hadoop刚启动不久，hdfs还处于安全模式中，hbase无法对其进行操作，只能用“hadoop fs -ls”命令查看数据，甚至连“hdfs dfs”命令也无法使用。所以应该等hdfs自动退出安全模式，或者我们手动将hdfs退出安全模式，然后再重启hbase就不会有这个问题了：
```
hdfs dfsadmin -safemode leave
```
+ 如果数据库写操作时报错
```
ERROR: org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
```
解决方案：

&emsp;&emsp;&emsp;查看hbase的master日志，发现启动hbase时连接zookeeper报这样一条的信息，虽然不是Error信息：
```
INFO  [ReadOnlyZKClient-cluster1:2181,cluster2:2181,cluster3:2181@0x69c9c441-EventThread] zookeeper.ClientCnxn: EventThread shut down for session: 0x100001d429b000b
```
这是由于中途一台机器重新安装zookeeper导致zookeeper的版本信息不一致问题，解决办法就是将zookeeper的版本信息删除，重启zookeeper
```
rm -rf /home/hadoop_files/hadoop_data/zookeeper/version-2
rm -rf /home/hadoop_files/hadoop_logs/zookeeper/dataLog/version-2
```
<span id = "anchor8">&emsp;</span>

## 安装Hive

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;安装hive之前先在mysql中新建hive数据库和远程用户hive：
+ 新建hive数据库
```
mysql> create database hive;
```
+ 创建hive用户。mysql8.0.15版本不再支持“grant all privileges on *.* to 'hive'@'%' identified by 'hive' with grant option;”来授权用户，必须分两步执行，先创建用户，再赋予权限：
```
#创建用户
mysql> create user hive@'%' identified by 'hive';
#赋予权限
mysql> grant all privileges on *.* to 'hive'@'%' with grant option;
mysql> flush privileges;
```

&emsp;准备好mysql数据库，下面开始安装hive。

&emsp;上传文件“zookeeper-3.4.13.tar.gz”至/usr/local路径下，并解压：
```
tar -zxvf apache-hive-2.3.4-bin.tar.gz
```
&emsp;1) 编辑配置文件hive-site.xml。先将“hive-default.xml.template”复制一份，改名称为“hive-site.xml”：
```
cp /usr/local/apache-hive-2.3.4-bin/conf/hive-default.xml.template /usr/local/apache-hive-2.3.4-bin/conf/hive-site.xml
vi /usr/local/apache-hive-2.3.4-bin/conf/hive-site.xml
```
&emsp;&emsp;找到hive-site.xml文件中以下几项，修改为如下内容（如果linux上不方便操作，可以先在window上编辑然后上传到虚拟机）：
```
  <property> 
   <name>javax.jdo.option.ConnectionURL </name> 
   <value>jdbc:mysql://cluster2:3306/hive</value> 
  </property> 
  <property> 
   <name>javax.jdo.option.ConnectionDriverName </name> 
   <value>com.mysql.cj.jdbc.Driver </value> 
  </property>
  <property> 
   <name>javax.jdo.option.ConnectionPassword </name> 
   <value>hive</value> 
  </property> 
  <property> 
    <name>hive.hwi.listen.port </name> 
    <value>9999</value> 
    <description>This is the port the Hive Web Interface will listen on</description> 
  </property> 
  <property>
    <name>datanucleus.autoCreateSchema</name>
    <value>true</value>
  </property>
  <property>
    <name>datanucleus.fixedDatastore</name>
    <value>false</value>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hive</value>
    <description>Username to use against metastore database</description>
  </property>
  <property>
    <name>hive.exec.local.scratchdir</name>
    <value>/home/hadoop_files/hadoop_tmp/hive/iotmp</value>
    <description>Local scratch space for Hive jobs</description>
  </property>
  <property>
    <name>hive.downloaded.resources.dir</name>
    <value>/home/hadoop_files/hadoop_tmp/hive/iotmp</value>
    <description>Temporary local directory for added resources in the remote file system.</description>
  </property>
  <property>
    <name>hive.querylog.location</name>
    <value>/home/hadoop_files/hadoop_logs/hive/querylog</value>
    <description>Location of Hive run time structured log file</description>
  </property>
```
&emsp;2) 上传mysql链接器“mysql-connector-java-8.0.15.jar”到hive库目录“/usr/local/apache-hive-2.3.4-bin/lib/”下。

&emsp;3) 拷贝hive的jline包到hadoop目录下，避免冲突：
```
cp /usr/local/apache-hive-2.3.4-bin/lib/jline-2.12.jar /usr/local/hadoop-3.1.2/share/hadoop/yarn/lib/
```
如果“/usr/local/hadoop-3.1.2/share/hadoop/yarn/lib/”路径下还有其他版本的jline包，改名，使其无效：
```
cd /usr/local/hadoop-3.1.2/share/hadoop/yarn/lib/
mv jline-0.9.94.jar jline-0.9.94.jar.bak
```
&emsp;3) 添加到环境变量：
```
vi /etc/profile
export HIVE_HOME=/usr/local/apache-hive-2.3.4-bin
export PATH=$HIVE_HOME/bin:$HIVE_HOME/conf:$PATH
source /etc/profile
```
&emsp;4) 创建目录，并赋予权限：
```
mkdir -p /home/hadoop_files/hadoop_tmp/hive/iotmp
mkdir -p /home/hadoop_files/hadoop_logs/hive/querylog
#赋予权限
chown -R hadoop:hadoop /home/hadoop_files/
chown -R hadoop:hadoop /usr/local/apache-hive-2.3.4-bin
```
&emsp;5) 初始化hive元数据，也就是将hive与mysql同步：
```
schematool -initSchema -dbType mysql
```
&emsp;注：初始化元数据是必须执行的，否则会在操作时hive报错“MetaException(message:Version information not found in metastore. )”

&emsp;6) 拷贝至其他节点：
```
scp -r /usr/local/apache-hive-2.3.4-bin/ cluster2:/usr/local/
scp -r /usr/local/apache-hive-2.3.4-bin/ cluster3:/usr/local/
```
&emsp;7) 进入hive shell（要先启动hdfs和mysql才能操作hive）
```
#启动hdfs
su hadoop
zkServer.sh start
start-dfs.sh
start-yarn.sh

#启动hive客户端，进入hive shell
hive
```
&emsp;8) hive采用类sql的语言Hive QL，所以hive可以按sql规范进行增删改查：
```
hive> show databases;
hive> show tables;
hive> create table users(id int, name string);
hive> insert into users values(1,'ly');
hive> select * from users;
hive> drop table users;
```
&emsp;hive执行sql语句示例：
<center>
    <img style="width:100%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hive.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">hive执行sql语句示例截图</div>
</center>
&emsp;从截图中可以看到，hive执行插入语句时，hadoop会进行map/reduce操作。

<span id = "anchor9">&emsp;</span>

## 安装Spark

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;因为spark基于scala语言编写的，所以安装spark之前要先安装scala。

<span id = "anchor9_1"></span>

### 安装Scala

&emsp;上传文件“scala-2.12.8.tgz”至/usr/local路径下，并解压：
```
tar -zxvf scala-2.12.8.tgz
```
&emsp;添加到环境变量：
```
vi /etc/profile
export SCALA_HOME=/usr/local/scala-2.12.8
export PATH=$SCALA_HOME/bin:$PATH
source /etc/profile
```
&emsp;查看scala版本：
```
scala -version
```
&emsp;拷贝至其他节点：
```
scp -r /usr/local/scala-2.12.8/ cluster2:/usr/local/
scp -r /usr/local/scala-2.12.8/ cluster3:/usr/local/
```
&emsp;赋予目录权限：
```
chown -R hadoop:hadoop /usr/local/scala-2.12.8
```

<span id = "anchor9_2"></span>

### 安装Spark

&emsp;上传文件“spark-2.4.0-bin-hadoop2.7.tgz”至/usr/local路径下，并解压：
```
tar -zxvf spark-2.4.0-bin-hadoop2.7.tgz
```
&emsp;1) 编辑脚本文件spark-env.sh。先将“spark-env.sh.template”复制一份，改名称为“spark-env.sh”：
```
cp /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh.template /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh
vi /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-env.sh
```
&emsp;&emsp;在spark-env.sh文件中配置环境变量，在spark-env.sh文件中加入以下7行：
```
export JAVA_HOME=/usr/local/jdk1.8.0_201
export SCALA_HOME=/usr/local/scala-2.12.8
export SPARK_MASTER_IP=cluster1
export HADOOP_CONF_DIR=/usr/local/hadoop-3.1.2/etc/hadoop
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop-3.1.2/bin/hadoop classpath)
export SPARK_CLASSPATH=$HIVE_HOME/lib/mysql-connector-java-8.0.15.jar
export SPARK_PID_DIR=/home/hadoop_files
```
&emsp;2) 在路径“/usr/local/spark-2.4.0-bin-hadoop2.7/conf/”下新建slaves文件：
```
vi /usr/local/spark-2.4.0-bin-hadoop2.7/conf/slaves
```
向slaves文件填入以下内容：
```
cluster1
cluster2
cluster3
```
&emsp;3) 如果要在spark中使用hive，可使用spark的接口HiveContext，而且要有hive-site.xml。要把hive/conf/hive-site.xml文件拷贝到$SPARK_HOME/conf路径下：
```
cp /usr/local/apache-hive-2.3.4-bin/conf/hive-site.xml /usr/local/spark-2.4.0-bin-hadoop2.7/conf/
```
注：HiveContext在基本的SQLContext上有了一些新的特性，可以用Hive QL写查询，可以读取Hive表中的数据，支持Hive的UDF

&emsp;4) 把hadoop的hdfs-site.xml和core-site.xml文件拷贝到$SPARK_HOME/conf路径下：
```
cp /usr/local/hadoop-3.1.2/etc/hadoop/hdfs-site.xml /usr/local/hadoop-3.1.2/etc/hadoop/core-site.xml /usr/local/spark-2.4.0-bin-hadoop2.7/conf/
```
&emsp;5) 编辑配置文件spark-defaults.conf。先将“spark-defaults.conf.template”复制一份，改名称为“spark-defaults.conf”：
```
cp /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf.template /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf
vi /usr/local/spark-2.4.0-bin-hadoop2.7/conf/spark-defaults.conf
```
&emsp;&emsp;在spark-defaults.conf文件中添加一行：
```
spark.files file:///usr/local/spark-2.4.0-bin-hadoop2.7/conf/hdfs-site.xml,file:///usr/local/spark-2.4.0-bin-hadoop2.7/conf/core-site.xml
```
&emsp;6) 拷贝至其他节点：
```
scp -r /usr/local/spark-2.4.0-bin-hadoop2.7/ cluster2:/usr/local/
scp -r /usr/local/spark-2.4.0-bin-hadoop2.7/ cluster3:/usr/local/
```
&emsp;7) 添加到环境变量：

&emsp;&emsp;主节点（cluster1）
```
vi /etc/profile
export SPARK_HOME=/usr/local/spark-2.4.0-bin-hadoop2.7
export PATH=$SPARK_HOME/bin:$PATH
export PATH=$SPARK_HOME/sbin:$PATH
source /etc/profile
```
&emsp;&emsp;从节点（cluster2和cluster3）
```
vi /etc/profile
export SPARK_HOME=/usr/local/spark-2.4.0-bin-hadoop2.7
export PATH=$SPARK_HOME/bin:$PATH
source /etc/profile
```
&emsp;8) 启动spark，只在cluster1上执行：
```
#spark数据通常是存储在hdfs，所以运行spark任务前先启动hdfs
zkServer.sh start
start-dfs.sh

#启动spark
start-master.sh
start-slaves.sh
```
&emsp;&emsp;使用jps命令查看进程，如果看到cluster1的进程中有“Master”和“Worker”，cluster2和cluster的进程中有“Worker”，说明spark启动成功。

&emsp;&emsp;启动成功后，在本地主机浏览器中输入网址“192.168.61.130:8080”，打开spark管理页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\spark_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">spark管理页面</div>
</center>
&emsp;9) 关闭spark
```
stop-slaves.sh
stop-master.sh
```
&emsp;10) 运行spark示例：
+ 运行自带例子SparkPi：
```
spark-submit --master spark://cluster1:7077 /usr/local/spark-2.4.0-bin-hadoop2.7/examples/src/main/python/pi.py
或
run-example SparkPi
```
去除多余的Info日志，截取输出结果信息：
```
run-example SparkPi 2>&1 | grep "Pi is roughly"
```
+ 运行自带例子WordCount：
```
#准备数据
echo "my name is xi jiawei and 27 years old. " > /usr/local/src/input.txt
hdfs dfs -mkdir /test
hdfs dfs -mkdir /test/input
hdfs dfs -put /usr/local/src/input.txt /test/input
```
```
#提交集群运行
spark-submit --master spark://cluster1:7077 /usr/local/spark-2.4.0-bin-hadoop2.7/examples/src/main/python/wordcount.py /test/input/input.txt
```
&emsp;hive运行自带例子WordCount的输出结果如下图所示：
<center>
    <img style="width:30%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\spark.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">spark自带例子WordCount输出结果截图</div>
</center>

<span id = "anchor10">&emsp;</span>

## 安装Storm

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;上传文件“apache-storm-1.2.2.tar.gz”至/usr/local路径下，并解压：
```
tar -zxvf apache-storm-1.2.2.tar.gz
```
&emsp;1) 编辑配置文件storm.yaml。
```
vi /usr/local/apache-storm-1.2.2/conf/storm.yaml
```
找到storm.yaml文件中以下几项，修改成以下内容（注意配置名称前要加空格，否则会出错）：
```
#配置zookeeper服务器
 storm.zookeeper.servers : 
- "cluster1"
- "cluster2"
- "cluster3"
#配置storm主控节点（可以配置多个）
 nimbus.seeds: ["cluster1"]
#配置监听端口
 supervisor.slots.ports:
     - 6700
     - 6701
     - 6702
     - 6703
```
在storm.yaml文件中添加一行（注意前面有一个空格）：
```
 storm.local.dir: "/home/hadoop_files/hadoop_tmp/storm/tmp"
```
&emsp;2) 添加到环境变量：
```
vi /etc/profile
export STORM_HOME=/usr/local/apache-storm-1.2.2
export PATH=$STORM_HOME/bin:$PATH
source /etc/profile
```
&emsp;3) 拷贝至其他节点：
```
scp -r /usr/local/apache-storm-1.2.2/ cluster2:/usr/local/
scp -r /usr/local/apache-storm-1.2.2/ cluster3:/usr/local/
```
&emsp;4) 创建目录，并赋予权限：
```
mkdir -p /home/hadoop_files/hadoop_tmp/storm/tmp
chown -R hadoop:hadoop /home/hadoop_files
chown -R hadoop:hadoop /usr/local/apache-storm-1.2.2
```
&emsp;5) 运行storm前，先查看python版本，如果版本是2.6甚至低于2.6，请安装2.6以上的版本。关于CentOS如何升级python版本可以参照博客“[Linux（CentOS 7）安装python3](https://blog.csdn.net/u012120103/article/details/103932299)”：
```
python -V
```
&emsp;6) 启动storm（启动storm之前应先启动zookeeper）：
```
zkServer.sh start

#开两个cluster1窗口，分别启动nimbus和ui
storm nimbus
storm ui

#cluster2、cluster3各自启动supervisor
storm supervisor

#（非必要）另外再给cluster1、cluster2、cluster3开一个窗口，各自启动logviewer
storm logviewer
```
&emsp;&emsp;使用jps命令查看进程，如果看到cluster1的进程中有“nimbus”和“core”，cluster2和cluster的进程中有“Supervisor”，说明storm启动成功。

&emsp;&emsp;启动成功后，在本地主机浏览器中输入网址“192.168.61.130:8080”，打开storm管理页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">storm管理页面</div>
</center>
&emsp;7) 运行storm示例：

&emsp;&emsp;可以运行storm自带的示例，也可以自己写一个demo。这里我以自己写的简易版WordCount为例，介绍如何提交执行storm任务。

&emsp;&emsp;WordCount代码：
```
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;
import org.apache.storm.utils.Utils;

import java.util.HashMap;
import java.util.Map;

public class WordCount {
    public static class Spout extends BaseRichSpout {
        private static final long serialVersionUID = 1L;
        private SpoutOutputCollector collector;

        private String[] messageArray={
                "my name is xijiawei",
                "and i am 27 years old",
                "i am from a small county of Ji'an, Jiangxi in China"
        };
        private int i=0;

        public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
            this.collector = spoutOutputCollector;
        }

        public void nextTuple() {
            if(i<messageArray.length){
                this.collector.emit(new Values(messageArray[i]));
                i++;
            }
        }

        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            declarer.declare(new Fields("sentence"));
        }
    }

    public static class SplitBolt extends BaseRichBolt {
        private OutputCollector collector;

        public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
            this.collector = outputCollector;
        }

        public void execute(Tuple tuple) {
            String sentence = tuple.getStringByField("sentence");//根据Spout中declareOutputFields()定义的数据格式，接收来自Spout的数据
            String[] words = sentence.split(" ");//分词

            for(String word:words){
                this.collector.emit(new Values(word,1));
            }
        }

        public void declareOutputFields(OutputFieldsDeclarer declarer) {
            declarer.declare(new Fields("word","count"));
        }
    }

    public static class CountBolt extends BaseRichBolt {
        private OutputCollector collector;
        private Map<String,Integer> result = new HashMap<String,Integer>();//定义一个Map集合保存最后的统计结果

        public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
            this.collector = outputCollector;
        }

        public void execute(Tuple tuple) {
            //接收来自SplitBolt的数据
            String word = tuple.getStringByField("word");
            int count = tuple.getIntegerByField("count");

            //判断一个result是否存在该单词
            if(this.result.containsKey(word)){
                //包含该单词
                int total = this.result.get(word);
                this.result.put(word, total+count);
            }else{
                //不存在该单词
                this.result.put(word, count);
            }
        }

        public void declareOutputFields(OutputFieldsDeclarer declarer) {
        }

        @Override
        public void cleanup() {
            for (Map.Entry<String, Integer> entry : this.result.entrySet()) {
                System.out.println(entry.getKey() + ": " + entry.getValue());
            }
        }
    }

    public static void main(String[] args) throws Exception {
        /*构造拓扑
         * 在这里有三层：
         * 1、Spout提供数据源
         * 2、SplitBolt分词
         * 3、CountBolt统计数量*/
        TopologyBuilder builder = new TopologyBuilder();//定义一个拓扑
        builder.setSpout("spout", new Spout());//设置1个Executor（线程），默认1个，builder.setSpout("spout", new demo.Spout(), 1);
        builder.setBolt("splitBolt", new SplitBolt()).setNumTasks(1).shuffleGrouping("spout");//设置1个Executor，2个Task。随机分组，无论Spout发出任何数据，即使发出同样字段的数据时，处理该数据的task是随机的
        builder.setBolt("countBolt", new CountBolt()).setNumTasks(1).fieldsGrouping("splitBolt",new Fields("word"));//设置1个Executor，1个Task。按照字段分组，即同样字段的数据只能发送给一个Task实例处理，这样保证结果正确

        /*配置*/
        Config config = new Config();

        /*提交运行*/
        if (args != null && args.length > 0) {
            //提交到集群运行
            StormSubmitter.submitTopology(args[0], config, builder.createTopology());
        } else {
            //本地模式运行
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology("WordCount", config, builder.createTopology());
            Utils.sleep(1000000);
            cluster.killTopology("WordCount");
            cluster.shutdown();
        }
    }
}
```
&emsp;&emsp;将代码打包成jar文件，上传虚拟机，我这里是上传到cluster3上，提交到集群执行：
```
storm jar storm-demo.jar WordCount wordcount
```
如下图所示，表示提交成功：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">WordCount提交到storm集群执行</div>
</center>
打开storm管理页面，可以看到刚执行的任务：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_web2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">storm管理页面显示刚执行的任务</div>
</center>
点击任务进入任务查看详细情况，有一排按钮，其中有一个“Kill”按钮，用来终结任务：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_topology.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">topology详情页</div>
</center>
往下拉，看到Bolts栏中有一个叫“countBolt”的bolt，最后的统计结果会在这个bolt实例中打印出来：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_topology2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">topology详情页2</div>
</center>
点击“countBolt”查看这个bolt是在哪个机器上执行的，可以看到是在cluster3上执行的，并且记住这个任务的id，待会看任务日志就去cluster3上看：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_topology_bolt.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">countBolt详情页</div>
</center>
因为代码中设置的是countBolt关闭时才打印结果，所以现在我们点击topology详情页的“Kill”按钮，然后就能在cluster3上看到统计结果：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\storm_output.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">WordCount输出结果</div>
</center>

<span id = "anchor11">&emsp;</span>

## 安装Flink

&emsp;三台机器都安装，cluster1为主节点，cluster2、cluster3为从节点。

&emsp;上传文件“flink-1.10.0-bin-scala_2.11.tgz”至/usr/local路径下，并解压：
```
tar -zxvf flink-1.10.0-bin-scala_2.11.tgz
```
&emsp;1) 编辑配置文件flink-conf.yaml。
```
vi /usr/local/flink-1.10.0/conf/flink-conf.yaml
```
找到flink-conf.yaml文件中的“jobmanager.rpc.address”参数，修改成以下内容：
```
#配置jobmanager服务器，即master节点
jobmanager.rpc.address: cluster1
```
&emsp;2) 修改文件slaves。
```
vi /usr/local/flink-1.10.0/conf/slaves
```
内容修改为：
```
cluster2
cluster3
```

&emsp;3) 添加到环境变量：
```
vi /etc/profile
export FLINK_HOME=/usr/local/flink-1.10.0
export PATH=$FLINK_HOME/bin:$PATH
source /etc/profile
```
&emsp;4) 拷贝至其他节点：
```
scp -r /usr/local/flink-1.10.0/ cluster2:/usr/local/
scp -r /usr/local/flink-1.10.0/ cluster3:/usr/local/
```
&emsp;5) 赋予权限：
```
chown -R hadoop:hadoop /usr/local/flink-1.10.0
```
&emsp;6) 启动flink（启动storm之前应先启动zookeeper）：
```
zkServer.sh start

#cluster1
start-cluster.sh
```
&emsp;&emsp;使用jps命令查看进程，如果看到cluster1的进程中有“StandaloneSessionClusterEntrypoint”，cluster2和cluster的进程中有“TaskManagerRunner”，说明flink启动成功。

&emsp;&emsp;启动成功后，在本地主机浏览器中输入网址“192.168.61.130:8081”，打开flink管理页面：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_web.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">flink管理页面</div>
</center>
&emsp;7) 关闭flink：
```
#cluster1
stop-cluster.sh
```
&emsp;8) 运行flink示例：

&emsp;&emsp;可以运行flink自带的示例，也可以自己写一个demo。这里我以自己写的简易版WordCount为例，介绍如何提交执行flink任务。

&emsp;&emsp;WordCount代码：
```
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.api.java.utils.ParameterTool;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class WordCount {
    public static void main(String[] args) throws Exception {
        ParameterTool params = ParameterTool.fromArgs(args);
        String hostname = params.get("hostname");
        int port = params.getInt("port");

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        DataStream<String> stream = env.socketTextStream(hostname, port);

        DataStream<Tuple2<String, Integer>> windowCounts = stream
                .flatMap(new FlatMapFunction<String, Tuple2<String, Integer>>() {
                    @Override
                    public void flatMap(String str, Collector<Tuple2<String, Integer>> collector) {
                        for (String word : str.split("\\s")) {
                            collector.collect(new Tuple2<String, Integer>(word, 1));
                        }
                    }
                })
                .keyBy(0)//以key分组统计
                .timeWindow(Time.seconds(5), Time.seconds(1))//定义一个5s的滑动时间窗口，每1s滑动一次
                .reduce(new ReduceFunction<Tuple2<String, Integer>>() {
                    @Override
                    public Tuple2<String, Integer> reduce(Tuple2<String, Integer> tuple1, Tuple2<String, Integer> tuple2) {
                        return new Tuple2<String, Integer>(tuple1.f0, tuple1.f1+tuple2.f1);
                    }
                });
        windowCounts.print();

        env.execute();
    }
}
```
&emsp;&emsp;在提交执行任务前先在另一虚拟机启动一个tcp监听端口作为数据源的发送端：
```
nc -lk 9999
```
&emsp;&emsp;将代码打包成jar文件，上传虚拟机，我这里是上传到cluster1，提交到集群执行，设置发送端主机名和端口参数：
```
flink run flink-demo.jar --hostname cluster2 --port 9999
```
如下图所示，表示提交成功：
<center>
    <img style="width:73%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">WordCount提交到flink集群执行</div>
</center>
输入命令“flink list”，可以查看当前正在执行的任务：
<center>
    <img style="width:80%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_list.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">命令查看刚执行的任务</div>
</center>
也可以打开flink管理页面，查看刚执行的任务：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_job.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">flink管理页面显示刚执行的任务</div>
</center>
点击任务进入任务查看详细情况，右侧有一个“Cancel Job”按钮，用来终结任务：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_job2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">flink任务详情页</div>
</center>
在cluster2刚打开的数据源发送端，任意输入字符：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_input.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">flink任务详情页</div>
</center>
然后就能在flink管理页面看到输出打印结果：
<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\flink_log_out.PNG">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">WordCount输出结果</div>
</center>