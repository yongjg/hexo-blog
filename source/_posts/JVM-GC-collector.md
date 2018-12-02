---
title: JVM GC collector
date: 2018-12-02 10:48:35
tags: [JVM,GC]
toc: true
thumbnail: /gallery/keyboard-black-notebook-input.jpeg
---

### 根对象集合
1. java栈中的对象引用
1. 本地方法栈中的对象引用
1. 运行时常量池中的对象引用
1. 方法区中类静态属性的对象引用
1. 与一个类对应的唯一数据类型的Class对象
<!-- more-->
---
### 垃圾回收器
![image](https://raw.githubusercontent.com/yongjg/hellogit/master/jvm/jvm-gc-gc-collector.jpg)
