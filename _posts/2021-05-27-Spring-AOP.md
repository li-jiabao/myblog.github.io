---
title: AOP面向切面编程
author: lijiabao
date: 2021-05-27 23:30
categories: "Spring学习"
tags: "Spring学习"
---



介绍spring中的面向切向变成AOP



# Spring AOP

AOP：面向切面编程，通过预编译的方式和运行其动态代理实现程序功能的同意维护的一种技术。利用AOP可以对业务逻辑的各个部分进行隔离，从而对不同业务逻辑进行隔离，从而降低业务逻辑各个部分之间的耦合度，提高程序的可重用性，同时提高开发的效率。AOP的通知可以在不改动原有代码逻辑的基础上增加新的功能。

Spring的AOP就是将公共的业务（日志，事务管理，安全，性能统计等功能）和实际的业务逻辑隔离开来，但是又通过

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

## AOP代理

spring的AOP默认使用标准JDK动态代理作为AOP代理，这使得任何接口（或者一系列接口）可以被代理

Spring的AOP同样可以使用CGLIB代理，如果想要代理类而不是接口这是很有必要的。如果商业类对象并没有实现接口，默认会使用CGLIB。

基于接口进行变成比基于类编程更好，因此商业代码中类通常会实现一个或者多个接口。当你需要通知没有声明在接口中的方法或者你需要将代理类作为明确的类型传递给一个方法时，强制使用CGLIB是可达成的

```xml
<aop:config proxy-target-class="true"></aop:config>

<!-- 当使用AspectJ自动代理支持的时候，需要按照下面的配置来开启CGLIB支持-->
<aop:aspectj-autoproxy proxy-target-class="true" />
```

<font color='red'>spring的AOP核心是基于代理实现的</font>：

```java
// 实现了Pojo接口类的方法
public class SimplePojo {
    public void foo() {
        this.bar();  // 这个方法调用是一个直接基于this引用的方法调用
    }
    public void bar() {
        // ....
    }
}

// 使用代理的方法访问调用
public class Main {
    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        
        Pojo pojo = (Pojo) factory.getProxy();
        pojo.funcName();  // 这是一个在代理中调用的方法
    }
}
```

理解上述代码的关键是：这个Main类中有一个引用指向代理，这意味着基于这个对象引用的方法调用都是在代理上调用，因此，这个代理可以委托和特定方法调用的所有拦截器（通知）。然而一旦这个方法调用最终到达目标对象也就是被代理的对象（上述代码中就是SimplePojo），任何方法调用都会基于this引用来进行调用而不是经过代理，这会带来一个问题，<font color='red'>就是自调用会导致和方法调用相关的通知不会运行</font>

对于这种情况，最好的办法就是重构代码使得自调用不会发生

```java
// 1. 第一种方法
// 使用AopContext。currentProxy()获取当前代理对象，然后再用这个代理对象来调用通知方法
public class SimplePojo {
    public void foo() {
        (Pojo) AopContext.currentProxy().bar();  // 这样代码就会继续走代理调用
    }
    public void bar() {
        // ...
    }
}

// 上述代码还需要在创建代理的时候进行额外的配置
// 可以使用@EnableAspectJAutoProxy(exposeProxy = true)来配置，或者按照代码中的配置
public class Main {
    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);  // 需要额外进行这项配置
        
        Pojo pojo = (Pojo) factory.getProxy();
        pojo.foo();
    }
}

// 2. 第二种方法，不适用基于代理实现的AOP，使用AspectJ的AOP框架
// 使用@EnableAsync(mode = AdviceMode.ASPECTJ)开启
```

## AspectJ框架的支持

@AspectJ是一种将aspect切面定义为带有注解的注解Java类的风格

### 开启@AspectJ支持

为了在spring中使用@AspectJ风格的切面，需要开启这项支持

自动代理功能：如果spring确定一个bean是由一个或者多个切面所通知，它将会自动为这个bean生成一个代理来拦截方法调用并且确保通知按照需求运行

可以通过xml配置或者Java注解配置的方法来激活

```java
// Java注解配置
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {
    //...
}
```

```xml
<aop:aspectj-autoproxy />   <!-- xml配置文件配置，在使用schema支持的前提下 -->
```

