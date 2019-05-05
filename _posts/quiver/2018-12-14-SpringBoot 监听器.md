---
layout: post
title: SpringBoot 监听器
date: 2018-12-14
tags:
   - listener
   - spring boot
---
# Spring 监听器的实现
> spring的监听器在spring boot中起到关键性作用，具体作用体现在哪，后面的补充说明

## 使用案例

```
public class SpringEventListenerDemo {

    public static void main(String[] args) {
        GenericApplicationContext context = new GenericApplicationContext();
        
        context.addApplicationListener(new ApplicationListener<ApplicationEvent>() {
            public void onApplicationEvent(ApplicationEvent event) {
                System.err.println("监听到的事件是："+event);
            }
        });
        context.addApplicationListener(new CloseListener());

        context.refresh();


        context.publishEvent("hello TanLiu");
        System.out.println("结束事件");
        context.close();
    }


    //监听的类型是通过ContextClosedEvent指定的
    private static class CloseListener implements ApplicationListener<ContextClosedEvent> {

        public void onApplicationEvent(ContextClosedEvent event) {
            System.out.println("关闭上下文："+event);
        }
    }
}

```
以上代码当context.publishEvent("hello L.Tan")的时候，配置的全局监听会收到事件。同时context也监听了关闭事件， 当context.close（）的时候会发出close!


#spring生产者注册都的机制
![Spring的事件机制.jpg](quiver-image-url/5C7A48B75CAEE10303C48E2B3AD40746.png =612x460)
## 1.发布事件的源码
```
	protected void publishEvent(Object event, @Nullable ResolvableType eventType) {
		Assert.notNull(event, "Event must not be null");
		if (logger.isTraceEnabled()) {
			logger.trace("Publishing event in " + getDisplayName() + ": " + event);
		}

		// Decorate event as an ApplicationEvent if necessary
		//当传入的类型不是ApplicationEvent子类的时候。会自己包装成PayloadApplicationEvent
		ApplicationEvent applicationEvent;
		if (event instanceof ApplicationEvent) {
			applicationEvent = (ApplicationEvent) event;
		}
		else {
			applicationEvent = new PayloadApplicationEvent<>(this, event);
			if (eventType == null) {
				eventType = ((PayloadApplicationEvent) applicationEvent).getResolvableType();
			}
		}

		// Multicast right now if possible - or lazily once the multicaster is initialized
		if (this.earlyApplicationEvents != null) {
			this.earlyApplicationEvents.add(applicationEvent);
		}
		else {
		  //这个是获取一个ApplicationEventMulticaster进行发布事件
			getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);
		}

		// Publish event via parent context as well...
		if (this.parent != null) {
			if (this.parent instanceof AbstractApplicationContext) {
				((AbstractApplicationContext) this.parent).publishEvent(event, eventType);
			}
			else {
				this.parent.publishEvent(event);
			}
		}
	}


```
## 广播事件的源码

```
	@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		for (final ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			Executor executor = getTaskExecutor();
			//可以选择性采用同步还是异步的方式
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
	}


```
#spring boot启动过程相关事件

&emsp;由于需要监听AplicationContext的生命周期，所以不可以通过提@Bean的方式创建的。spring提供了两种创建方式。第一个是SpringApplicationBuilder.listeners(…​)的方式。另外一种是配置在META-INF/spring.factories中。期中。springboot的就是配置在**META-INF/spring.factories**下的。以下是spring boot的配置。



- `ApplicationStartingEvent:` 调用run（）后，准备开始时发出的事件
- `ApplicationEnvironmentPreparedEvent`:使用Environment，但还没有创建context时发出
- `ApplicationPreparedEvent`:类已经加载了，但是context还同有refresh()
- `ApplicationStartedEvent`:refresh刚刚调用完,但是还同有调用callRunners（）
- `ApplicationReadyEvent`：调用callRunners后运行。
- `ApplicationFailedEvent`:启动过程中只要有异常都会发出这个事件


# spring boot中基于以上6个事件实现了哪些监听器
> spring boot实现的了xxxx， [官方介绍:23.5 Application Events and Listeners](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#boot-features-application-events-and-listeners)


```
# Application Listeners
org.springframework.context.ApplicationListener=\ 
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```


# Spring Cloud监听的事件

 - BootstrapApplicationListener
 - LoggingSystemShutdownListener
 
 

<p>1.tanliu<br>
r2.rjddd</p>

