---
layout: post
title: Spring MVC 源码浅读
date: 2019-05-07
tags:
   - spring boot
---
# 初始化

```
	/**
	 * 重写HttpServletBean的方法。创建WebApplicationContext会在bean创建的时候
	 * Overridden method of {@link HttpServletBean}, invoked after any bean properties
	 * have been set. Creates this servlet's WebApplicationContext.
	 */
	@Override
	protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
		if (this.logger.isInfoEnabled()) {
			this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
		}
		long startTime = System.currentTimeMillis();

		try {
		  // 初始化
			this.webApplicationContext = initWebApplicationContext();
			initFrameworkServlet();
		}
		catch (ServletException ex) {
			this.logger.error("Context initialization failed", ex);
			throw ex;
		}
		catch (RuntimeException ex) {
			this.logger.error("Context initialization failed", ex);
			throw ex;
		}

		if (this.logger.isInfoEnabled()) {
			long elapsedTime = System.currentTimeMillis() - startTime;
			this.logger.info("FrameworkServlet '" + getServletName() + "': initialization completed in " +
					elapsedTime + " ms");
		}
	}


```

## mapper的注册
![IMAGE](http://cn-isoda-oss.yy.com/admin/video/FD49628A17FD653B6393370EB6DD1FDA.jpg)
`RequestMappingInfoHandlerMapping`通过实现`InitializingBean`接口实现对@Controller和@RequestMapping对应HandlerMothed的创建及及相关HandlerInterceptor的加载。


```
	@Override
	protected boolean isHandler(Class<?> beanType) {
		return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
				AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
	}


```


## 初始化内容


```
	//向下调用追踪会调用方法：
protected void initStrategies(ApplicationContext context) {
		//上传文件处理
		initMultipartResolver(context);
		//国际化的配置
		initLocaleResolver(context);
		//主题目样式的配置
		initThemeResolver(context);

		//每个环境的接口实现
		initHandlerMappings(context);
		
		initHandlerAdapters(context);
		//对异常的处理
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}



```
## 为什么HandlerMappings会是一个存储在一个list之中？
因为url做灵活了。map是无法对多种格式做匹配。如spring中有一个bestMatch机制。

# 请求处理流程
![Spring MVC的结构图.jpg](quiver-image-url/4EA27AB3F99993BB6AAB8853B9351A97.png =1373x989)
1. 遍历HandlerMapping并创建对HandlerExcetionChain
2. 调用HandlerExcetionChain中interception的pre方法
3. 获取创建handler(object类型)对应的HandlerAdapter,并调用handler方法，返回ModeAndView
   - 调用 HandlerMethodArgumentResolver
   - 调用controller方法
   - 调用 HandlerMethodReturnValueHandler
4. ViewResolver处理ModeAndView，返回view
5. 调用调用HandlerExcetionChain中interception的post方法

# 可扩展点
1. 如果自定义的controller的中url的加载。
可以实现HanderMapping,生产自定义的hander，并实现对应的HanderAdapter来处理相应的请求

2. 对请求参数的加工，如：封面的客户端通过请求头。
可以实现HandlerMethodArgumentResolver

3. 对返回的数据进行加工
可以实现

