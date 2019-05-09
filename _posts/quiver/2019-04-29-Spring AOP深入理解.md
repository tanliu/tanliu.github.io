---
layout: post
title: Spring AOP深入理解
date: 2019-04-29
tags:
   - ￼spring boot
---

# 基本概念：
- 切面(Aspect):官方的抽象定义为“一个关注点的模块化，这个关注点可能会横切多个对象”。“切 面”在 ApplicationContext 中`<aop:aspect>`来配置。
- 连接点(Joinpoint) : 程序执行过程中的某一行为

- 通知(Advice): “切面”对于某个“连接点”所产生的动作。其中，一个“切面”可以包含多个“Advice”。切入点(Pointcut):匹配连接点的断言，在AOP中通知和一个切入点表达式关联。切面中的所有通知所关注的连接点，都由切入点表达式来决定。

- 目标对象(Target Object) : 被一个或者多个切面所通知的对象。例如，AServcieImpl 和 BServiceImpl，当然在实际运行时，Spring AOP 采用代理实现，实际 AOP 操作的是 TargetObject 的代理对象。

- AOP 代理(AOP Proxy) : 在 Spring AOP 中有两种代理方式，JDK 动态代理和 CGLIB 代理。默认情 况下，TargetObject 实现了接口时，则采用 JDK 动态代理，例如，AServiceImpl;反之，采用 CGLIB 代理，例如，BServiceImpl。强制使用 CGLIB 代理需要将` <aop:config>`的 proxy-target-class 属性设为 true。


# spring Aop 实现的的方式
spring aop采用的是JDK代理和CGLIB实现切面的。


# JDK代理及CGLIB代理选择的流程
默认的策略是如果目标类是接口，则使 用 JDK 动态代理技术，否则使用 Cglib 来生成代理.具体实现在DefaultAopProxyFactory下


```
	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			//如果目标类是接口的话，就使用jdk
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}

```

# AOP在什么情况下不生效
- 私有方法（接口没有方法，所以不会有jdk。cglib是继承的方式，私有方法不可以被字类重写）
- 同一类下调用（jdk代理时，是对被操作方法进行代码。代理时传的是this,只会调用原来类的方法）


# org.aspectj.lang.JoinPoint
 - getArgs() (返回方法参数)
 - getThis() (返回代理对象)
 - getTarget() (返回目标)
 - getSignature() (返回正在被通知的 方法相关信息)
 

