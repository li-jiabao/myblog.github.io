---
title: Spring的bean概述
author: lijiabao
date: 2021-05-21 23:30
categories: "Spring学习"
tags: "Spring学习"
---



介绍spring中的面向切向变成AOP



# Spring AOP

## AOP的术语

- aspect（方面）：跨越多个类对象的一个关注点的模块化，在Java企业级应用中，事务管理是一个多跨越的关注的良好示例。Spring中的aspect实现可以通过使用常规的类对象或者使用使用了@Aspect注解注解的类对象
- Joint point（连接点）：程序执行期间的节点，比如方法的执行或者异常的处理。Spring AOP中连接点往往代表了方法执行
- Advice（通知）：由Aspect在特定的Ioint point采取的行动，包括了几种不同类型的通知：around，before，after通知大部分的AOP框架，将通知模拟为一个拦截器并且在Joint point处维持了一系列的拦截器
- Pointcut（切入点）：切入点就是匹配连接点的一种预测描述。通知和切入点表达式相联系，并且会在任何匹配切入点的连接点上运行。切入点的示例：有特定名字的方法的执行。连接点被看作是切入点的匹配是AOP的中心思想，并且Spring默认使用AspectJ pointcut expression language
- Introduction（引入）：声明代表了一种类型的额外的方法或者字段。Spring AOP允许你可以引入任何的新接口或者相应的接口实现到某个advised对象。比如：可以让某个bean实现IsModified接口来简化缓存操作（在AspectJ群体，引入常被看成一个类型之间的声明定义）
- Target Object（目标对象）：被一个或者多个aspect advised的对象，又被看作advised对象。Spring AOP是由运行时代理实现的，因此，这个目标对象又被叫做被代理对象
- AOP proxy（AOP代理）：为了实现aspect contract（advise method执行等）操作而由AOP框架创建的对象，Spring框架中，是JDK动态代理或者CGLIB代理
- Weaving（编织联系）：将aspect和其他应用类型或者对象联系起来创建一个advised对象，这可以在编译时完成（使用AspectJ编译器）加载时或者运行时完成。Spring框架在运行时完成这个操作。

## Spring AOP的advice类型

- before advice（前置通知）：在连接点方法执行之前运行的通知，并不能组织连接点方法的执行，除非这个通知抛出异常
- after returning advice（返回后通知）：连接点正常执行完成之后执行的通知advice（程序没有抛出异常）
- after throwing advice（抛出异常后通知）：如果方法因为抛出异常而退出后执行的advice
- after advice（运行后通知）：不管正常或者异常退出都执行的advice
- around advice（环绕通知）：环绕连接点的（比如方法调用）运行的advice，最强大的一种advice。可以选择是否继续执行到连接点或者结束advised 方法执行通过返回自己的值或者抛出异常