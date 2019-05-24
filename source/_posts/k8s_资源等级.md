
---
title: Kubernetes之QoS
date: 2018-05-24 14:50:25
updated: 2018-05-24 14:50:25
categories: Devops&Kubernetes
tags: ['Devops','Kubernetes','k8s','运维开发']
---


[Kubernetes之服务质量保证（QoS）](http://dockone.io/article/2592)

## 知识点

QoS优先级
3种QoS优先级从有低到高（从左向右）：
Best-Effort pods -> Burstable pods -> Guaranteed pods

Best-Effort：如果对于全部的resources来说requests与limits均未设置，该pod的QoS即为Best-Effort

Burstable: pod中只要有一个容器的requests和limits的设置不相同，该pod的QoS即为Burstable。

Guaranteed：pod中所有容器都必须统一设置limits，并且设置参数都一致，如果有一个容器要设置requests，那么所有容器都要设置，并设置参数同limits一致，那么这个pod的QoS就是Guaranteed级别。
注：如果一个容器只指明limit而未设定request，则request的值等于limit值。

OOM分数值根据OOM_ADJ参数计算得出，对于Guaranteed级别的pod，OOM_ADJ参数设置成了-998，对于BestEffort级别的pod，OOM_ADJ参数设置成了1000，对于Burstable级别的POD，OOM_ADJ参数取值从2到999。对于kuberntes保留资源，比如kubelet，docker，OOM_ADJ参数设置成了-999，表示不会被OOM kill掉。OOM_ADJ参数设置的越大，通过OOM_ADJ参数计算出来OOM分数越高，表明该pod优先级就越低，当出现资源竞争时会越早被kill掉，对于OOM_ADJ参数是-999的表示kubernetes永远不会因为OOM而被kill掉。
