## 一、概述

Posix IPC主要有三种类型：

- Posix 消息队列 <mqueue.h>
- Posix 信号量 <semaphore.h>
- Posix 共享内存 <sys/mman.h>

## 二、Posix 消息队列

### 2.1 概述

消息队列可以认为是一个消息链表，具有写权限的线程可以往队列中放置消息，具有读权限的线程可以从队列中取走消息。每个消息都是一个记录，由发送者赋予一个优先级，优先级高的消息会优先被读。

Posix消息队列和System V消息队列的区别：**Posix返回优先级最高的消息，System V返回指定优先级的消息。**

在消息队列中的消息具有以下属性：

- 一个数字表示优先级
- 消息的数据部分的长度
- 数据本身

![image-20220903145727814](https://tbwan.oss-cn-chengdu.aliyuncs.com/imgs/UNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B2/202209031457871.png)

