# SpringMVC配置文件

SpringMVC是一个基于DispatcherServlet的MVC框架，每一个请求最先访问的都是DispatcherServlet，DispatcherServlet负责转发每一个Request请求给相应的Handler,Handler处理以后再返回相应的视图（View)和模型（Model)，返回的视图和模型都可以不指定，即可以只返回Model或只返回View或都不返回。

## 1.web.xml

```xml
<!-- Spring MVC配置 -->
<servlet>
    <servlet-name>spring</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 可以自定义servlet.xml配置文件的位置和名称，默认为WEB-INF目录下，名称为[<servlet-name>]-servlet.xml，如spring-servlet.xml-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>spring</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>

<!--设置字符编码方式-->
<!--<filter> 
     <filter-name>setcharacter</filter-name> 
     <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class> 
     <init-param> 
       <param-name>encoding</param-name> 
       <param-value>utf-8</param-value> 
     </init-param> 
 </filter> 
<filter-mapping> 
    <filter-name>setcharacter</filter-name> 
    <url-pattern>/*</url-pattern> 
 </filter-mapping> -->

<!-- Spring配置 -->
<listener>
   <listenerclass>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!-- 指定Spring Bean的配置文件所在目录。默认配置在WEB-INF目录下，默认路径：/WEB-INF/applicationContext.xml，在WEB-INF目录下创建的xml文件的名称必须是applicationContext.xml -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:config/applicationContext.xml</param-value>
</context-param>
```

**关于ContextLoaderListener**

部署applicationContext.xml文件，如果在web.xml中不写任何参数配置信息，默认路径是"/WEB-INF/applicationContext.xml"，在WEB-INF目录下创建的xml文件的名称必须是applicationContext.xml，如果要自定义文件名可以在web.xml里加入contextConfigLocation这个context参数，在<param-value> </param-value>里指定相应的xml文件名，如果有多个xml文件，可以写在一起并一“,”号分隔。

**由此可见，applicationContext.xml的文件位置就可以有两种默认实现：**

1.直接将其放置在/WEB-INF下，只在web.xml中声明一个ContextLoaderListener。(默认路径放置)。

2.将其放置在classpath下，但是此时需要在web.xml中加入<context-param>，用来指定文件放置位置。

## 2.spring-servlet.xml配置

spring-servlet这个名字是因为上面web.xml中<servlet-name>标签配的值为spring（<servlet-name>spring</servlet-name>），再加上“-servlet”后缀而形成的spring-servlet.xml文件名，如果改为springMVC，对应的文件名则为springMVC-servlet.xml。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"     
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"     
        xmlns:context="http://www.springframework.org/schema/context"     
   xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd   
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd   
       http://www.springframework.org/schema/context <a href="http://www.springframework.org/schema/context/spring-context-3.0.xsd">http://www.springframework.org/schema/context/spring-context-3.0.xsd</a>">
    <!-- 启用spring mvc 注解 -->
    <context:annotation-config />
    <!-- 设置使用注解的类所在的jar包 -->
    <context:component-scan base-package="controller"></context:component-scan>
     <!-- 配置js,css等静态文件直接映射到对应的文件夹，不被DispatcherServlet处理 -->
    <mvc:resources location="/WEB-INF/resources/" mapping="/resources/**" />
    <!-- 完成请求和注解POJO的映射 -->
    <bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" />
    <!-- 对转向页面的路径解析。prefix：前缀， suffix：后缀 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" p:prefix="/jsp/" p:suffix=".jsp" />
