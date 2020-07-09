---
layout: post
title:  "《深入分析javaweb技术内幕》之一——SpringMVC的工作机制与设计模式"
date:   2017-07-07 00:00:00
categories: java
tags: java springMVC
mathjax: true
---


# 一、springMVC

## 1.1 springMVC的总体设计

springMVC源码下载：http://download.csdn.net/detail/yangchao13341408947/8905157

要使用springMVC，需要：

1） 在web.xml文件中配置一个DispatcherServlet,如下<servlet>




    <servlet>
		<servlet-name>springmvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
		<load-on-startup>2</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

2） 定义一个springmvc-servlet.xml，定义Mapping、view、control



    <!--view-->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>
    <!--使用注解方式Mapping\control-->





	<context:component-scan base-package="com.webyun" use-default-filters="false">
		<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
	</context:component-scan>

	<mvc:annotation-driven/>


总结：springMVC的使用非常简单，我们只要扩展一个路径映射关系；定义一个试图解析器；再定义一个业务逻辑的处理流程规则，springMVC 就能够帮你完成所有的MVC功能了。

要搞清楚springMVC如何工作，主要看DispatcherServlet的代码与DispatcherServlet类相关的结构。



## 1.2 DispatcherServlet

###1.2.1 DispatcherServlet 类图

DispatcherServlet-->FrameworkServlet-->HttpServletBean-->HttpServlet（-->表示继承）



DispatcherServlet 类继承了HttpServlet,在servlet的init方法调用时DispatcherServlet执行springMVC的初始化工作。可以在其initStrategies方法中知道：



	protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}


1） initMutipartResolver  用于处理文件上传

2）initLocaleResolver  用于处理国际化

3）initThemeResolver 用于定义一个主题

4） initHandlerMappings 定义用户设置的请求映射关系

5） initHandlerAdapters 根据handler的类型定义不同的处理规则

6）initHandlerExceptionResolvers 当handler出错时，会通过这个handler统一处理

7）initRequestToViewNameTranslator 将指定的viewname按照定义的requestToViewNameTranslator替换成想要的格式，如加上前缀或者后缀

8）initViewResolvers 将view解析成页面

9）initFlashMapManager ：Initialize the {@link FlashMapManager} used by this servlet instance.



### 1.2.2 DispatcherServlet启动都做了什么

在springMVC框架中，有3个组件是用户必须要定义和扩展的：定义URL映射规则、实现业务逻辑的handler实例对象、渲染模板资源。

HttpServlet 初始化时调用了HttpServletBean 的init方法：该方法的作用是获取servlet中的init参数，并创建一个beanwrapper对象，然后由子类真正执行beanwrapper的初始化工作，代码如下：



    @Override
	public final void init() throws ServletException {
		if (logger.isDebugEnabled()) {
			logger.debug("Initializing servlet '" + getServletName() + "'");
		}

		// Set bean properties from init parameters.
		try {
			PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
			BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
			ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
			bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
			initBeanWrapper(bw);
			bw.setPropertyValues(pvs, true);
		}
		catch (BeansException ex) {
			logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
			throw ex;
		}

		// Let subclasses do whatever initialization they like.
		initServletBean();

		if (logger.isDebugEnabled()) {
			logger.debug("Servlet '" + getServletName() + "' configured successfully");
		}
	}

但是HttpServletBean和DispatcherServlet都没有覆盖其initBeanWrapper(bw)方法，所以创建的BeanWrapper对象没有任何作用，spring容器也不是通过BeanWrapper创建的。





