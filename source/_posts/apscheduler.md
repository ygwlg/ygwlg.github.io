---
title: apscheduler
date: 2025-04-23 20:22:53
tags: 
    - Python
---

## apscheduler组件

1. task: 可调用的对象，任务有三种角色：job开始时作为运行目标；具有唯一id，用于限制多个scheduler的任务调度数量；提供带有具体参数的模板，以用于异常情况分析

2. tigger: 包括用于计算何时运行 task 的逻辑和状态

3. schedule: 由一个task和一个trigger组成，加上一些配置参数

4. job: 是对于task 运行的请求，可以在schedule运行过程中创建，也可以在用户请求运行时直接创建

5. data store: 用于存储schedule和job，并且跟踪task的运行过程

6. job executor: 用于执行job，通过调用job对应的task的函数，一个executor可能会通过直接调用，或者是子进程、子线程、外部服务的方式

7. event broker: 存储、传输事件，这些事件用于scheduler之间的通信

8. scheduler: 库的主要接口，包括的组件有：

	1. 一个 data store 

	2. 一个 event broker

	3. 一个或多个 job executor.

scheduler提供函数，使用者可以调用这些函数处理task,schedules和jobs. 

scheduler还会在后台进行如下操作：

1. 处理到期schedule

2. 生产job

3. 更新下次运行时间

此外还有如下功能：

1. 维护task，创建何时的job executor并运行，将运行的结果放回到data store
