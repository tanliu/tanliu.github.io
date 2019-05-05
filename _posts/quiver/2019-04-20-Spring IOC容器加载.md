---
layout: post
title: Spring IOC容器加载
date: 2019-04-20
---
# BeanFactory
 ![IMAGE](http://cn-isoda-oss.yy.com/admin/video/5B4F0A12C04014AE7CF4FD60EC4E6E52.jpg)
 BeanFactory 有三 个子类:ListableBeanFactory、HierarchicalBeanFactory 和 AutowireCapableBeanFactory。 但是从上图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory
- ListableBeanFactory 接口表示这些 Bean 是可列表的，
- HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个 Bean 有可能有父 Bean。
- AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则


```
public interface BeanFactory {
    //对 FactoryBean 的转义定义，因为如果使用 bean 的名字检索 FactoryBean 得到的对象是工厂生成的对象， //如果需要得到工厂本身，需要转义
    String FACTORY_BEAN_PREFIX = "&";
    //根据 bean 的名字，获取在 IOC 容器中得到 bean 实例 Object getBean(String name) throws BeansException;
    //根据 bean 的名字和 Class 类型来得到 bean 实例，增加了类型安全验证机制。
    <T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;
    Object getBean(String name, Object... args) throws BeansException;
    <T> T getBean(Class<T> requiredType) throws BeansException;
    <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
    //提供对 bean 的检索，看看是否在 IOC 容器有这个名字的 bean boolean containsBean(String name);
    //根据 bean 名字得到 bean 实例，并同时判断这个 bean 是不是单例
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
    boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException; boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
    //得到 bean 实例的 Class 类型
    @Nullable
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;
    //得到 bean 的别名，如果根据别名检索，那么其原名也会被检索出来 String[] getAliases(String name);
}

```



# ApplicationContext
ApplicationContext 是 Spring 提供的一个高级的 IOC 容器，它除了能够提供 IOC 容器的基本功 能外，还为用户提供了以下的附加服务。
从 ApplicationContext 接口的实现，我们看出其特点:
1. 支持信息源，可以实现国际化。(实现 MessageSource 接口)
2. 访问资源。(实现 ResourcePatternResolver 接口，后面章节会讲到)
3. 支持应用事件。(实现 ApplicationEventPublisher 接口)


# BeanDefinition
> SpringIOC 容器管理了我们定义的各种 Bean 对象及其相互的关系，Bean 对象在 Spring 实现中是
以 BeanDefinition 来描述的

Bean 的解析过程非常复杂，功能被分的很细，因为这里需要被扩展的地方很多，必须保证有足够的灵 活性，以应对可能的变化。Bean 的解析主要就是对 Spring 配置文件的解析。这个解析过程主要通过下 图中的类完成:
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/69411B366C4C178FC89982B4E0EFD322.jpg)

 
# IOC加载的过程
 IOC 容器的初始化包括 BeanDefinition 的 Resource 定位、载入和注册这三个基本的过程

## 定位
先从FileSystemXmlApplicationContext入手。

```
public FileSystemXmlApplicationContext(
		String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
		throws BeansException {
  //设置applicationContext
	super(parent);
	//再 调 用 父 类 AbstractRefreshableConfigApplicationContext 的 setConfigLocations(configLocations)方法设置 Bean 定义资源文件的定位路径,解析Bean定义资源文件的路径，处理多个资源文件字符串数组
	setConfigLocations(configLocations);
	//refresh容器
	if (refresh) {
		refresh();
	}
}

```进入一步查看super方法，AbstractApplicationContext 构造方法中调用 PathMatchingResourcePatternResolver 的构造 方法创建 Spring 资源加载器。
在设置容器的资源加载器之后，接下来 FileSystemXmlApplicationContext 执行 setConfigLocations 方法通过调用其父类 AbstractRefreshableConfigApplicationContext 的 方法进行对 Bean 定义资源文件的定位。


