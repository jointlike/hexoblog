---
title: Kubernetes监控
---

## Node节点
- 基础命令

	`kubectl describe node node3`

## kubelet

+ 组件  
  1. **cAdvisor**  
	描述：收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)


## Heapster

+ 描述  
> Heapster首先从K8S Master获取集群中所有Node的信息，然后通过这些Node上的kubelet获取有用数据，而kubelet本身的数据则是从cAdvisor得到。所有获取到的数据都被推到Heapster配置的后端存储中，并还支持数据的可视化。现在后端存储 + 可视化的方法，如InfluxDB + grafana。

+ 组件  
  1. **InfluxDB**  
    描述：存储监控数据

## 相关链接  

> 1. [Kubernetes监控之Heapster介绍](https://segmentfault.com/a/1190000007708162)
