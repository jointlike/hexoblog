---
title: Kubernetes之健康检查与服务依赖处理
---

# 健康检查
## 使用Liveness及Readness探针
1.Liveness探针：主要用于判断Container是否处于运行状态，比如当服务crash或者死锁等情况发生时，kubelet会kill掉Container, 然后根据其设置的restart policy进行相应操作（可能会在本机重新启动Container，或者因为设置Kubernetes QoS，本机没有资源情况下会被分发的其他机器上重新启动）。
2.Readness探针：主要用于进一步判断服务是否已经正常工作，如果服务没有加载完成或工作异常，服务所在的Pod的IP地址会从服务的endpoints中被移除，也就是说，当服务没有ready时，会将其从服务的load balancer中移除，不会再接受或响应任何请求。

liveness可用不一定readness可用，容器存活期间服务可能卡死状态。因此服务检查周期要短于健康检查周期。

-----

## 探针处理Handler类型
无论对于Readness或Liveness探针，Handler均支持以下3种类型：ExecAction, TCPSocketAction, HTTPGetAction。每种类型说明与举例如下：

1. ExecAction：Container内部执行某个具体的命令，成功返回：shell命令返回0。
例子：
```
   livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

2. TCPSocketAction：通过container的IP、port执行tcp进行检查，确认port是否打开，类似telnet
例子。
```
   readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```
3. HTTPGetAction: 通过container的IP、port、path，用HTTP Get请求进行检查，程序自定义返回结果及程序可用标准。
例子。
```
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

-----

## 服务可用性与自动恢复
1. 如果服务的健康检查（readiness）失败，故障的服务实例从service endpoint中下线，外部请求将不会再转发到该服务上，一定程度上保证正在提供的服务的正确性，如果服务自我恢复了（比如网络问题），会自动重新加入service endpoint对外提供服务。
2. 如果设置了Container（liveness）的探针，对故障服务的Container（liveness）的探针同样会失败，container会被kill掉，并根据原设置的container重启策略，系统倾向于在其原所在的机器上重启该container、或其他机器重新创建一个pod。
3. 由于上面的机制，整个服务实现了自身可用与自动恢复。

-----

##使用建议：
1. 建议对全部服务同时设置服务（readiness）和Container（liveness）的健康检查
2. 通过TCP对端口检查形式（TCPSocketAction），仅适用于端口已关闭或进程停止情况。因为即使服务异常，只要端口是打开状态，健康检查仍然是通过的。
3. 基于第二点，一般建议用ExecAction自定义健康检查逻辑，或采用HTTP Get请求进行检查（HTTPGetAction）。
4. 无论采用哪种类型的探针，一般建议设置检查服务（readiness）的时间短于检查Container（liveness）的时间，也可以将检查服务（readiness）的探针与Container（liveness）的探针设置为一致。目的是故障服务先下线，如果过一段时间还无法自动恢复，那么根据重启策略，重启该container、或其他机器重新创建一个pod恢复故障服务。

-----

# 服务依赖

## 理解Init Container
一个pod中可以有一或多个Init Container。Pod的中多个Init Container启动顺序为yaml文件中的描述顺序，且串行方式启动，下一个Init/app Container必须等待上一个Init Container完成后方可启动

由于Init Container必须要在pod状态变为Ready之前完成，所以其不需要readiness探针。另外在资源的requests与limits上与普通Container有细微差别，详见 Resources，除以上2点外，Init Container与普通Container并无明显区别

Init Containers用途
1. 前文已经提及，由于Init Container必须在app Containers启动之前完成，所以可利用其特性，用作服务依赖处理。比如某一个服务A，需依赖db或memcached，那么可以利用服务A pod的Init Container判断db/memcached是否正常提供服务，如果启动服务失败或工作异常，设置Init Container启动失败，那么pod中的服务A就不会被启动了。
2. 应用镜像因安全原因等没办法安装或运行的工具，可放到Init Container中运行。另外，Init Container独立于业务服务，与业务无关的工具如sed, awk, python, dig等也可以按需放到Init Container之中。最后，Init Container也可以被授权访问应用Container无权访问的内容。