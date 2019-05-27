---
title: 构建基本的 Spring Web 应用程序示例
date: 2018-10-15
tags: Java
---

### 追踪 Spring MVC 的请求
用户在Web浏览器中点击链接或者提交表单，请求工作就开始。从离开浏览器开始到获取响应放回，会经历很多的站，图1展示了请求使用Spring MVC所经历的所有站点

![图1.png](https://upload-images.jianshu.io/upload_images/1990028-27b050dc0d3d20de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

请求的第一站是Spring的DispatcherServlet。Spring MVC所有的请求都是通过一个前端控制器Servlet，前端控制器是常用的Web应用程序模式，在这里一个单实例的Servlet将请求委托给应用程序的其它组件执行实际的处理。在Spring MVC中，DispatcherServlet就是前端控制器

接下来DispatcherServlet会查询一个或多个处理器映射，来确定请求的下一站去哪里，处理器映射会根据请求所携带的URL信息来进行决策

选择好了合适的控制器，DispatcherServlet会将请求发送给选中的控制器，由控制器去处理这些信息，在良好的设计模式下，通常控制器本身是很少处理工作，而是将业务逻辑委托给一个或者多个服务对象进行处理

处理完信息之后 将信息转换成模型数据并制定传入的视图，由视图解析器来将逻辑视图名匹配为一个特定的视图实现，响应给用户

### 搭建Spring MVC

##### 1：配置 DispatcherServlet

DispatcherServlet是Spring MVC的核心，配置方法有两种，一种是在web.xml文件中进行配置，这个文件会放到应用的WAR包里面。另一种是通过继承"AbstractAnnotationConfigDispatcherServletInitializer"的方式，通过继承的方式与配置web.xml并没有多大的区别，唯一的问题在于它只能部署在支持Servlet3.0的服务器中才能正常工作

扩展了"AbstractAnnotationConfigDispatcherServletInitializer"的任意类都会自动的部署DispatcherServlet的Spring应用上下文，Spring的应用上下文会位于应用程序的Servlet上下文之中

解析"AbstractAnnotationConfigDispatcherServletInitializer"过程中发现，在支持Servlet3.0环境中，容器会在类路径中查找实现"javax.servlet.ServletContainerInitializer"接口的类，如果发现，就会用它来配置Servlet容器，Spring提供的"SpringServletContainerInitializer"类就实现了该接口。在"SpringServletContainerInitializer"类中又回去查找实现"WebApplicationInitializer"的类，"AbstractAnnotationConfigDispatcherServletInitializer"就扩展了"WebApplicationInitializer"。因此当我们部署容器的时候，容器就会自动发现它并配置Servlet上下文

	/**

	* @Description:    配置DispatcherServlet

	* @Author:         HJ

	* @CreateDate:     2018/10/11 上午10:06

	* @UpdateUser:     HJ

	* @UpdateDate:     2018/10/11 上午10:06

	* @UpdateRemark:   修改内容

	* @Version:        1.0

	*/
	package cn.funnyhuang.config;


	import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

	public class SpringWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    	@Override
    	protected Class<?>[] getRootConfigClasses() {
        	return new Class[] {RootConfig.class};
    	}

    	@Override
    	protected Class<?>[] getServletConfigClasses() {
        	//指定配置类
        	return new Class[] {ServletConfig.class};
    	}

    	@Override
   	 	protected String[] getServletMappings() {
        	//将DispatcherServlet映射到"/"
        	return new String[] {"/"};
    	}
	}


在"getServletMappings"方法中，它会将一个或多个路径映射到DispatcherServlet上，示例中映射到的是"/"，这代表它会处理进入应用的所有请求

当DispatcherServlet启动时，它会创建Spring应用的上下文，并加载配置文件或配置类中所申明的bean。我们要求DispatcherServlet加载应用上下文时，使用定义在ServletConfig配置类中的bean。通常还会有另一个应用上下文，由ContextLoaderListener创建

我们希望DispatcherServlet加载包含Web组件的bean，如控制器、视图解析器以及处理映射器；而ContextLoaderListener要加载应用中其他的bean，通常是驱动应用后端的中间层和数据层组件

getServletConfigClasses方法返回的带有@Configuration注解的类将会用来定义DispatcherServlet上下文的bean，getRootConfigClasses方法返回的带有@Configuration注解的类将会用来配置ContextLoaderListener创建的应用上下文的bean

#### 2：启用Spring MVC

启用 Spring MVC 组件的方法也不仅一种，可以再xml文件中使用注解驱动:"<mvc:annotation-driven>"，也可在配置类上使用@EnableWebMvc注解。

@EnableWebMvc注解存在三个问题：
①：没有配置视图解析器
②：没有启动组件扫描
③：由于选择的是默认Servlet，所以会处理所有的请求，包括对静态资源的请求，大多时候这并不是我们所需要的

