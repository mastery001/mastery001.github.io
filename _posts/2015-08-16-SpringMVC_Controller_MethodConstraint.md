---
layout: post
category: Spring
title: springmvc的Controller的方法约束
tagline: by Mastery
tags: [Spring]
---

<!--more-->

## Controller内的方法限制传递的参数类型

	(1)`HttpServlet`的内置对象。{`ServletRequest`,`HttpServletRequest`,`ServletResponse`,`HttpServletResponse`}
	
    (2)`HttpSession`对象：当传递这种参数类型时必须保证会话是存在的，不能为null。
		Notes：Session访问可能不是线程安全的，特别是在一个Servlet环境。如果多个请求被允许同时访问一个会话,考虑设置	
		RequestMappingHandlerAdapter的“synchronizeOnSession”标志为“true”，

	(3)`org.springframework.web.context.request.WebRequest`或者`org.springframework.web.context.request.NativeWebRequest`。
	
	(4)`java.util.Locale`：当前请求的语言环境，LocaleResolver在Servlet环境中配置

    (5)`java.io.InputStream`/`java.io.Reader`：request中的内容

    (6)`java.io.OutputStream` / `java.io.Writer`：用于生成response对象中的内容

    (7)`java.security.Principal`:包含当前身份验证的用户

    (8)`HttpEntity<?>`:用于接收Http的请求头中的内容（headers和contents）.通过HttpMessageConverter来将请求数据转换成实体。

    (9)`java.util.Map` / `org.springframework.ui.Model` / `org.springframework.ui.ModelMap`:将后台需要的数据传递到视图层

    (10)`org.springframework.web.servlet.mvc.support.RedirectAttributes`:当完成一个数据库对应的操作时，需要给前台做出
    一个响应，为了防止刷新重复提交表单，不能采用request的方式，所以采用这种方式将信息临时存储在服务器端，使其可用于
    重定向之后的请求.

    (11)`org.springframework.validation.Errors` / `org.springframework.validation.BindingResult`:这个参数必须紧接在配置
    了@Valid注解的参数之后，有多少个配置了@Valid注解的参数就有多少个该对象。用于验证一个表单对象或者结果前的命令

    (12)`org.springframework.web.bind.support.SessionStatus`：可以通过该类型 status 对象显式结束表单的处理，
    这相当于触发 session 清除其中的通过 @SessionAttributes 定义的属性

    (13)`org.springframework.web.util.UriComponentsBuilder`:针对当前请求的主机，端口，scaheme,context path和servlet
    对应的映射路径来构造uri

下面是一些可传递的注解类型的参数：

    (1)`@PathVariable`:注解用于访问uri上的模板参数，可接收一些基本类型，如int，long，String，Date等基本类型

    (2)`@MatrixVariable`:注解用于访问uri上的若干组键值对参数；该注解传递参数的uri有如下格式：
        1."/cars;color=red;year=2012"
        2. "color=red,green,blue"
        3."color=red;color=green;color=blue"

    (3)`@RequestParam`:注解用于访问request请求中参数，并会将值自动转换成注解对应的类型。

    (4)`@RequestHeader`:注解用于访问request HTTP请求头信息，并且值也会做相应的转换。

    (5)`@RequestBody`:注解用于访问HTTP请求体中的内容。值通过HttpMessageConverter来转换。

    (6)`@RequestPart`:注解用于访问当表单的enctype设置为'multipart/form-data'时的请求内容。

## Controller内的方法支持的返回类型

    (1)`ModelAndView Object`:返回对应的视图层和Model层的数据.

    (2)`Model Object`:返回对应设定的Model层的数据,主要包含Spring封装好的model和modelMap，以及java.util.Map
    当没有视图返回时视图名称由RequestToViewNameTranslator决定，一般是配置了RequestMapping的映射路径。

    (3)`Map Object`:与Model类型相似。

    (4)`View Object`:返回对应的视图对象，并且可以通过`render(Map<String ,?> , request , response)`参数向前台传递
    属性。

    (5)`String Object`:返回一个视图名称，Spring会通过相应的ViewResolver来解析

    (6)`void`:返回空值。

    (7)`other type`:其他任何被认为不是属于上述的类型的返回类型，使用@ModelAttribute在方法级别上指定名称。

>以上内容仅属于个人观点，有错误的地方希望各位同仁能加以指正！谢谢！
