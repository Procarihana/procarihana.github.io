---
title: "Servlet"
date: 2020-09-17T20:02:18+08:00
draft: true
tags: ["Servlet",""]
---

Servlet的生命周期分为5个阶段：加载、创建、初始化、处理客户请求、卸载。
(1)加载：容器通过类加载器使用servlet类对应的文件加载servlet
(2)创建：通过调用servlet构造函数创建一个servlet对象
(3)初始化：调用init方法初始化
(4)处理客户请求：每当有一个客户请求，容器会创建一个线程来处理客户请求
(5)卸载：调用destroy方法让servlet自己释放其占用的资源