---
title: kubernetes使用   
description: kubernetes基本使用
categories:
- kubernetes
tags:
- k8s   
---


`Kuberneters` 是一个开放的开发平台。

    不局限于任何一种语言
    
    没有限定任何编程接口

所以不论是用` Java`、` Go`、 `C++`还是用 `Python` 编写的服务，
都可以毫无困难地映射为 `Kuberneters 的 Service`， 
并通过标准的 `TCP 通信协议`进行交互。

由于 Kubernetes 平台对现有的编程语言、编程框架、中间件没有任何侵入性，因此现有的系统也很容易改造升级井迁移到Kuberneters平台上



`Kuberneters` 是一个完备的`分布式系统支撑平台`。
 
 Kuberneters 具有`完备的集群管理能力`
 
    包括多层次的安全防护和准入机制、
    多租户应用支撑能力、
    透明的服务注册和服务发现机制、
    内建智能负载均衡器、强大的故障发现和自我修复能力、
    服务滚动升级和在线扩容能力、
    可扩展的资源自动调度机制，
    多粒度的资源配额管理能力。
    
Kuberneters 提供了`完善的管理工具`

    包括开发、
    部署测试、
    运维监控在内的各个环节
    
Kuberneters是一个全新的`基于容器技术`的`分布式架构解决方案`

并且是一个`一站式的完备的分布式系统开发和支撑平台`。


#### Kuberneters 的一些基本知识

`Service (服务)`是分布式集群架构的核心

    拥有一个唯一指定的名字 (比如mysql-server)。
    
    拥有一个虚拟IP (ClusterIP、ServiceIP或VIP)和端口号。
    
    能够提供某种远程服务能力。
    
    被映射到了提供这种服务能力的 一组容器应用上
    

Service的服务进程目前都基于`Socket通信方式`对外提供服务 
比如 `Redis、 Memcache、 MySQL、 Web Server`，或者是实现了某个具体业务的一个特定的`TCP Server进程 `


容器提供了强大的隔离功能，所以有必要把为 Service 提供服务的`这组进程`放入容器中进行`隔离`
Kuberneters 设计了`Pod对象`，将每个服务进程包装到相应的Pod中，使其成为Pod中运行的一个`容器(Container)`

为了建立Service和Pod间的关联关系， Kuberneters 给每个Pod贴上一个`标签(Label)`
    
    给运行MySQL的Pod贴上name=mysql标签
    给运行PHP的Pod贴上name=php标签
    
    然后给相应的 Service 定义标签选择器( Label Selector)，比如MySQL Service的标签选择器的选择条件为name=mysql 
 
    意为该Service要作用于所有包含name=mysqlLabel的Pod上
    
    这样一来，就巧妙地解决了Service与Pod的关联问题。

`Pod`运行在一个我们称之为`节点(Node)`的环境中，这个节点既可以是物理机，也可以是私有云或者公有云中的一个虚拟机，通常在`一个节点上运行几百个Pod`
其次，每个 Pod 里运行着一个特殊的被称之为`Pause的容器`, 其他容器则为`业务容器`，这些业务容器共享Pause容器的网络栈和Volume挂载卷，
因此它们之间的通信和数据交换更为高效，在设计时我们可以充分利用这一特性将一组密切相关的服务进程放入同一个Pod 中