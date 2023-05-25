---
layout: post
title: 'SpringBoot特性1'
date: 2023-05-24
description: 'SpringBoot特性1'
tags: 'SpringBoot'
--- 

## Web开发
Spring Boot Web 开发非常的简单，其中包括常见的json输出、filters、property、log等

### josn接口开发
在以前使用Spring开发项目，需要提供json接口时需要做哪些配置呢

> 1. 添加jackjson等相关jar包
> 2. 配置Spring Controller扫描
> 3. 对接的方法添加@ResponseBody

就这样我们会经常由于配置错误，导致406错误等等，Spring Boot如何做呢，只需要类添加`@RestController`即可，默认类中的方法都会以json的格式返回

```java
@RestController
public class HelloController {
    @RequestMapping("/getUser")
    public User getUser() {
    	User user=new User();
    	user.setUserName("小明");
    	user.setPassWord("xxxx");
        return user;
    }
}
```

### 自定义Filter
我们常常在项目中会使用 filters 用于录调用日志、排除有XSS威胁的字符、执行权限验证等等。SpringBoot 自动添加了OrderedCharacterEncodingFilter 和 HiddenHttpMethodFilter，并且我们可以自定义 Filter。

两个步骤：
> 1. 实现Filter接口，实现Filter方法
> 2. 添加`@Configuration`注解，将自定义Filter加入过滤链

```java
@Configuration
public class WebConfiguration {
    //把对象的创建交给Bean创建,这样我们就能通过spring来获取对象
    @Bean
    public RemoteIpFilter remoteIpFilter() {
        return new RemoteIpFilter();
    }

    //FilterRegistrationBean过滤器注册Bean
    @Bean
    public FilterRegistrationBean testFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new MyFilter());
        registration.addUrlPatterns("/*");
        registration.addInitParameter("paramName","paramValue");
        registration.setName("MyFilter");
        registration.setOrder(1);
        return registration;
    }

    public class MyFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
        }

        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest servletRequest = (HttpServletRequest)request;
            System.out.println("this is MyFilter,url:"+((HttpServletRequest) request).getRequestURI());
            chain.doFilter(request,response);
        }

        @Override
        public void destroy() {
        }
    }
}
```

### 自定义Property
在Web开发的过程中，我经常需要自定义一下配置文件，如何使用呢

```
com.neo.title=纯洁的微笑
com.neo.description=分享生活和技术
```

自定义配置类
```java
@Component
public class NeoProperties {
	@Value("${com.neo.title}")
	private String title;
	@Value("${com.neo.description}")
	private String description;

	//省略getter settet方法

	}
```
