---
title: "Java Thread Pool Params"
date: 2022-06-20T22:47:13+08:00
draft: false
---


## 基本策略

1. 关注点分离，任务提交和执行分离。
2. 延迟策略，延迟初始化。

## 图解

![java-线程池参数](https://raw.githubusercontent.com/stardustman/pictures/main/img/java-thread-pool-params.drawio.svg)

## 线程数量和提交任务的关系

1. corePoolSize = 4
2. maxPoolSize = 8
3. queueCapacity = 16
4. totalTasks = 30
5. 提交的任务执行时间较长，也就是提交后相当于占用这个线程。


![threads-tasks-count-visual](https://raw.githubusercontent.com/cloudedseal/pictures/main/img/thread-task-number.png)

## References

1. [你管这破玩意叫线程池](https://mp.weixin.qq.com/s/OKTW_mZnNJcRBrIFHONR3g)

