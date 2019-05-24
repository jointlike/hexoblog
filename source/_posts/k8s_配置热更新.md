---
title: Kubernetes ConfigMap热更新
---
 
[Kubernetes ConfigMap热更新](https://www.kubernetes.org.cn/3138.html)
-----
## 知识点
ConfigMap是用来存储配置文件的kubernetes资源对象，所有的配置内容都存储在etcd中


## 创建

1. 文件：
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
  
```

2. 命令：
```
kubectl create configmap ...
```

## 使用

1. evn挂载：
```
        envFrom:
        - configMapRef:
            name: env-config
```
2. volume挂载：

```
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
      volumes:
        - name: config-volume
          configMap:
            name: special-config
```

## 其他

更新 ConfigMap 后：

使用该 ConfigMap 挂载的 Env 不会同步更新
使用该 ConfigMap 挂载的 Volume 中的数据需要一段时间（实测大概10秒）才能同步更新
ENV 是在容器启动的时候注入的，启动之后 kubernetes 就不会再改变环境变量的值，且同一个 namespace 中的 pod 的环境变量是不断累加的，参考 [Kubernetes中的服务发现与docker容器间的环境变量传递源码探究](https://jimmysong.io/posts/exploring-kubernetes-env-with-docker/)。为了更新容器中使用 ConfigMap 挂载的配置，可以通过滚动更新 pod 的方式来强制重新挂载 ConfigMap，也可以在更新了 ConfigMap 后，先将副本数设置为 0，然后再扩容。