---
title: kubernetes 部署
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: Kubernetes
tags: ['分布式系统','Kubernetes','k8s']
---

## 准备
1. 准备虚拟机，配置环境，关闭防火墙
systemctl disable firewalld  
systemctl stop firewalld 

2. 安装etcd
yum install etcd -y

## 配置
```
#[member]  
ETCD_NAME=etcd1  
ETCD_DATA_DIR="/var/lib/etcd/test.etcd"  
#ETCD_WAL_DIR=""  
#ETCD_SNAPSHOT_COUNT="10000"  
#ETCD_HEARTBEAT_INTERVAL="100"  
#ETCD_ELECTION_TIMEOUT="1000"  
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"  
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379,http://0.0.0.0:4001"  
#ETCD_MAX_SNAPSHOTS="5"  
#ETCD_MAX_WALS="5"  
#ETCD_CORS=""  
#  
#[cluster]  
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://master1:2380"  
# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."  
ETCD_INITIAL_CLUSTER="etcd1=http://master1:2380,etcd2=http://master2:2380,etcd3=http://master3:2380"  
ETCD_INITIAL_CLUSTER_STATE="new"  
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-baby"  
ETCD_ADVERTISE_CLIENT_URLS="http://master1:2379,http://master1:4001" 
```
## 部署
### 安装docker 
```
yum install docker -y  
chkconfig docker on  
service docker start 
````
### 安装k8s 
```
yum install kubernetes -y  
```

- 修改配置 
```
# kubernetes system config  
#  
# The following values are used to configure the kube-apiserver  
#  
  
# The address on the local server to listen to.  
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"  
  
# The port on the local server to listen on.  
KUBE_API_PORT="--port=8080"  
  
# Port minions listen on  
# KUBELET_PORT="--kubelet-port=10250"  
  
# Comma separated list of nodes in the etcd cluster  
KUBE_ETCD_SERVERS="--etcd-servers=http://etcd:2379"  
  
# Address range to use for services  
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"  
  
# default admission control policies  
# KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"  
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"  
  
# Add your own!  
KUBE_API_ARGS=""  
---
###  
# kubernetes system config  
#  
# The following values are used to configure various aspects of all  
# kubernetes services, including  
#  
#   kube-apiserver.service  
#   kube-controller-manager.service  
#   kube-scheduler.service  
#   kubelet.service  
#   kube-proxy.service  
# logging to stderr means we get it in the systemd journal  
KUBE_LOGTOSTDERR="--logtostderr=true"  
  
# journal message level, 0 is debug  
KUBE_LOG_LEVEL="--v=0"  
  
# Should this cluster be allowed to run privileged docker containers  
KUBE_ALLOW_PRIV="--allow-privileged=false"  
  
# How the controller-manager, scheduler, and proxy find the apiserver  
KUBE_MASTER="--master=http://master1:8080"  
```

- 启动 
systemctl enable kube-apiserver  
systemctl start kube-apiserver  
systemctl enable kube-controller-manager  
systemctl start kube-controller-manager  
systemctl enable kube-scheduler  
systemctl start kube-scheduler  


- deploy node
```
yum install docker -y  
chkconfig docker on  
service docker start  
yum install kubernetes -y  
```

- 修改配置
```  
# kubernetes system config  
#  
# The following values are used to configure various aspects of all  
# kubernetes services, including  
#  
#   kube-apiserver.service  
#   kube-controller-manager.service  
#   kube-scheduler.service  
#   kubelet.service  
#   kube-proxy.service  
# logging to stderr means we get it in the systemd journal  
KUBE_LOGTOSTDERR="--logtostderr=true"  
  
# journal message level, 0 is debug  
KUBE_LOG_LEVEL="--v=0"  
  
# Should this cluster be allowed to run privileged docker containers  
KUBE_ALLOW_PRIV="--allow-privileged=false"  
  
# How the controller-manager, scheduler, and proxy find the apiserver  
KUBE_MASTER="--master=http://etcd:8080"  
###  
# kubernetes kubelet (minion) config  
  
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)  
KUBELET_ADDRESS="--address=0.0.0.0"  
  
# The port for the info server to serve on  
# KUBELET_PORT="--port=10250"  
  
# You may leave this blank to use the actual hostname  
KUBELET_HOSTNAME="--hostname-override=node1"  
  
# location of the api-server  
KUBELET_API_SERVER="--api-servers=http://etcd:8080"  
  
# pod infrastructure container  
KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"  
  
# Add your own!  
KUBELET_ARGS=""  
```

- 启动
```
systemctl enable kubelet  
systemctl start kubelet  
systemctl enable kube-proxy  
systemctl start kube-proxy  
```

- 创建覆盖网络flannel
``` 
yum install flannel -y  
``` 

- 在master、node上均编辑 /etc/sysconfig/flanneld 文件
```
# Flanneld configuration options    
  
# etcd url location.  Point this to the server where etcd runs  
FLANNEL_ETCD_ENDPOINTS="http://etcd:2379"  
  
# etcd config key.  This is the configuration key that flannel queries  
# For address range assignment  
FLANNEL_ETCD_PREFIX="/atomic.io/network"  
  
# Any additional options that you want to pass  
#FLANNEL_OPTIONS=""  
```

- 配置etcd中关于flannel的key--
```
etcdctl mk /atomic.io/network/config '{ "Network": "10.0.0.0/16" }'  
```

- 重新启动
启动修改后的 flannel
```
systemctl enable flanneld  
systemctl start flanneld  
service docker restart  
systemctl restart kube-apiserver  
systemctl restart kube-controller-manager  
systemctl restart kube-scheduler  

systemctl enable flanneld  
systemctl start flanneld  
service docker restart  
systemctl restart kubelet  
systemctl restart kube-proxy 
``` 
