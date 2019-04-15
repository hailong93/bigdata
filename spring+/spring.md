[TOC]

# Spring

## 概述

### 简介

​	是轻量级的SE/EE的一站式的框架.

### 优点

1. 方便解耦，简化开发
2. AOP编程的支持
3. 声明式事务的支持
4. 方便程序测试

## 入门

1. 导入jar包：spring-framework-4.2.4.RELEASE-dist.zip

## 主要功能

### IOC

* 概念

  Inversion of Control控制反转.控制反转指的是 将对象的创建权 反转(交给) 给Spring.

#### Spring的工厂类

![img](file:///C:/Users/KANGHA~1/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

ApplicationContext是新版本的使用工厂类的对象.

- ClassPathXmlApplicationContext		:加载classpath下面的applicationContext.xml
- FileSystemXmlApplicationContext		:加载文件系统上的applicationContext.xml

#### 实例化Bean

1. 使用无参的构造方法实例化bean

   ~~~xml
   <!-- 无参数的构造方法的实例化 -->
   <bean id="bean1" class="com.spring.demo2.Bean1"></bean>
   
   ~~~

2. 静态工厂实例化bean

   ~~~xml
   <!-- 静态工厂实例化Bean -->
   <bean id="bean2" class="com.test.spring.demo2.Bean2Factory" factory-method="getBean2"></bean>
   
   ~~~

3. 实例工厂实例化bean

~~~xml
    <!-- 实例工厂实例化Bean -->
    <bean id="bean3Factory" class="com.test.spring.demo2.Bean3Factory"></bean>
    <bean id="bean3" factory-bean="bean3Factory" factory-method="getBean3"></bean>

~~~

#### bean的配置

* 格式：<bean id="" name="" class="" scope=""/>

* id:使用了唯一约束，id属性中不能使用特殊字符。
* name:没有使用唯一约束，配置name也要是唯一的. name中可以使用特殊字符的
* scope:
  * singleton		:单例的.
  * prototype		:多例的.
  * request		:应用在web工程中的,创建一个Bean的实例,将Bean的实例存入到request的域中.
  * session		:应用在web工程中的,创建一个Bean的实例,将Bean的实例存入到session的域中.
  * globalsession	:应用在web工程中的,集群环境.如果没有集群环境的话,配置globalsession与session一致

### DI

* 概念

  Dependency Injection.依赖注入.DI需要有IOC的环境的. 依赖注入指的是 在Spring创建对象的过程中,将对象的依赖的属性注入(设置)进去.

#### 属性注入

1. 构造方法属性注入

   ~~~xml
   <bean id="car" class="com.test.spring.demo5.Car">
       <!-- 构造方法注入 -->
       <constructor-arg name="name" value="奔驰"/>
       <constructor-arg name="price" value="1000000"/>
   </bean>
   
   ~~~

   

2. set方法属性注入

   ~~~xml
   	<!-- 第二种:set方法的属性注入 -->
   	<bean id="car2" class="com.test.spring.demo5.Car2">
   		<property name="name" value="宝马"></property>
   		<property name="price" value="500000"></property>
   	</bean>
   
   ~~~

   

3. 注入对象的类型

~~~xml
	<bean id="man" class="com.test.spring.demo5.Man">
		<property name="name" value="涛哥"></property>
		<!-- ref:引用另一个对象的id或name -->
		<property name="car2" ref="car2"></property>
	</bean>

~~~

4. Spring 2.5版本时提供了p名称空间的注入

~~~xml
    引入p名称空间：
    <beans 
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:p="http://www.springframework.org/schema/p"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="car2" class="com.itheima.spring.demo5.Car2" p:name="奥拓" p:price="30000"></bean>
    <bean id="man" class="com.itheima.spring.demo5.Man" p:name="小军军" p:car2-ref="car2"></bean>

~~~

5. Spring3.0之后提供了SpEL的注入

   ~~~xml
   SpEL:Spring Excepress Language.
   语法:#{ SpEL }
       <bean id="car2" class="com.itheima.spring.demo5.Car2">
       	<property name="name" value="#{'奥迪'}"></property>
       	<property name="price" value="#{'300000'}"></property>
       </bean>
   
   	<bean id="man" class="com.itheima.spring.demo5.Man">
   		<property name="name" value="#{'小凤'}"></property>
   		<property name="car2" value="#{car2}"></property>
   	</bean>
   
   ~~~

6. 复杂类型的注入

   ~~~xml
   	<!-- 注入复杂类型: -->
   	<bean id="collectionBean" class="com.itheima.spring.demo6.CollectionBean">
   		<property name="arrs">
   			<list>
   				<value>li</value>
   				<value>李凤</value>
   				<value>李娇</value>
   			</list>
   		</property>
   		
   		<property name="list">
   			<list>
   				<value>xiao</value>
   				<value>肖如花</value>
   				<value>肖娇</value>
   			</list>
   		</property>
   		
   		<property name="map">
   			<map>
   				<entry key="aaa" value="小军军"></entry>
   				<entry key="bbb" value="小童童"></entry>
   				<entry key="ccc" value="小花花"></entry>
   			</map>
   		</property>
   		
   		<property name="properties">
   			<props>
   				<prop key="username">root</prop>
   				<prop key="password">123</prop>
   			</props>
   		</property>
   	</bean>
   
   ~~~

   

### Spring 注解

#### bean管理相关

* @Component("userService")  相当于 <bean id="userService" class=""/>

  在核心配置中配置：组件的扫描：

  <context:component-scan base-package="com.test.spring.demo1"><</context:component-scan>

  * Controller       WEB层
  * Service           业务层
  * Repository      持久层

#### 属性注入相关

* @Value           用于注入普通值
* @Autowired      :对象类型注入，默认按类型进行注入的

* @Autowired 和 @Qualifier一起使用的效果可以被@Resource的注解替换
* @Resource(name="customerDao")
  等价于
  @Autowired
  @Qualifier("customerDao")

* @Scope(value="prototype")

### AOP

#### 概述

AOP为Aspect Oriented Programming的缩写，面向切面编程。通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。

* 好处：AOP采取**横向抽取机制**，取代了传统**纵向继承体系**重复性代码（性能监视、事务管理、安全检查、缓存）

* 相关术语
  * Joinpoint(连接点):所谓连接点是指那些被拦截到的点。在spring中,这些点指的是方法,因为spring只支持方法类型的连接点.
  * Pointcut(切入点):所谓切入点是指我们要对哪些Joinpoint进行拦截的定义.
  * Advice(通知/增强):所谓通知是指拦截到Joinpoint之后所要做的事情就是通知.通知分为前置通知,后置通知,异常通知,最终通知,环绕通知(切面要完成的功能)
  * Introduction(引介):引介是一种特殊的通知在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field.
  * Target(目标对象):代理的目标对象
  * Weaving(织入):是指把增强应用到目标对象来创建新的代理对象的过程.
    	spring采用动态代理织入，而AspectJ采用编译期织入和类装载期织入
  * Proxy（代理）:一个类被AOP织入增强后，就产生一个结果代理类
  * Aspect(切面): 是切入点和通知（引介）的结合

* 底层实现：代理机制。

#### 底层实现

* JDK动态代理:

* 对实现了接口的类才能生成代理.

* CGLIB动态代理:

* 对没有实现接口的类产生代理.产生这个类的子类对象.

**Spring**的传统AOP中根据类是否实现接口

* 如果实现了接口   :使用JDK动态代理完成AOP.

* 如果没有实现接口 :采用CGLIB动态代理完成AOP.

* jdk的动态代理代码

  ~~~java
  public class MyJdkProxy implements InvocationHandler {
  
  	private UserDao userDao;
  
  	public MyJdkProxy(UserDao userDao) {
  		this.userDao = userDao;
  	}
  
  	public UserDao createProxy() {
  		UserDao proxy = (UserDao) Proxy.newProxyInstance(userDao.getClass()
  				.getClassLoader(), userDao.getClass().getInterfaces(), this);
  		return proxy;
  	}
  
  	@Override
  	public Object invoke(Object proxy, Method method, Object[] args)
  			throws Throwable {
  		if("save".equals(method.getName())){
  			System.out.println("==================权限校验===============");
  			return method.invoke(userDao, args);
  		}
  		return method.invoke(userDao, args);
  	}
  }
  
  ~~~

  

* CGLIB动态代理

  ~~~java
  public class MyCglibProxy implements MethodInterceptor{
  	
  	private OrderDao orderDao;
  	
  	public MyCglibProxy(OrderDao orderDao) {
  		this.orderDao = orderDao;
  	}
  
  	public OrderDao createProxy(){
  		// 1.创建一个CGLIB的核心类:
  		Enhancer enhancer = new Enhancer();
  		// 2.设置父类:
  		enhancer.setSuperclass(orderDao.getClass());
  		// 3.设置回调:
  		enhancer.setCallback(this);
  		// 4.生成代理 :
  		OrderDao proxy = (OrderDao) enhancer.create();
  		return proxy;
  	}
  
  	@Override
  	public Object intercept(Object proxy, Method method, Object[] args,
  			MethodProxy methodProxy) throws Throwable {
  		if("update".equals(method.getName())){
  			long begin = System.currentTimeMillis();
  			System.out.println("开始时间:=========="+begin);
  			
  			Object obj = methodProxy.invokeSuper(proxy, args);
  			
  			long end = System.currentTimeMillis();
  			System.out.println("结束时间:=========="+end);
  			return obj;
  		}
  		return methodProxy.invokeSuper(proxy, args);
  	}
  }
  
  ~~~

  

#### 传统AOP开发

##### 定义通知类型

Spring按照通知Advice在目标类方法的连接点位置，可以分为5类

* 前置通知(在方法之前执行) org.springframework.aop.MethodBeforeAdvice
  在目标方法执行前实施增强
* 后置通知(在方法之后执行) org.springframework.aop.AfterReturningAdvice
  在目标方法执行后实施增强
* 环绕通知 org.aopalliance.intercept.MethodInterceptor
  在目标方法执行前后实施增强
* 异常抛出通知 org.springframework.aop.ThrowsAdvice
  在方法抛出异常后实施增强
* 引介通知 org.springframework.aop.IntroductionInterceptor
  在目标类中添加一些新的方法和属性

##### 定义切面的类型

* Advisor 			: 代表一般切面，Advice本身就是一个切面，对目标类所有方法进行拦截.(不带切入点切面:增强所有方法)
* PointcutAdvisor : 代表具有切点的切面，可以指定拦截目标类哪些方法.(带有切入点切面:增强某些方法)
* IntroductionAdvisor : 代表引介切面，针对引介通知而使用切面

#### 传统不带切入点切面的开发

1. 引入jar包

   com.springsource.org.aopalliance-1.0.0.jar   spring-aop-3.2.0.RELEASE.jar

2. 创建类和接口

   * com.itheima.spring.demo3
     * ProductDao
     * ProductDaoImpl

3. 配置bean

   ~~~xml
   <bean id="productDao" class="com.itheima.spring.demo3.ProductDaoImpl"/>
   ~~~

4. 定义增强:(使用一般切面:不带切入点的切面增强类中的所有的方法:增强本身就是一个切面.)

   ~~~java
   public class MyBeforeAdvice implements MethodBeforeAdvice{
       @Override
       public void before(Method method, Object[] args, Object target)
               throws Throwable {
           System.out.println("==============前置通知=============");
       }
   
   }
   ~~~

5. 配置通知到Spring中

   ~~~xml
   <bean id="beforeAdvice" class="com.itheima.spring.demo3.MyBeforeAdvice"></bean>
   ~~~

6. :配置对DAO生成代理

   ~~~xml
    <!-- 配置生成代理 -->
   
    <bean id="productDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
   
   	<!-- 配置目标类 -->
   
        <property name="target" ref="productDao"/>
   
       <!-- 配置类的实现的接口 -->
   
       <property name="proxyInterfaces" value="com.itheima.spring.demo3.ProductDao"/>
   
       <!-- 配置切面要拦截的名称 -->
   
       <property name="interceptorNames" value="beforeAdvice"/>
   
    </bean>
   ~~~

   ProxyFactoryBean常用可配置属性

   ​	target           : 代理的目标对象

   ​	proxyInterfaces : 代理要实现的接口

   ​	如果多个接口可以使用以下格式赋值

   ​	<list>

   ​    	<value></value>

      	 ....

   ​	</list>

   ​	proxyTargetClass : 是否对类代理而不是接口，设置为true时，使用CGLib代理

   ​	interceptorNames : 需要织入目标的Advice

   ​	singleton         : 返回代理是否为单实例，默认为单例

   ​	optimize         : 当设置为true时，强制使用CGLib

7. 编写测试类

   ~~~java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:applicationContext.xml")
   public class SpringDemo3 {
   
   	// @Resource(name = "productDao")
   	// 注入代理类:
   	@Resource(name="productDaoProxy")
   	private ProductDao productDao;
   
   	@Test
   	public void demo1() {
   		productDao.save();
   		productDao.update();
   		productDao.delete();
   		productDao.find();
   	}
   }
   
   ~~~

   

#### 传统带切入点切面的开发

1. 1-5步同上

2. 配置切面:(带有切入点的切面)

   ~~~xml
   	<!-- 配置带有切入点的切面 -->
   	<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
   		<!-- 表达式: 正则表达式 : .:任意字符  *:任意次数-->
   		<property name="pattern" value=".*"/>
   		<!-- 配置增强 -->
   		<property name="advice" ref="myAroundAdvice"/>
   	</bean>
   ~~~

3. 配置DAO代理

   ~~~xml
   	<!-- 配置生成代理 -->
   	<bean id="customerDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
   		<!-- 配置目标 -->
   		<property name="target" ref="customerDao"/>
   		<!-- 配置代理目标类 -->
   		<property name="proxyTargetClass" value="true"/>
   		<!-- 配置拦截的名称 -->
   		<property name="interceptorNames" value="myAdvisor"/>
   	</bean>
   
   ~~~

   

4. 编写测试类

   ~~~java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:applicationContext.xml")
   public class SpringDemo4 {
   
   	// @Resource(name = "customerDao")
   	// 注入代理对象:
   	@Resource(name="customerDaoProxy")
   	private CustomerDao customerDao;
   
   	@Test
   	public void demo1() {
   		customerDao.save();
   		customerDao.update();
   		customerDao.delete();
   		customerDao.find();
   	}
   }
   
   ~~~

   

#### Spring的传统的AOP的开发:基于ProxyFactoryBean的代理类

* 缺点

   配置麻烦:

  需要为每一个要增强的类配置一个

  ProxyFactoryBean

#### Spring的传统AOP的开发:自动代理

##### 方式

* BeanNameAutoProxyCreator 			:根据Bean名称创建代理 
* DefaultAdvisorAutoProxyCreator 	:根据Advisor本身包含信息创建代理
* AnnotationAwareAspectJAutoProxyCreator :基于Bean中的AspectJ 注解进行自动代理

##### 与ProxyFactoryBean的不同

​	自动代理都是基于BeanPostProcessor:
* 在类的生成过程中就产生了代理.

  基于ProxyFactoryBean代理:

* 先有被增强类 ,将被增强的类传递给ProxyFactoryBean ,生成带代理对象

##### BeanNameAutoProxyCreator

~~~~xml
     只留下目标类和通知:
	<!-- 配置目标类: -->
	<bean id="productDao" class="com.itheima.spring.demo3.ProductDaoImpl"/>
	<bean id="customerDao" class="com.itheima.spring.demo4.CustomerDao"/>
	
	<!-- 配置通知:(前置通知) -->
	<bean id="beforeAdvice" class="com.itheima.spring.demo3.MyBeforeAdvice"/>
	<!-- 配置通知:(环绕通知) -->
	<bean id="myAroundAdvice" class="com.itheima.spring.demo4.MyAroundAdvice"/>

    通过配置完整自动代理:
	<!-- 配置基于Bean名称的自动代理 -->
	<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
		<!-- 配置Bean名称 -->
		<property name="beanNames" value="*Dao"/>
		<!-- 配置拦截的名称 -->
		<property name="interceptorNames" value="beforeAdvice"/>
	</bean>
~~~~

~~~java
    * 测试:
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("classpath:applicationContext2.xml")
    public class SpringDemo5 {
        @Resource(name="productDao")
        private ProductDao productDao;

        @Resource(name="customerDao")
        private CustomerDao customerDao;

        @Test
        public void demo1(){
            productDao.save();
            productDao.update();
            productDao.delete();
            productDao.find();

            customerDao.save();
            customerDao.update();
            customerDao.delete();
            customerDao.find();
        }
    }
~~~

~~~~xm

~~~~

##### DefaultAdvisorAutoProxyCreator

~~~xml
* 只留下目标类和增强:
	<!-- 配置目标类: -->
	<bean id="productDao" class="com.itheima.spring.demo3.ProductDaoImpl"/>
	<bean id="customerDao" class="com.itheima.spring.demo4.CustomerDao"/>
	
	<!-- 配置通知:(前置通知) -->
	<bean id="beforeAdvice" class="com.itheima.spring.demo3.MyBeforeAdvice"/>
	<!-- 配置通知:(环绕通知) -->
	<bean id="myAroundAdvice" class="com.itheima.spring.demo4.MyAroundAdvice"/>

* 配置切面:
	<!-- 配置切面 -->
	<bean id="myAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
		<!-- 表达式 -->
		<property name="pattern" value="com\.itheima\.spring\.demo4\.CustomerDao\.save"/>
		<!-- 配置增强 -->
		<property name="advice" ref="myAroundAdvice"/>
	</bean>

* 配置生成代理:
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

~~~

测试

~~~java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext3.xml")
public class SpringDemo6 {

	@Resource(name="productDao")
	private ProductDao productDao;
	
	@Resource(name="customerDao")
	private CustomerDao customerDao;
	
	@Test
	public void demo1(){
		productDao.save();
		productDao.update();
		productDao.delete();
		productDao.find();
		
		customerDao.save();
		customerDao.update();
		customerDao.delete();
		customerDao.find();
	}
}

~~~

#### 基于AspectJ的注解方式

1. 引入jar包

   com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar  spring-aspects-3.2.0.RELEASE.jar

2. 开发类并引入到spring配置中

   com.itheima.spring.demo1

   ​     OrderDao

   ~~~xml
   <bean id="orderDao" class="com.itheima.spring.demo1.OrderDao"/>
   ~~~

   

3. 编写切面

   * AspectJ提供的通知类型
     * @Before 			前置通知，相当于BeforeAdvic
     * @AfterReturning 	后置通知，相当于AfterReturningAdvice
     * @Around 			环绕通知，相当于MethodInterceptor
     * @AfterThrowing	抛出通知，相当于ThrowAdvice
     * @After 			最终final通知，不管是否异常，该通知都会执行
     * @DeclareParents 引介通知，相当于IntroductionInterceptor 

   * 切入点的表达方式

     * [访问修饰符] 方法返回值 方法名(参数)

       * execution(public * com.itheim.spring.demo1.OrderDao.save(..))

       * execution(* *.*(..))
       * execution(public * com.itheim.spring.demo1.*.*(..))
       * execution(public * com.itheim.spring.demo1..*.*(..))
       * execution(public * com.itheim.spring.demo1.OrderDao+.*(..))

   * 编写切面

     ~~~java
     @Aspect
     public class MyAspectAnno {
     
     	// 定义通知:
     	@Before("execution(* com.itheima.spring.demo1.OrderDao.save(..))")
     	public void before(){
     		System.out.println("前置通知================");
     	}
     }
     
     ~~~

4. 在配置文件中开启注解,并将切面配置到配置文件中

   ~~~xml
   <!-- 开启AspectJ的注解 -->
   	<aop:aspectj-autoproxy/>
   <!-- 配置切面类 -->
   	<bean class="com.itheima.spring.demo1.MyAspectAnno"/>
   
   ~~~

5. 测试

   ~~~java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:applicationContext.xml")
   public class SpringDemo1 {
   
   	@Resource(name="orderDao")
   	private OrderDao orderDao;
   	
   	@Test
   	public void demo1(){
   		orderDao.save();
   		orderDao.update();
   		orderDao.delete();
   		orderDao.find();
   	}
   }
   
   ~~~

6. 总结

    切入点的定义：

~~~~java
        @Pointcut(value="execution(* com.itheima.spring.demo1.OrderDao.find(..))")
        private void pointcut1(){}

~~~~

​	Advisor与Aspect的区别

​		Advisor :Spring传统的切面.一般都是由一个切入点和一个通知的组合.

​		Aspect  :Aspect是真正意义上的切面.是由多个切入点和多个通知组合.

#### AspectJ的XML方式的AOP开发

1. 1-4 跟上面一致

2. 定义切面,并配置

   ~~~java
   public class MyAspectXml {
   
   	public void before(){
   		System.out.println("前置通知==============");
   	}
   }
   
   ~~~

   ~~~xml
   <bean id="myAspectXml" class="com.itheima.spring.demo2.MyAspectXml"/>
   ~~~

   

3.  配置AOP

   ~~~xml
   	<!-- 配置AOP -->
   	<aop:config>
   		<aop:pointcut expression="execution(* com.itheima.spring.demo2.CustomerDao+.save(..))" id="pointcut1"/>
   		<aop:aspect ref="myAspectXml">
   			<aop:before method="before" pointcut-ref="pointcut1"/>
   		</aop:aspect>
   	</aop:config>
   
   ~~~

4. 测试

   ~~~java
   @RunWith(SpringJUnit4ClassRunner.class)
   @ContextConfiguration("classpath:applicationContext2.xml")
   public class SpringDemo2 {
   
   	@Resource(name="customerDao")
   	private CustomerDao customerDao;
   	@Test
   	public void demo1(){
   		customerDao.save();
   		customerDao.update();
   		customerDao.delete();
   		customerDao.find();
   	}
   }
   
   ~~~

5. 总结

   XML方式通知类型的配置

   ~~~xml
   <!-- 配置AOP -->
   	<aop:config>
   		<aop:pointcut expression="execution(* com.itheima.spring.demo2.CustomerDao+.save(..))" id="pointcut1"/>
   		<aop:pointcut expression="execution(* com.itheima.spring.demo2.CustomerDao+.update(..))" id="pointcut2"/>
   		<aop:pointcut expression="execution(* com.itheima.spring.demo2.CustomerDao+.delete(..))" id="pointcut3"/>
   		<aop:pointcut expression="execution(* com.itheima.spring.demo2.CustomerDao+.find(..))" id="pointcut4"/>
   		<aop:aspect ref="myAspectXml">
   			<aop:before method="before" pointcut-ref="pointcut1"/>
   			<aop:after-returning method="afterReturing" pointcut-ref="pointcut2" returning="result"/>
   			<aop:around method="around" pointcut-ref="pointcut3"/>
   			<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut4" throwing="e"/>
   			<aop:after method="after" pointcut-ref="pointcut4"/>
   		</aop:aspect>
   	</aop:config>
   
   ~~~

   

### spring 管理事务

#### 事务

* 定义

  事务是逻辑上的一组操作，组成这组操作的各个逻辑单元，要么一起成功，要么一起失败。

* 特性

  原子性：强调事务的不可分割

  一致性：事务的执行的前后数据的完整性保持一致

  隔离性：一个事务执行的过程中，不应该受到其他事物的干扰

  持久性：事务一旦结束，数据就持久化到数据库

* 不考虑隔离性引发的安全问题

  脏读：一个事务读到了另一个事未提交的数据

  不可重复读：一个事务读到了另一个事务已经提交的update的数据导致多次查询结果不一致。

  虚读：一个事务读到了另一个事务已经提交的insert的数据导致多次查询结果不一致。

* 解决

  设置事务的隔离级别

  * 未提交读   ：避免了脏读
  * 已提交读   ：避免脏读
  * 可重复读   ：避免脏读和不可重复读
  * 串行化      ：避免所有读问题。

#### Spring 进行事务管理API

* 真正管理事务的对象
  * org.springframework.jdbc.datasource.**DataSourceTransactionManager**	使用Spring JDBC或iBatis 进行持久化数据时使用。
  * org.springframework.orm.hibernate3.HibernateTransactionManager		使用Hibernate版本进行持久化数据时使用。

* TransactionDefinition:事务定义信息
  * 事务定义信息：
    * 隔离级别
    * **传播行为**
    * 超时信息
    * 是否只读

* TransactionStatus:事务的状态

  记录事务的状态

* 事务的传播行为

  * 保证在同一个事务中
    * PROPAGATION_REQUIRED		支持当前事务，如果不存在 就新建一个(默认)
    * PROPAGATION_SUPPORTS       支持当前事务，如果不存在，就不使用事务
    * PROPAGATION_MANDATORY	支持当前事务，如果不存在，抛出异常

  * 保证没有在同一事务中
    * PROPAGATION_REQUIRES_NEW	如果有事务存在，挂起当前事务，创建一个新的事务
    * PROPAGATION_NOT_SUPPORTED	以非事务方式运行，如果有事务存在，挂起当前事务
    * PROPAGATION_NEVER 	以非事务方式运行，如果有事务存在，抛出异常
    * PROPAGATION_NESTED	如果当前事务存在，则嵌套事务执行

* 声明式事务管理：基于TransactionProxyFactoryBean方式

  * 配置

  ~~~xml
  <!-- 配置事务管理器 -->
  	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  		<property name="dataSource" ref="dataSource"/>
  	</bean>
  
  <!-- 配置生成代理类 -->
  	<bean id="accountServiceProxy" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
  		<property name="target" ref="accountService"/>
  		<property name="transactionManager" ref="transactionManager"/>
  		<property name="transactionAttributes">
  			<props>
  				<!-- prop的格式：传播行为,隔离级别,只读,-Exception +Exception -->
  				<prop key="transfer">PROPAGATION_REQUIRED</prop>
  			</props>
  		</property>
  	</bean>
  
  ~~~

  * 缺点

    管理麻烦，每个需要进行事务管理的类都要配置一个TransactionProxyFactoryBean,后期维护麻烦。

* 声明式事务：基于AspectJ的AOP配置

  * 配置

    ~~~xml
    <!-- 配置事务管理器 -->
    	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    		<property name="dataSource" ref="dataSource"></property>
    	</bean>
    <!-- 配置一个事务的增强 -->
    	<tx:advice id="txAdvice" transaction-manager="transactionManager">
    		<tx:attributes>
    			<!-- 
    				propagation	:传播行为
    				isolation	:隔离级别
    				read-only	:是否只读
    				timeout		:"-1"
    				rollback-for:发生哪些异常回滚
    				no-rollback-for:发生哪些异常不回滚.
    			 -->
    			<tx:method name="transfer" propagation="REQUIRED"/>
    		</tx:attributes>
    	</tx:advice>
    <!-- 配置AOP的事务 -->
    	<aop:config>
    		<aop:pointcut expression="execution(* com.itheima.spring.demo3.*Service.*(..))" id="pointcut1"/>
    		<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
    	</aop:config>
    
    ~~~

    

* 声明式事务管理：基于注解的方式

  * 配置

    ~~~xml
    	<!-- 配置事务管理器 -->
    	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    		<property name="dataSource" ref="dataSource"></property>
    	</bean>
    <!-- 开启事务管理的注解 -->
    	<tx:annotation-driven transaction-manager="transactionManager"/>
    
    ~~~

  * 在要进行事务管理的类上添加一个注解@Transational