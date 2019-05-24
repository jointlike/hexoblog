
---
title: k8s权限管理
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: Devops&Kubernetes
tags: ['Devops','Kubernetes','k8s','运维开发']
---

补充：
https://www.cnblogs.com/cf532088799/p/7977083.html

## 原理概述：  
K8s授权管理分用户认证和授权两部分，授权根据RBAC角色控制方法为不同用户配置不同角色完成权限管理  


## 管理组件  
RBAC 是让用户能够访问 Kubernetes API 资源的授权方式。  

## 认证顺序  
1. 用户认证  
2. 权限认证  


## 用户类型  
k8s里面有两种用户，一种是User，一种就是service account，User给人用的，service account给进程用的，让进程有相关的权限。
如dasboard就是一个进程，我们就可以创建一个service account给它，让它去访问k8s。

## 角色  
角色是一系列的权限的集合，例如一个角色可以包含读取 Pod 的权限和列出 Pod 的权限， ClusterRole 跟 Role 类似，但是可以在集群中到处使用（ Role 是 namespace 一级的）。

角色绑定 RoleBinding 把角色映射到用户，从而让这些用户继承角色在 namespace 中的权限。ClusterRoleBinding 让用户继承 ClusterRole 在整个集群中的权限。

## 操作流程：  

1. 创建用户，serviceaccount 或 user. user需要自设token。请看下面详细操作  

2. 创建角色，分为 ClusterRole, Role. 指定操作资源及操作类型（增删改查等）  

3. 绑定用户与角色，kind: ClusterRoleBinding or RoleBinding   

 
## 参考链接  
> https://zhuanlan.zhihu.com/p/30684413?from_voters_page=true
> https://www.kubernetes.org.cn/3238.html
> https://kubernetes.io/docs/admin/authentication/
> http://www.infoq.com/cn/articles/Kubernetes-API
> https://docs.bitnami.com/kubernetes/how-to/configure-rbac-in-your-kubernetes-cluster/
> https://blog.csdn.net/qq_34463875/article/details/78728041


## 更多知识点：  
 
  - 集群用户权限管理： 内部集群之间访问只能根据serviceaccount定义的角色权限绑定每一个deploy pod
   （如：dashboard）的权限来访问。serviceaccount若限制了权限，可以追加，减少权限只能重新绑定发布无法更改。

  - 在 pod 创建之初 service account 就必须已经存在，否则创建将被拒绝设置非默认的 service account，
    只需要在 pod 的spec.serviceAccountName 字段中将name设置为您想要用的 service account 名字即可。
    不能更新已创建的 pod 的 service account。
    
  - 可以在现有的serviceaccount 中增加imagePullSecrets 使pod构建镜像时在当前命名空间默认引用私有仓库的认证。
  
  - 内部集群之间访问只能根据serviceaccount定义的角色权限绑定每一个deploy pod的权限来访问。
    serviceaccount若限制了权限，可以追加，减少权限只能重新绑定发布无法更改。

  - kubeconfig namespace 设置可以限定角色可访问的命名空间，若角色权限过大，binding中也可以进一步限定。


## 案例：


1. service account(sc)案例   

进程用户（service account）权限管理，是专门为进程用户创建，也可以达到权限管理的目的，然而建议是同一个进程多个用户。作为示例我们这里部署一个进程权限管理作为演示。部署流程如下：

1). 创建sc   

	apiVersion: v1
	kind: ServiceAccount
	metadata:
	  labels:
		k8s-app: admin-dashboard
	  name: admin-dashboard
	  namespace: kube-system

2). 创建角色

	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRole
	metadata:
	  #无命名空间，clusterrole用于全集群
	  name: admin-dashboard-minimal
	rules:
	- apiGroups: [""]
	  resources: ["secrets"]
	  verbs: ["create"]
	- apiGroups: ["", "extensions", "apps"]
	  resources: ["deployments", "pods", "services"]
	  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]


3). 绑定角色

	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRoleBinding
	metadata:
	  name: admin-dashboard-rolebinding
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: admin-dashboard-minimal
	subjects:
	- kind: ServiceAccount
	  name: admin-dashboard
	  namespace: kube-system

4). kubernetes dashboard必须以nodeport方式部署。



2. 普通用户案例

普通用户权限管理：k8s APIserver不支持普通用户创建，只需生成token（# head -c 16 /dev/urandom | od -An -t x| tr -d ' '）,
将该用户的token追加到kuberentesAPI启动参数中指定的token（通常在/etc/kubernetes/token.csv）文件中即可。设置用户ssh认证，
根据绑定的角色权限并追加到kubeconfig文件中，供客户端访问。终端命令行可以通过不同用户用不同上下文达到访问权限管理。
这个上下文可以转换成第三方用户管理平台，Header追加。authorization Bearer token 来访问。注意：普通用户需要重启（重新部署）apiserver。


1). 创建用户token并录入token指定tom用户名。任何Linux终端执行如下（可选）：  

    ```  
	head -c 16 /dev/urandom | od -An -t x| tr -d ' '  
	```  
	生成的token以'token,user,uid,"group1,group2,group3"'格式写入到其中一个master节点`/etc/kubernetes/ssl/token.csv`文件中。uid不重复的编号即可。如：716c8d3cd0e75dc25d1eb758e4635d39, tom, 10002, dev1    
	然后重启apiserver以是当前新加的token生效，然后继续以下操作。
	说明：此步骤可选, 2）中命令可生成用户

2). SSH认证并绑定tom：  

	```  
	### 生成证书
	openssl genrsa -out employee.key 2048
	openssl req -new -key employee.key -out employee.csr -subj "/CN=tom/O=dev1"  #用户tom，分组dev1
	openssl x509 -req -in employee.csr -CA /etc/kubernetes/ssl/k8s-root-ca.pem -CAkey /etc/kubernetes/ssl/k8s-root-ca-key.pem -CAcreateserial -out employee.crt -days 5000
	
	### 将证书和tom用户绑定
	kubectl config set-credentials tom --client-certificate=employee.crt --client-key=employee.key 
	kubectl config set-context employee-context --cluster=kubernetes --namespace=shelve-mgr --user=tom
	
	## 可用kubectl config view 查看添加是否成功。cluster, context, user

3). 创建角色   

	```  
	kind: Role
	apiVersion: rbac.authorization.k8s.io/v1
	metadata:
	  namespace: shelve-mgr
	  name: employee
	rules:
	- apiGroups: [""]
	  resources: ["pods"]
	  verbs: ["get", "list", "watch", "create", "update", "patch"]
	```
	
4). 角色绑定   

	```
	kind: RoleBinding
	apiVersion: rbac.authorization.k8s.io/v1
	metadata:
	  name: employee-role-binding
	  namespace: shelve-mgr
	subjects:
	- kind: User
	  name: tom
	roleRef:
	  kind: Role
	  name: employee
	  apiGroup: ""
	```  
完成用户配置，现在用户终端受当前角色权限限制。普通用户需要重启apiserver后方可生效。

1). 配置kubectl默认终端用户  
    ```  
	# 验证
	kubectl --config=employee-context get pods
	# 设置默认会话上下文
	kubectl config use-context employee-context
	```  
2). 其他客户端及api请求以header中携带 Authorization: Bearer + token方式访问。


