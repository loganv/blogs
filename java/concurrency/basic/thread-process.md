# 线程与进程基础

## 什么是进程

* 是指计算机中已经运行的程序。
* 曾是分时系统的基本运作单位。
* 面向进程设计的系统中，进程是程序的基本执行实体。
* 面向线程设计的系统中，进程不是基本运行的单位，而是线程的容器。
* 程序本身只包含指令、数据及其组织结构的描述，进程才是程序的真正运行实例。（这点和Docker image 和 Docker container的关系很像）

进程有五种状态：

* 新生： 进程产生中（主动）
* 运行： 正在运行中（被动）
* 等待： 等待某事发生，比如等待用户输入完成。也称为阻塞。（主动）
* 就绪： 进入CPU的等待队列，等待获取CPU（主动）
* 结束： 完成运行（主动/被动）

进程之间的各个状态是不能随便切换的。
比如无法从**等待**直接转为**运行**，因为**运行**是指CPU目前正在执行，**等待**结束之后，只能进入CPU的等待队列，等待CPU的调度，然后等CPU去激活

## 什么是线程

* 线程是操作系统能够进行运算的最小单位。
* 大部分情况下他被包含在进程之中，是进程的实际运作单位。
* 一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程执行不同的任务。
* 在unix 系统中，也被成为轻量级进程，但轻量级进程更多的是指**内核线程**，而把用户线程成为线程。

线程有四种基本状态：

* 产生
* 阻塞
* 非阻塞
* 结束

## 线程和进程的区别与联系

* 一个进程可以有很多线程，每条线程并行执行不同的任务。
* 同一个进程中的多条线程将共享该进程中的全部系统资源，比如：虚拟地址空间，文件描述符，信号处理等。
* 同一个进程中的多个线程有各自的调用栈，自己的寄存器环境，自己的线程本地存储。