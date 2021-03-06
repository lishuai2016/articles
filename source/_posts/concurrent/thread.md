---
title: thread解析
categories: 
- concurrent
tags:
---

# 1、线程优先级的介绍
[通过设置优先级，数字越大的优先级别越高(1-10)  获取cpu概率大]

java 中的线程优先级的范围是1～10，默认的优先级是5。“高优先级线程”会优先于“低优先级线程”执行。
java 中有两种线程：用户线程和守护线程。可以通过isDaemon()方法来区别它们：如果返回false，则说明该线程是“用户线程”；
否则就是“守护线程”。用户线程一般用于执行用户级任务，而守护线程也就是“后台线程”，一般用来执行后台任务。
需要注意的是：Java虚拟机在“用户线程”都结束后会后退出。

JDK 中关于线程优先级和守护线程的介绍如下：
每个线程都有一个优先级。“高优先级线程”会优先于“低优先级线程”执行。每个线程都可以被标记为一个守护进程或非守护进程。
在一些运行的主线程中创建新的子线程时，子线程的优先级被设置为等于“创建它的主线程的优先级”，
当且仅当“创建它的主线程是守护线程”时“子线程才会是守护线程”。

当Java虚拟机启动时，通常有一个单一的非守护线程（该线程通过是通过main()方法启动）。JVM会一直运行直到下面的任意一个条件发生，JVM就会终止运行：
(01) 调用了exit()方法，并且exit()有权限被正常执行。
(02) 所有的“非守护线程”都死了(即JVM中仅仅只有“守护线程”)。
每一个线程都被标记为“守护线程”或“用户线程”。当只有守护线程运行时，JVM会自动退出。