</beans>
```

DispatcherServlet会利用一些特殊的bean来处理Request请求和生成相应的视图返回。

关于视图的返回，Controller只负责传回来一个值，然后到底返回的是什么视图，是由视图解析器控制的，在jsp中常用的视图解析器是InternalResourceViewResovler，它会要求一个前缀和一个后缀

在上述视图解析器中，如果Controller返回的是blog/index，那么通过视图解析器解析之后的视图就是/jsp/blog/index.jsp。

## 3.ApplicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>   
<beans xmlns="http://www.springframework.org/schema/beans"  
 xmlns:aop="http://www.springframework.org/schema/aop" xmlns:context="http://www.springframework.org/schema/context"  
 xmlns:p="http://www.springframework.org/schema/p" xmlns:tx="http://www.springframework.org/schema/tx"  
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
 xsi:schemaLocation="   
         http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd   
   http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd   
   http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd   
   http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">   
 <context:annotation-config />   
 <context:component-scan base-package="com.mvc" />  <!-- 自动扫描所有注解该路径 -->   
 <context:property-placeholder location="classpath:/hibernate.properties" />   
 <bean id="sessionFactory"  
  class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">   
  <property name="dataSource" ref="dataSource" />   
  <property name="hibernateProperties">   
   <props>   
    <prop key="hibernate.dialect">${dataSource.dialect}</prop>   
    <prop key="hibernate.hbm2ddl.auto">${dataSource.hbm2ddl.auto}</prop>   
    <prop key="hibernate.hbm2ddl.auto">update</prop>   
      </props>   
  </property>   
  <property name="packagesToScan">   
   <list>   
    <value>com.mvc.entity</value><!-- 扫描实体类，也就是平时所说的model -->   
   </list>   
    </property>   
 </bean>   
 <bean id="transactionManager"  
  class="org.springframework.orm.hibernate3.HibernateTransactionManager">   
  <property name="sessionFactory" ref="sessionFactory" />   
  <property name="dataSource" ref="dataSource" />   
 </bean>   
 <bean id="dataSource"  
  class="org.springframework.jdbc.datasource.DriverManagerDataSource">   
  <property name="driverClassName" value="${dataSource.driverClassName}" />   
  <property name="url" value="${dataSource.url}" />   
  <property name="username" value="${dataSource.username}" />   
  <property name="password" value="${dataSource.password}" />   
 </bean> 
 <!-- Dao的实现 -->   
 <bean id="entityDao" class="com.mvc.dao.EntityDaoImpl">     
  <property name="sessionFactory" ref="sessionFactory" />   
 </bean>   
 <tx:annotation-driven transaction-manager="transactionManager" />   
 <tx:annotation-driven mode="aspectj"/>    
    <aop:aspectj-autoproxy/>     
</beans>  
 <!-- 通过AOP配置提供事务增强，让service包下所有Bean的所有方法拥有事务 --> 
 <aop:config proxy-target-class="true"> 
     <aop:pointcut id="serviceMethod" 
         expression=" execution(* com.service..*(..))" /> 
     <aop:advisor pointcut-ref="serviceMethod" advice-ref="txAdvice" /> 
 </aop:config> 
 <tx:advice id="txAdvice" transaction-manager="transactionManager"> 
     <tx:attributes> 
         <tx:method name="*" /> 
     </tx:attributes> 
 </tx:advice> 
<!-- 配置Jdbc模板 --> 
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"> 
  <property name="dataSource" ref="dataSource"></property> 
</bean> 
 <!-- 配置数据源 --> 
 <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" 
     destroy-method="close"> 
     <property name="driverClass"> 
         <value>${jdbc.driverClassName}</value> 
     </property> 
     <property name="jdbcUrl"> 
         <value>${jdbc.url}</value> 
     </property> 
     <property name="user"> 
         <value>${jdbc.username}</value> 
     </property> 
     <property name="password"> 
         <value>${jdbc.password}</value> 
     </property> 
     <!--连接池中保留的最小连接数。 --> 
     <property name="minPoolSize"> 
         <value>5</value> 
     </property> 
     <!--连接池中保留的最大连接数。Default: 15 --> 
     <property name="maxPoolSize"> 
         <value>30</value> 
     </property> 
     <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 --> 
     <property name="initialPoolSize"> 
         <value>10</value> 
     </property> 
     <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 --> 
     <property name="maxIdleTime"> 
         <value>60</value> 
     </property> 
     <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 --> 
     <property name="acquireIncrement"> 
         <value>5</value> 
     </property> 
     <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。但由于预缓存的statements 属于单个connection而不是整个连接池。所以设置这个参数需要考虑到多方面的因素。  
         如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0 --> 
     <property name="maxStatements"> 
         <value>0</value> 
     </property> 
     <!--每60秒检查所有连接池中的空闲连接。Default: 0 --> 
     <property name="idleConnectionTestPeriod"> 
         <value>60</value> 
     </property> 
     <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 --> 
     <property name="acquireRetryAttempts"> 
         <value>30</value> 
     </property> 
     <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效 保留，并在下次调用getConnection()的时候继续尝试获取连接。如果设为true，那么在尝试  
         获取连接失败后该数据源将申明已断开并永久关闭。Default: false --> 
     <property name="breakAfterAcquireFailure"> 
         <value>true</value> 
     </property> 
     <!--因性能消耗大请只在需要的时候使用它。如果设为true那么在每个connection提交的 时候都将校验其有效性。建议使用idleConnectionTestPeriod或automaticTestTable  
         等方法来提升连接测试的性能。Default: false --> 
     <property name="testConnectionOnCheckout"> 
         <value>false</value> 
     </property> 
 </bean> 
```