配置WebMvcConfigurer：
	/**

	* @Description:    用来定义DispatcherServlet应用上下文的bean

	* @Author:         HJ

	* @CreateDate:     2018/10/11 上午10:31

	* @UpdateUser:     HJ

	* @UpdateDate:     2018/10/11 上午10:31

	* @UpdateRemark:   修改内容

	* @Version:        1.0

	*/
	package cn.funnyhuang.config;

	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.servlet.ViewResolver;
	import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
	import org.springframework.web.servlet.config.annotation.EnableWebMvc;
	import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
	import org.springframework.web.servlet.view.InternalResourceViewResolver;

	@Configuration
	/**
 	* EnableWebMvc注解存在的问题：
 	* 1: 没有配置视图解析器
 	* 2: 没有开启组件扫描
 	* 3: DispatcherServlet会映射默认的servlet，所以会处理所有的请求，包括对静态资源的请求，比如图片和样式表等
 	*/
	@EnableWebMvc
	/**
 	* 启动组件扫描
 	*/
	@ComponentScan(basePackages = {"cn.funnyhuang.controller"})
	public class ServletConfig implements WebMvcConfigurer {

    	/**
     	* 配置JSP视图解析器
     	* @return
     	*/
    	@Bean
    	public ViewResolver viewResolver() {
        	InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        	//设置前缀
        	resolver.setPrefix("WEB-INF/views/");
        	//设置后缀
        	resolver.setSuffix(".jsp");
        	resolver.setExposeContextBeansAsAttributes(true);
        	return resolver;
    	}

    	/**
     	* 配置静态资源的处理
     	* 实现WebMvcConfigurer的configureDefaultServletHandling方法
     	*
     	* @param configurer
     	*/
    	@Override
    	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) 	{
        	//要求DispatchServlet将对静态的资源请求转发到Servlet容器默认的servlet中，
        	//而不是使用DispatchServlet本身来处理此请求。
        	configurer.enable();
    	}
	}

配置简单的RootConfig:

	/**

	* @Description:    配置ContextLoaderListener创建的应用上下文的bean

	* @Author:         HJ

	* @CreateDate:     2018/10/11 上午10:32

	* @UpdateUser:     HJ

	* @UpdateDate:     2018/10/11 上午10:32

	* @UpdateRemark:   修改内容

	* @Version:        1.0

	*/
	package cn.funnyhuang.config;

	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.context.annotation.FilterType;
	import org.springframework.web.servlet.config.annotation.EnableWebMvc;

	@Configuration
	@ComponentScan(basePackages = {"cn.funnyhuang"},
        	excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class)})
	public class RootConfig {

	}


### 编写基本的控制器

	package cn.funnyhuang.controller;

	import org.springframework.stereotype.Controller;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RequestMethod;

	/**

 	* @Description:    java类作用描述 基本的控制器请求

 	* @Author:         HJ

 	* @CreateDate:     2018/10/12 上午10:54

 	* @UpdateUser:     HJ

 	* @UpdateDate:     2018/10/12 上午10:54

 	* @UpdateRemark:   修改内容

 	* @Version:        1.0

 	*/
	@Controller
	public class HomeController {

    	@RequestMapping(value = "/getHome", method = RequestMethod.GET)
    	public String home() throws Exception {
        	return "home";
    	}
	}

@Controller 注解是用来声明控制器的，目的就是辅助实现组件扫描
@RequestMapping注解Value属性用来指定这个方法所处理的请求路径，method属性处理HTTP请求方式
在方法中，只是返回了一个字符串，这个字符串将会被Spring MVC解读为要渲染的视图名称。DispatcherServlet会要求视图解析器将这个逻辑视图解析为实际的视图

jsp代码

	<%@ taglib prefix="c" uri="http://www.springframework.org/tags" %>
	<%--
  	Created by IntelliJ IDEA.
  	User: ran
  	Date: 2018/10/11
  	Time: 上午10:39
  	To change this template use File | Settings | File Templates.
	--%>
	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	<head>
    	<title>SpringWeb</title>
    	<link rel="stylesheet"
          	typ="text/css"
          	href="<c:url value="/resources/style.css"/>">
	</head>
	<body>
    	<h1>welcome to SpringWeb</h1>
    	<a href="<c:url value="/baseInfo"/>">springWebInf</a> |
    	<a href="<c:url value="/springWeb/register"/> ">Register</a>
	</body>
	</html>


测试控制器

	package cn.funnyhuang.controller;

	import org.junit.Before;
	import org.junit.Test;
	import org.springframework.test.web.servlet.MockMvc;


	import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;
	import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
	import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;

	public class HomeControllerTest {
    	private HomeController homeController;

    	private MockMvc mockMvc;
    	@Before
   		public void setUp() throws Exception {
        	homeController = new HomeController();
        	mockMvc = standaloneSetup(homeController).build();
    	}

    	@Test
    	public void home() throws Exception  {
        	/**
         	* RequestMapping的路径不要和jsp文件名称相同
         	* value = "/home" jsp文件名称home，便会报错
         	*/
        	mockMvc.perform(get("/getHome")).andExpect(view().name("home"));
    	}
	}

首先传递一个控制器实例到MockMvcBuilders.standaloneSetup()并调用build()来构建MockMvc实例，然后执行Get请求并设置期望得到的视图名称
