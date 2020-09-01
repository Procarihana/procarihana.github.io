---
title: "Spring"
date: 2020-09-01T11:37:47+08:00
draft: false
toc: false
images:
tags: [SpringMvc]
---

## Spring MVC
-  模型model(javabean),  视图view(jsp/img)   控制器Controller(Action/servlet)  
- C 存在的目的就是为了保证M和V的一致性
   当M发生改变时,C可以把M中的新内容更新到V中.
- **SpringMVC是Spring框架内置的MVC的实现，一个Spring内置的MVC框架**  
MVC框架，它解决WEB开发中常见的问题(参数接收、文件上传、表单验证、国际化、等等)，而且使用简单，与Spring无缝集成。   
支持 RESTful风格的 URL 请求 。  
采用了松散耦合可插拔组件结构，比其他 MVC 框架更具扩展性和灵活性。
- 为了解决页面代码和后台代码的分离

## Spring 容器原理
- Ioc 容器的事实标准
- Ioc (Inverse of Control控制反转) :只需要告诉容器对象的依赖关系，容器就会自动完成依赖和Beans的生成，通过依赖注入完成依赖，整个过程就是控制反转
- Java对象是Bean
- 当A对象必须使用B对象才能完成自己的工作的时候，就是A依赖B 

