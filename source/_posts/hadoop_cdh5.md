
---
title: hadoop 部署
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: 大数据
tags: ['大数据','hadoop']
---

基于cloudera,oracle jdk 1.8.0.111, centos 6.8

## 准备  

### **环境配置**  

0. 要求：CENTOS6(el6)，主4C8G，从2C4G 20G

1. 关闭selinux

``` 
vi /etc/selinux/config

SELINUX=disabled
```  

2. 配置hostname, host????????????????????

```
vi /etc/sysconfig/network  
vi /etc/hosts  
```

3. ssh打通

```  
ssh-keygen -t rsa  
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  
scp ~/.ssh/authorized_keys root@datanode1:~/.ssh/
(ssh-copy-id hosts)  

```  

4. 主从时间同步ntp
```  
# vim /etc/ntp.conf   
driftfile /var/lib/ntp/drift
restrict 127.0.0.1
restrict -6 ::1
restrict default nomodify notrap 
server ntp3.aliyun.com prefer 
server ntp7.aliyun.com iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys

# 时间同步
service ntpd start
chkconfig ntpd on

# 查看
ntpstat 

synchronised to NTP server (203.107.6.88) at stratum 3 
   time correct to within 40 ms
   polling server every 1024 s

```  

5. 装mysql 
```  
yum install -y mysql mysql-server mysql-devel 
chkconfig mysqld on  
service mysqld start  
create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on *.* to root@"%" Identified by "hadoop";
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'hadoop' WITH GRANT OPTION;   
```  

6. 安装java

注意需要是 oracle java
 
```
#openjdk
rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
#sun jdk
yum -y install oracle-j2sdk1.7.x86_64

vi /etc/profile
# 环境变量
export JAVA_HOME=/opt/java
export PATH=$JAVA_HOME/bin:$PATH
```

[JDBC 参考](http://dev.mysql.com/downloads/connector/j/)


7. 内核缓存配置

`vi /etc/sysctl.conf`  
`vm.swappiness=10`
`vi /etc/rc.local` 
`echo never > /sys/kernel/mm/transparent_hugepage/defrag`

---

### **部署**

1. ????CM.??????????5.9??   

  > Cloudera Manager 5.9??http://archive-primary.cloudera.com/cm5/cm/5/cloudera-manager-el6-cm5.9.0_x86_64.tar.gz  

  > CDH5.9 ???????http://archive-primary.cloudera.com/cdh5/parcels/5.9.0.23/CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel  

  > CDH5.9 sha?????http://archive-primary.cloudera.com/cdh5/parcels/5.9.0.23/CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha1  

  > manifest ?????http://archive-primary.cloudera.com/cdh5/parcels/5.9.0.23/manifest.json  


2. ????????????????  

  - ??? tar -zxvf cloudera-manager-el6-cm5.9.0_x86_64.tar.gz -P /opt/cloudera/  
  - mv CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel /opt/clourdera/parcel.repo/  
  - mv CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha1 CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha
  - mv CDH-5.9.0-1.cdh5.9.0.p0.23-el6.parcel.sha /opt/clourdera/parcel.repo/  
  - mv manifest.json /opt/clourdera/parcel.repo/  

3. ?????????  
```  
/opt/cloudera/share/cmf/schema/scm_prepare_database.sh mysql cm -hlocalhost -uroot -phdp --scm-host localhost scm scm scm
??????????????/usr/share/cmf
```  

4. ????????????????cloudera-daemon, cloudera-server

```
sudo yum --nogpgcheck localinstall cloudera-manager-daemons-*.rpm
sudo yum --nogpgcheck localinstall cloudera-manager-server-*.rpm
```

7. ????localhost:7180???????,????????

??????jdbc???????????oozie hive??????jdbc????????????


### ??????

1. jdk ??????????????????????????????yum repo?????

2. ????????????????  

### ????????  
> [Cloudera Manager 5 ?? CDH5 ????????????????](http://www.aboutyun.com/thread-9086-1-1.html)  
> [Install Cloudera ](https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_server.html)  
> [?????CDH5??????](https://www.jianshu.com/p/57179e03795f)  
> [Cloudera Hadoop???????????](https://blog.csdn.net/qq_30004245/article/details/75735275?locationNum=9&fps=1)  
> [??CDH5??hadoop???](https://zhuanlan.zhihu.com/p/25533211)  
> [??????????Hadoop??????](https://blog.csdn.net/hliq5399/article/details/78193113#t5)
> [CDH5???????????](https://www.jianshu.com/p/72dc1c591647)
