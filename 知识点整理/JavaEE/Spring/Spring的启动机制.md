# Spring的启动机制

Spring的启动过程其实就是其IoC容器的启动过程。

对于web程序，IoC容器启动过程即是建立上下文的过程。

因此，结合web.xml的配置来进行说明：

**首先，Servlet容器启动，为应用创建全局的上下文环境，即"ServletContext"。**

```xml
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
    </context-param>
<listener>      
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>

```

这段是加载spring配置文件，初始化上下文，ContextLoaderListener是一个实现了ServletContextListener接口的监听器，**在启动项目时会触发contextInitialized方法**（该方法主要完成ApplicationContext对象的创建），**在关闭项目时会触发contextDestroyed方法**（该方法会执行ApplicationContext清理操作）。

1.启动项目时触发contextInitialized方法，该方法就做一件事：通过父类contextLoader的initWebApplicationContext方法创建Spring上下文对象。

2.initWebApplicationContext方法做了三件事：创建 WebApplicationContext；加载对应的Spring文件创建里面的Bean实例；将WebApplicationContext放入 ServletContext（就是Java Web的全局变量）中。

3.createWebApplicationContext创建上下文对象，支持用户自定义的 上下文对象，但必须继承自ConfigurableWebApplicationContext，而Spring MVC默认使用ConfigurableWebApplicationContext作为ApplicationContext（它仅仅是一个接口）的实现。

> ConfigurableWebApplicationContext继承WebApplicationContext，而WebApplicationContext继承ApplicationContext。

4.configureAndRefreshWebApplicationContext方法用 于封装ApplicationContext数据并且初始化所有相关Bean对象。它会从web.xml中读取名为 contextConfigLocation的配置，这就是spring xml数据源设置，然后放到ApplicationContext中，**最后调用传说中的refresh方法执行所有Java对象的创建**。

5.完成ApplicationContext创建之后就是将其放入ServletContext中，注意它存储的key值常量。

```xml
<servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

这段为启动初始化DispatcherServlet ，web.xml中设置了Servlet的**load-on-startup****：**表示启动容器时初始化该Servlet。

**url-pattern****：**表示哪些请求交给Spring Web MVC处理， “/” 是用来定义默认servlet映射的。也可以如“*.html”表示拦截所有以html为扩展名的请求。

DispatcherServlet默认使用WebApplicationContext**（ContextLoaderListener初始化产生）**作为上下文，Spring默认配置文件为“/WEB-INF/[servlet名字]-servlet.xml”（该名字可以自定义，在<param-value>）。

DispatcherServlet也可以配置自己的初始化参数，覆盖默认配置，因此我们可以通过添加初始化参数。

**DispatcherServlet初始化顺序：**

1.HttpServletBean继承HttpServlet，因此在Web容器启动时将调用它的init方法，该初始化方法的主要作用：将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用；提供给子类初始化扩展点，initServletBean()，该方法由FrameworkServlet覆盖。

2.FrameworkServlet继承HttpServletBean，通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情：初始化web上下文；提供给子类初始化扩展点。

3.DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置。

**整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：**

1、初始化Spring Web MVC使用的Web上下文，并且指定父容器为WebApplicationContext（ContextLoaderListener加载了的根上下文）；

2、初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。



## 总结：

### 概括一下Spring的启动过程

1.首先，对于一个web应用，其部署在web容器中，web容器提供其一个全局的上下文环境，这个上下文就是ServletContext，其为后面的spring IoC容器提供宿主环境；

2.其次，在web.xml中会提供有contextLoaderListener。**在web容器启动时，会触发容器初始化事件，此时 contextLoaderListener会监听到这个事件**，其contextInitialized方法会被调用，在这个方法中，spring会初始化一个启动上下文，这个上下文被称为根上下文，即WebApplicationContext，这是一个接口类，确切的说，其实际的实现类是 XmlWebApplicationContext。这个就是spring的IoC容器，其对应的Bean定义的配置由web.xml中的 context-param标签指定。在这个IoC容器初始化完毕后，spring以WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE为属性Key，将其存储到ServletContext。

3.再次，contextLoaderListener监听器初始化完毕后，开始初始化web.xml中配置的Servlet，这里是DispatcherServlet，这个servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个servlet请 求。DispatcherServlet上下文在初始化的时候会建立自己的IoC上下文，用以持有spring mvc相关的bean。在建立DispatcherServlet自己的IoC上下文时，会利用WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE 先从ServletContext中获取之前的根上下文(即WebApplicationContext)作为自己上下文的parent上下文。有了这个 parent上下文之后，再初始化自己持有的上下文。这个DispatcherServlet初始化自己上下文的工作在其initStrategies方 法中可以看到，大概的工作就是初始化处理器映射、视图解析等。这个servlet自己持有的上下文默认实现类也是XmlWebApplicationContext。初始化完毕后，spring以与servlet的名字相关(此处不是简单的以servlet名为 Key，而是通过一些转换，具体可自行查看源码)的属性为属性Key，也将其存到ServletContext中，以便后续使用。这样每个servlet 就持有自己的上下文，即拥有自己独立的bean空间，同时各个servlet共享相同的bean，即根上下文(第2步中初始化的上下文)定义的那些 bean。