spring容器的创建是在FrameworkServlet类中的initServletBean（）方法中完成的，这个方法会创建WebApplicationContext 对象。



	@Override
	protected final void initServletBean() throws ServletException {
		getServletContext().log("Initializing Spring FrameworkServlet '" + getServletName() + "'");
		if (this.logger.isInfoEnabled()) {
			this.logger.info("FrameworkServlet '" + getServletName() + "': initialization started");
		}
		long startTime = System.currentTimeMillis();

		try {
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
	
spring容器的加载时会调用DispatcherServlet的initStrategies（）方法来完成DispatcherServlet中定义的初始化工作。在initStrategies（）方法中会初始化springMVC框架需要的8个组件，这个8个组件对应的8个bean对象都保存在DispatcherServlet类中。

## 1.3 control设计

springMVC的control主要由HandlerMapping和HandlerAdapters两个组件提供。

HandlerMapping负责映射用户的URL和对应的处理类，HandlerAdapters并没有规定这个URL与应用的处理类如何映射，在HandlerMapping接口中只定义了根据一个URL必须返回一个由HandlerExecutionChain代表的处理链，我们可以在这个处理链中添加任意的HandlerAdapters实例来处理这个URL请求。



### 1.3.1 HandlerMapping 初始化

Spring MVC 本身提供了很多HandlerMapping的实现类，默认使用BeanNameUrlHandlerMapping。以SimpleUrlHandlerMapping为例看Spring MVC如何将请求的URL映射到我们定义的bean的。



    package org.springframework.web.servlet;
    import javax.servlet.http.HttpServletRequest;
    public interface HandlerMapping {
    	String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";
    	String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";	
    	String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";
    	String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";
    	String MATRIX_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".matrixVariables";
    	String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";
    	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
    
    }






    package org.springframework.web.servlet.handler;
    
    import java.util.HashMap;
    import java.util.Map;
    import java.util.Properties;
    
    import org.springframework.beans.BeansException;
    import org.springframework.util.CollectionUtils;
    
    public class SimpleUrlHandlerMapping extends AbstractUrlHandlerMapping {
    
    	private final Map<String, Object> urlMap = new HashMap<String, Object>();
    
    	public void setMappings(Properties mappings) {
    		CollectionUtils.mergePropertiesIntoMap(mappings, this.urlMap);
    	}
    
    	public void setUrlMap(Map<String, ?> urlMap) {
    		this.urlMap.putAll(urlMap);
    	}
    
    	public Map<String, ?> getUrlMap() {
    		return this.urlMap;
    	}
    
    	@Override
    	public void initApplicationContext() throws BeansException {
    		super.initApplicationContext();
    		registerHandlers(this.urlMap);
    	}
    
    	protected void registerHandlers(Map<String, Object> urlMap) throws BeansException {
    		if (urlMap.isEmpty()) {
    			logger.warn("Neither 'urlMap' nor 'mappings' set on SimpleUrlHandlerMapping");
    		}
    		else {
    			for (Map.Entry<String, Object> entry : urlMap.entrySet()) {
    				String url = entry.getKey();
    				Object handler = entry.getValue();
    				// Prepend with slash if not already present.
    				if (!url.startsWith("/")) {
    					url = "/" + url;
    				}
    				// Remove whitespace from handler bean name.
    				if (handler instanceof String) {
    					handler = ((String) handler).trim();
    				}
    				registerHandler(url, handler);
    			}
    		}
    	}
    
    }
    
### 1.3.2 HandlerAdapter 初始化


package org.springframework.web.servlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public interface HandlerAdapter {
	boolean supports(Object handler);
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
	long getLastModified(HttpServletRequest request, Object handler);
}




HandlerMapping可以完成URL和Handler（思考：这里的Handler是什么？）的映射关系，那么HandlerAdapter就可以帮助自定义各种Handler了。在springMVC中提供的三种典型的HandlerMapping实现类。

SimpleServletHandlerAdapter ：可以继承servlet接口

HttpRequestHandlerAdapter：可以继承HttpRequestHandler接口

SimpleControllerHandlerAdapter：可以继承controller接口

HandlerAdapter 的初始化：简单的创建一个HandlerAdapter对象，将这个对象保存在DispatcherServlet的HandlerAdapters集合中。当springMVC将某个URL对应到某个Handler时，在HandlerAdapters集合中查询那个HandlerAdapter接口对应的方法。

### 1.3.3 control的调用逻辑

整个springMVC的调用是从DispatcherServlet的doService()方法开始的，在doService方法中会将ApplicationContext、localResolver、themeResolver等对象添加到request中以便于在后面使用。接着调用doDispatch方法，这个方法是主要的处理用户请求的地方。



	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		if (logger.isDebugEnabled()) {
			String requestUri = urlPathHelper.getRequestUri(request);
			String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
			logger.debug("DispatcherServlet with name '" + getServletName() + "'" + resumed +
					" processing " + request.getMethod() + " request for [" + requestUri + "]");
		}

		// Keep a snapshot of the request attributes in case of an include,
		// to be able to restore the original attributes after the include.
		Map<String, Object> attributesSnapshot = null;
		if (WebUtils.isIncludeRequest(request)) {
			logger.debug("Taking snapshot of request attributes before include");
			attributesSnapshot = new HashMap<String, Object>();
			Enumeration<?> attrNames = request.getAttributeNames();
			while (attrNames.hasMoreElements()) {
				String attrName = (String) attrNames.nextElement();
				if (this.cleanupAfterInclude || attrName.startsWith("org.springframework.web.servlet")) {
					attributesSnapshot.put(attrName, request.getAttribute(attrName));
				}
			}
		}

		// Make framework objects available to handlers and view objects.
		request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
		request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
		request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
		request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

		FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
		if (inputFlashMap != null) {
			request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
		}
		request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
		request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

		try {
			doDispatch(request, response);
		}
		finally {
			if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				return;
			}
			// Restore the original attribute snapshot, in case of an include.
			if (attributesSnapshot != null) {
				restoreAttributesAfterInclude(request, attributesSnapshot);
			}
		}
	}


	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = processedRequest != request;

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						String requestUri = urlPathHelper.getRequestUri(request);
						logger.debug("Last-Modified value for [" + requestUri + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				try {
					// Actually invoke the handler.
					mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
				}
				finally {
					if (asyncManager.isConcurrentHandlingStarted()) {
						return;
					}
				}

				applyDefaultViewName(request, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Error err) {
			triggerAfterCompletionWithError(processedRequest, response, mappedHandler, err);
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				return;
			}
			// Clean up any resources used by a multipart request.
			if (multipartRequestParsed) {
				cleanupMultipart(processedRequest);
			}
		}
	}

control处理的关键是DispatcherServlet的handlerMappings集合中根据请求的URL匹配每个HandlerMapping对象中的某个Handler,匹配成功返回这个Handler的处理链HandlerExecutionChain对象，这个HandlerExecutionChain对象中包含了多个HandlerInterceptor对象。

HandlerInterceptor接口中定义的三个方法：preHandle和postHandle分别在Handler执行前和执行后执行，afterCompletion在view渲染完成、DispatcherServlet返回之前执行。









Q1:HandlerExecutionChain

参考：《深入分析javaweb技术内幕》——许立波，电子工业出版社
