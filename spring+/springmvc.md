[TOC]

# Springmvc

## 简介

* Spring web mvc和Struts2都属于表现层的框架,它是Spring框架的一部分。

* 处理流程：

  ![img](file:///C:/Users/KANGHA~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.gif)

## 快速入门

1. 导入jar包

   ![1554557948221](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554557948221.png)

2. 配置前段控制器

   ~~~xml
   <!-- 前端控制器 -->
     <servlet>
     	<servlet-name>springmvc</servlet-name>
     	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     	<init-param>
     		<param-name>contextConfigLocation</param-name>
     		<param-value>classpath:springmvc.xml</param-value>
     	</init-param>
     </servlet>
     <servlet-mapping>
     	<servlet-name>springmvc</servlet-name>
     	<url-pattern>*.action</url-pattern>
     </servlet-mapping>
   
   ~~~

   

3. 创建一个Controller

   * 有两种方法：1.实现Conroller接口；2.添加@Controller

     ~~~java
     @Controller
     public class ItemController {
     
     	@RequestMapping("/itemList")
     	public ModelAndView itemList() throws Exception {
     		
     		List<Items> itemList = new ArrayList<>();
     		
     		//商品列表
     		Items items_1 = new Items();
     		items_1.setName("联想笔记本_3");
     		items_1.setPrice(6000f);
     		items_1.setDetail("ThinkPad T430 联想笔记本电脑！");
     		
     		Items items_2 = new Items();
     		items_2.setName("苹果手机");
     		items_2.setPrice(5000f);
     		items_2.setDetail("iphone6苹果手机！");
     		
     		itemList.add(items_1);
     		itemList.add(items_2);
     		//创建modelandView对象
     		ModelAndView modelAndView = new ModelAndView();
     		//添加model
     		modelAndView.addObject("itemList", itemList);
     		//添加视图
     		modelAndView.setViewName("/WEB-INF/jsp/itemList.jsp");
     //		modelAndView.setViewName("itemsList");	
     		return modelAndView;
     	}
     
     }
     
     ~~~

     

4. 将Controller配置到springmvc.xml中

   ~~~xml
   	<bean id="itemsController" name="/itemsList.action" class="cn.itcast.mvc.controller.ItmesController"/>
   ~~~

   

## Spingmvc的架构

### 架构图

![img](file:///C:/Users/KANGHA~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.gif)

 ### 执行流程

1. 用户发送请求至前端控制器DispatcherServlet

2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

5. 执行处理器(Controller，也叫后端控制器)。

6. Controller执行完成返回ModelAndView

7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9. ViewReslover解析后返回具体View

10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

11. DispatcherServlet响应用户

 

### 组件

*  DispatcherServlet：前端控制器

  用户请求到达前端控制器，它就相当于mvc模式中的c，dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet的存在降低了组件之间的耦合性。

* HandlerMapping：处理器映射器

  HandlerMapping负责根据用户请求找到Handler即处理器，springmvc提供了不同的映射器实现的不同映射方式，例如：配置文件方式，实现接口方式，注解方式等。

  HandlerMapping 负责根据request请求找到对应的Handler处理器及Interceptor拦截器，将它们封装在HandlerExecutionChain 对象中给前端控制器返回。

* Handler：处理器

  Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。
  由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。

* HandlAdapter：处理器适配器

  通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

* View Resolver：视图解析器

  View
  Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户

* View：视图

  springmvc框架提供了很多的View视图类型的支持，包括：jstlView、freemarkerView、pdfView等。我们最常用的视图就是jsp。

#### 非注解的处理器映射器和适配器

* BeanNameUrlHandlerMapping

  BeanNameUrl处理器映射器，根据请求的url与spring容器中定义的bean的name进行匹配，从而从spring容器中找到bean实例。

* SimpleControllerHandlerAdapter

  简单控制器处理器适配器，所有实现了org.springframework.web.servlet.mvc.Controller 接口的Bean通过此适配器进行适配、执行。

#### 注解映射器和适配器

* 注解扫描器

  使用组件扫描器省去在spring容器配置每个controller类的繁琐。使用<context:component-scan自动扫描标记@controller的控制器类，配置如下：

  ~~~xml
  <!-- 扫描controller注解,多个包中间使用半角逗号分隔 -->
  	<context:component-scan base-package="cn.itcast.springmvc.controller.first"/>
  
  ~~~

  

* RequestMappingHandlerMapping

  注解式处理器映射器，对类中标记@ResquestMapping的方法进行映射，根据ResquestMapping定义的url匹配ResquestMapping标记的方法，匹配成功返回HandlerMethod对象给前端控制器，HandlerMethod对象中封装url对应的方法Method。 

  从spring3.1版本开始，废除了DefaultAnnotationHandlerMapping的使用，推荐使用RequestMappingHandlerMapping完成注解式处理器映射。

  ~~~xml
  <!--注解映射器 -->
  	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
  
  ~~~

  

* RequestMappingHandlerAdapter

  注解式处理器适配器，对标记@ResquestMapping的方法进行适配。

  从spring3.1版本开始，废除了AnnotationMethodHandlerAdapter的使用，推荐使用RequestMappingHandlerAdapter完成注解式处理器适配。

  ~~~xml
  <!--注解适配器 -->
  	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
  
  ~~~

  

* <mvc:annotation-driven>

  springmvc使用<mvc:annotation-driven>自动加载RequestMappingHandlerMapping和RequestMappingHandlerAdapter，可用在springmvc.xml配置文件中使用<mvc:annotation-driven>替代注解处理器和适配器的配置。

#### 视图解析器

~~~xml
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="viewClass"
			value="org.springframework.web.servlet.view.JstlView" />
		<property name="prefix" value="/WEB-INF/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>

~~~

InternalResourceViewResolver：支持JSP视图解析

viewClass：JstlView表示JSP模板页面需要使用JSTL标签库，所以classpath中必须包含jstl的相关jar 包。此属性可以不设置，默认为JstlView。

prefix 和suffix：查找视图页面的前缀和后缀，最终视图的址为：

前缀+**逻辑视图名**+后缀，逻辑视图名需要在controller中返回ModelAndView指定，比如逻辑视图名为hello，则最终返回的jsp视图地址 “WEB-INF/jsp/hello.jsp”

## 高级参数绑定

#### 绑定数组

Controller方法中可以用String[]接收，或者pojo的String[]属性接收。两种方式任选其一即可

定义如下：

~~~~java
@RequestMapping("/queryitem")
	public String queryItem(QueryVo queryVo, String[] ids) {
		System.out.println(queryVo.getItems().getName());
		System.out.println(queryVo.getItems().getPrice());
		System.out.println(ids.toString());
		return null;
	}

~~~~

#### 绑定集合

**注意** ： 接收List类型的数据必须是pojo的属性，方法的形参为List类型无法正确接收到数据。

~~~java
@RequestMapping("/queryitem")
	public String queryItem(QueryVo queryVo, String[] ids) {
		System.out.println(queryVo.getItems().getName());
		System.out.println(queryVo.getItems().getPrice());
		System.out.println(ids.toString());
		return null;
	}

~~~

## @RequestMapping注解详解

* 路径映射

  @RequestMapping(value="/item")或@RequestMapping("/item）
  value的值是数组，可以将多个url映射到同一个方法

* 窄化请求映射

  在class上添加@RequestMapping(url)指定通用请求前缀， 限制此类下的所有方法请求url必须以请求前缀开头，通过此方法对url进行分类管理。

  如：

  ~~~java
  @RequestMapping放在类名上边，设置请求前缀 
  @Controller
  @RequestMapping("/item")
  
  方法名上边设置请求映射url：
  @RequestMapping放在方法名上边，如下：
  @RequestMapping("/queryItem ")
  
  ~~~

  最终访问地址为：/item/queryItem

  

* 请求方法限定

  ~~~java
  //限定GET
  @RequestMapping(method = RequestMethod.GET)
  
  如果通过Post访问则报错：
  HTTP Status 405 - Request method 'POST' not supported
  
  例如：
  @RequestMapping(value="/editItem",method=RequestMethod.GET)
  //限定POST方法
  
  @RequestMapping(method = RequestMethod.POST)
  
  //如果通过Post访问则报错：
  HTTP Status 405 - Request method 'GET' not supported
  
  //GET和POST都可以
  @RequestMapping(method={RequestMethod.GET,RequestMethod.POST})
  
  ~~~

  

## Controller方法的返回值

* 返回ModelAndView

  controller方法中定义ModelAndView对象并返回，对象中可添加model数据、指定view。

* 返回void

  在controller方法形参上可以定义request和response，使用request或response指定响应结果：

  1. 使用request转向页面，如下：

  request.getRequestDispatcher("页面路径").forward(request, response);

   

  2. 也可以通过response页面重定向：

  response.sendRedirect("url")

   

  3. 也可以通过response指定响应结果，例如响应json数据如下：

  response.setCharacterEncoding("utf-8");

  response.setContentType("application/json;charset=utf-8");

  response.getWriter().write("json

  串");

* 返回字符串

  * 逻辑视图名

    **return** "item/editItem";

  * Redirect重定向

    **return** "redirect:queryItem.action";

  * 转发

    return "forward:editItem.action";

## JSON数据交互

* @RequestBody

  @RequestBody注解用于读取http请求的内容(字符串)，通过springmvc提供的HttpMessageConverter接口将读到的内容转换为json、xml等格式的数据并绑定到controller方法的参数上.

* @ResponseBody

  该注解用于将Controller的方法返回的对象，通过HttpMessageConverter接口转换为指定格式的数据如：json,xml等，通过Response响应给客户端.

## RESTFUL支持

	Restful就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格，是对http协议的诠释。
资源定位：互联网所有的事物都是资源，要求url中没有动词，只有名词。没有参数
Url格式：http://blog.csdn.net/beat_the_world/article/details/45621673 4S
资源操作：使用put、delete、post、get，使用不同方法对资源进行操作。分别对应添加、删除、修改、查询。一般使用时还是post和get。Put和Delete几乎不使用

### 使用

1. 添加DispatcherServlet的rest配置

   ~~~xml
   <servlet>
   		<servlet-name>springmvc-servlet-rest</servlet-name>
   		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   		<init-param>
   			<param-name>contextConfigLocation</param-name>
   			<param-value>classpath:spring/springmvc.xml</param-value>
   		</init-param>
   	</servlet>
   	<servlet-mapping>
   		<servlet-name>springmvc-servlet-rest</servlet-name>
   		<url-pattern>/</url-pattern>
   	</servlet-mapping>
   
   ~~~

   

2. 编写Controller

   ~~~java
   @RequestMapping("/viewItems/{id}") 
   	public @ResponseBody viewItems(@PathVariable("id") String id,Model model) throws Exception{
   		//方法中使用@PathVariable获取useried的值，使用model传回页面
   		//调用 service查询商品信息
   		ItemsCustom itemsCustom = itemsService.findItemsById(id);
   		return itemsCustom;
   }
   
   ~~~

   @RequestMapping(value="/ viewItems/{id}")：{×××}占位符，请求的URL可以是“/viewItems/1”或“/viewItems/2”，通过在方法中使用@PathVariable获取{×××}中的×××变量。

   @PathVariable

   用于将请求URL中的模板变量映射到功能处理方法的参数上。

   如果RequestMapping中表示为"/viewItems/{id}"，id和形参名称一致，@PathVariable不用指定名称。