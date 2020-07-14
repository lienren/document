# Hadoop Cluster Deploy & Defend
Hadoop集群部署及维护

### 环境准备
1. Centos 7.X
2. Centos系统语言改为英语
3. JDK1.8及以上（卸载openJDK）
4. 关闭防火墙
   1. systemctl stop firewalld.service（停止firewall）
   2. systemctl disable firewalld.service（禁止firewall开机启动）
5. 所有集群机器准备好mysql-connection.jar
6. mysql数据库开启以下配置
   1. 启用innodb引擎;
   2. set global innodb_large_prefix = ON;
   3. set global innodb_file_format = BARRACUDA;

### 搭建Ambari
[使用Ambari快速部署Hadoop集群](https://developer.aliyun.com/article/706968)
[Ambari 2.7.3 部署指南（CentOS 7.6）](https://support.huaweicloud.com/dpmg-hdp-kunpengbds/kunpengbds_04_0014.html)

> Ambari 自身也是一个分布式架构的软件，主要由两部分组成：Ambari Server 和 Ambari Agent。我们可以通过 Ambari Server 通知 Ambari Agent 安装对应的软件；甚至连Ambari Agent我们都可以在Web界面上来进行安装和部署。

1. 集群所有服务器上运行（包含管理服务器）
```
# ssh-keygen -t rsa
```
> 注意：配置无密码

2. 将Ambari Server的公钥scp到其它节点上
```
# scp .ssh/id_rsa.pub 其它节点IP:/root/.ssh/authorized_keys
```

3. 设置权限（服务器全部设置）
```
# chmod 600 ~/.ssh/authorized_keys
```

4. 测试连接
```
# ssh 其它节点IP
```

5. 安装ambari-server
```
# cd /etc/yum.repos.d/
# wget http://public-repo-1.hortonworks.com/ambari/centos7/2.x/updates/2.7.4.0/ambari.repo
# yum install -y ambari-server
```

6. 设置ambari-server
```
# ambari-server setup
```
> 1) 检测SELinux
> 2) 询问是否自定义用户，默认否-（可以安装完毕之后再进行用户管理。）
> 3) 检测iptables选择JDK版本，默认Oracle JDK 1.8。如果你已经安装了jdk，可以选择自定义jdk。如果你是yum安装的openjdk，那么路径位于/usr/lib/jvm/java-1.8.0-openjdk-xxx.x86_64/jre/目录下。
> 4) 询问是否打开高级的数据库配置，默认-否

7. 启动ambari-server
```
# ambari-server start
```
> 查看监听端口 # netstat -ntlp | grep 8080

8. 访问ambari server
```
http://192.168.56.11:8080/
```

9. 创建集群
> 注意
> 1. 注意使用FQDN名称，并且主机名要能够解析。
   >    - vi /etc/hosts，加入：10.10.141.207 hp207.example.com 141-207
   >    - echo 'domain example.com' >> /etc/resolv.conf
   >    - reboot
   >    - hostname -f
> 2. 配置管理主机 /etc/hosts 文件，加入10.10.141.207 hp207.example.com
> 3. 集群所有服务器需要配置FQDN名称访问
> 4. 重启集群时，YARN DNS报错“ERROR: Cannot set priority of registrydns process 30860”，可能是占用53端口，关闭即可。
> 5. 安装Hive服务器，需要安装mysql或者用已有mysql：
   >    - yum install mysql
   >    - yum list installed | grep mariadb
   >    - yum install mariadb-server
> 6. 安装Spark thrift server：
   >    - iptables -I OUTPUT -p tcp --dport 10016 -j ACCEPT
   >    - iptables -I INPUT -p tcp --dport 10016 -j ACCEPT
