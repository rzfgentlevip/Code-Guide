---
title: 面试指引说明
createTime: 2025/03/05 07:23:37
permalink: /compre-guide/interview/
icon: material-icon-theme:folder-aws-open
---

编写者：[::noto:red-heart::rand777](/friends/persons/)

:::tip 本文提供一些技术组件的重点面试内容,模拟面试;
:::

## 面试题大纲

1. Redis的特点[优缺点]，redis在项目中如何使用，为什么选择redis? 
2. redis有哪些数据类型，项目中都使用哪些数据类型，如何设计使用的?
3. redis内存淘汰策略，有哪些算法，redis是如何维护内存的?
4. redis的架构，为什么redis是单线程的，设计为单线程有什么好处?
5. redis为什么这么快，使用了哪些技术?
6. 什么是redis持久化，redis有哪几种持久化，为什么要做持久化？
7. redis同步机制了解么？
8. redis集群模式和哨兵模式对比？
9. redis为什么要有哨兵模式？
10. 缓存雪崩，缓存穿透，缓存击穿有什么区别，都有哪些解决方案？
11. 什么是COW，redis中哪里用到cow?
12. 什么是aof重写，为什么要重写？

[Redis参考资料](/compre-guide/interview/redis/)

---

1. 介绍一下kafka架构，讲一下kafka在项目中如何使用，为什么使用kafka?
2. 你们在k8s中如何部署kafka服务？
3. kafka生产者在发送数据的时候，有哪些分区策略？
4. kafka有哪些分区分配策略？
5. 什么是重平衡，介绍一下重平衡原理?
6. kafka为什么吞吐量大，使用了什么技术？
7. kafka是如何保证消息的可靠性？
8. ISR是什么，为什么要有ISR？

[kafka参考资料](/compre-guide/interview/kafka/)

---

1. 什么是k8s，k8s和docker的区别?
2. k8s的组件有哪些,作用分别是什么？
3. kubelet的功能、作用是什么？
4. kube-api-server的端口是多少？各个pod是如何访问kube-api-server的？
5. k8s中命名空间作用是什么？
6. k8s中创建pod的流程是什么？
7. pod中有哪些健康检查？常见的探测方式有哪些？
8. deployment的升级策略？
9. service将流量分发到后端pod的策略有哪些？
10. k8s中如何使用pv和pvc，生产环境pv使用什么方式创建？pv的生命周期有哪些？
11. Worker节点宕机,简述Pods驱逐流程？
12. 有状态服务和无状态服务的区别？
13. ingress-controller的工作机制？
14. kube-proxy的三种工作模式和原理？
15. pod的创建流程是怎样的？
16. pod创建过程中一般有哪些状态？
17. pod一直处于pending状态一般有哪些情况,怎么排查？
18. service有哪几种类型？
19. service、endpoint、kube-proxys三种的关系是什么？
20. 无头service和普通的service有什么区别,无头service使用场景是什么？
21. deployment的滚动更新策略有两个特别主要的参数,解释一下它们是什么意思？
22. pv的回收策略有哪几种？
23. 在pv的生命周期中,一般有几种状态？
24. Kubernetes中的Ingress是什么,它如何工作？
25. Kubernetes的Affinity和Anti-Affinity规则是什么？它们的应用场景有哪些？
26. Kubernetes中的Horizontal Pod Autoscaler (HPA) 和 Vertical Pod Autoscaler (VPA) 有何区别？
27. Kubernetes中的Taints和Tolerations是什么？它们是如何工作的？
28. Kubernetes 网络插件有哪些？
29. 请简述 Kubernetes 中的监控方案。
30. 请简述 Kubernetes 中的服务网格(Service Mesh)概念。

[k8s参考资料](/compre-guide/interview/k8s/)

---

1. docker的工作原理是什么,讲一下?
2. docker架构都包含哪些工作组件？
3. docker技术的三大核心概念是什么？
4. Dockerfile中的CMD和ENTRYPOINT指令有什么区别？
5. Docker Compose的作用及其优点？
6. Docker 如何实现容器的网络隔离？
7. Docker 中的 Volume 和 Bind Mount 的区别是什么？
8. 讲一下镜像的分层结构以及为什么要使用镜像的分层？
9. Dockerfile的基本指令有哪些？
10. 简单描述一下Dockerfile的整个构建镜像过程？
11. 讲一下容器的copy-on-write特性,修改容器里面的内容会修改镜像吗？
12. 描述Docker 容器的生命周期管理命令？
13. Docker 容器的隔离机制是如何工作的？
14. Docker 如何实现跨主机容器通信？
15. Docker 环境中实现高可用性(HA)的策略？
16. Docker 中的 Layer Caching 如何加速构建过程？
17. Docker 容器启动慢的可能原因和解决方法？

[docker参考资料](/compre-guide/interview/docker/)

---

1. 什么是RDD，为什么会产生RDD？
2. spark中算子的分类，是如何触发计算任务的？
3. reduceByKey，groupByKey，foldByKey等不同算子的区别？
4. 什么是DAG，为什么存在DAG？
5. Spark 广播变量和累加器介绍一下？
6. 介绍一下RDD的持久化和缓存？
7. RDD 容错机制 Checkpoint？
8. 什么是RDD的宽窄依赖？
9. 介绍一下spark的计算模型，和mr模型有什么差异？



[spark参考资料](/compre-guide/interview/spark/)