```
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {


	/**
	 * Create a new AbstractApplicationContext with no parent.
	 */
	public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
	}

	/**
	 * Create a new AbstractApplicationContext with the given parent context.
	 * @param parent the parent context
	 */
	public AbstractApplicationContext(@Nullable ApplicationContext parent) {
		this();
		setParent(parent);
	}
	/**
	 * Return the ResourcePatternResolver to use for resolving location patterns
	 * into Resource instances. Default is a
	 * {@link org.springframework.core.io.support.PathMatchingResourcePatternResolver},
	 * supporting Ant-style location patterns.
	 * <p>Can be overridden in subclasses, for extended resolution strategies,
	 * for example in a web environment.
	 * <p><b>Do not call this when needing to resolve a location pattern.</b>
	 * Call the context's {@code getResources} method instead, which
	 * will delegate to the ResourcePatternResolver.
	 * @return the ResourcePatternResolver for this context
	 * @see #getResources
	 * @see org.springframework.core.io.support.PathMatchingResourcePatternResolver
	 */
	//获取一个Spring Source的加载器用于读入Spring Bean定义资源文件
	protected ResourcePatternResolver getResourcePatternResolver() {
		//AbstractApplicationContext继承DefaultResourceLoader，因此也是一个资源加载器
		//Spring资源加载器，其getResource(String location)方法用于载入资源
		return new PathMatchingResourcePatternResolver(this);
	}
}


```
## 加载
AbstractApplicationContext 的 refresh 函数载入 Bean 定义过程:
```
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			//调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			//告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从
			//子类的refreshBeanFactory()方法启动
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			//为BeanFactory配置容器特性，例如类加载器、事件处理器等
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				//为容器的某些子类指定特殊的BeanPost事件处理器
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				//调用所有注册的BeanFactoryPostProcessor的Bean
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				//为BeanFactory注册BeanPost事件处理器.
				//BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				//初始化信息源，和国际化相关.
				initMessageSource();

				// Initialize event multicaster for this context.
				//初始化容器事件传播器.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				//调用子类的某些特殊Bean初始化方法
				onRefresh();

				// Check for listener beans and register them.
				//为事件传播器注册事件监听器.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				//初始化所有剩余的单例Bean
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				//初始化容器的生命周期事件处理器，并发布容器的生命周期事件
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				//销毁已创建的Bean
				destroyBeans();

				// Reset 'active' flag.
				//取消refresh操作，重置容器的同步标识.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}


```refresh()方法的作用是:在创建 IOC 容器前，如果已经有容器存在，则需要把已有的容器销毁和关 闭，以保证在 refresh 之后使用的是新建立起来的 IOC 容器。refresh 的作用类似于对 IOC 容器的重 启，在新建立好的容器中对容器进行初始化，对 Bean 定义资源进行载入
# 方法调用流程
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/EC5DF8066E100689A8A13AE7FBC26FBE.jpg)
 
 
# BeanFactory 与 FactoryBean的区别
在 Spring 中，有两个很容易混淆的类:BeanFactory 和 FactoryBean。
- BeanFactory:Bean 工厂，是一个工厂(Factory)，我们 Spring IOC 容器的最顶层接口就是这个 BeanFactory，它的作用是管理 Bean，即实例化、定位、配置应用程序中的对象及建立这些对象间的 依赖。
FactoryBean:工厂 Bean，是一个 Bean，作用是产生其他 bean 实例。通常情况下，这种 bean 没有什 么特别的要求，仅需要提供一个工厂方法，该方法用来返回其他 bean 实例。通常情况下，bean 无须自 己实现工厂模式，Spring 容器担任工厂角色;但少数情况下，容器中的 bean 本身就是工厂，其作用是 产生其它 bean 实例。
当用户使用容器本身时，可以使用转义字符”&”来得到 FactoryBean 本身，以区别通过 FactoryBean 产生的实例对象和 FactoryBean 对象本身。在 BeanFactory 中通过如下代码定义了该转义字符: String FACTORY_BEAN_PREFIX = "&";
如果 myJndiObject 是一个 FactoryBean，则使用&myJndiObject 得到的是 myJndiObject 对象，而 不是 myJndiObject 产生出来的对象。


