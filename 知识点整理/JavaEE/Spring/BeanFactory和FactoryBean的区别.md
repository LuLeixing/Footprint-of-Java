# BeanFactory和FactoryBean的区别

> 菜鸟面试的时候被问到Spring中BeanFactory和FactoryBean的区别。
>
> 一脸懵逼，BeanFactory我虽然知道，但平时用的比较多的是ApplicationContext，而FactoryBean是什么东东？
>
> 之前完全没了解过，有点小尴尬，于是事后展开了一波学习！

## BeanFactory

是一个接口，public interface BeanFactory，提供如下方法：

- Object getBean(String name)
- <T> T getBean(String name, Class<T> requiredType)
- <T> T getBean(Class<T> requiredType)
- Object getBean(String name, Object... args)
- boolean containsBean(String name)
- boolean isSingleton(String name)
- boolean isPrototype(String name)
- boolean isTypeMatch(String name, Class<?> targetType)
- Class<?> getType(String name)
- String[] getAliases(String name)

在Spring中，BeanFactory是IoC容器的核心接口。它的职责包括：实例化，定位，配置应用程序中的对象及建立这些对象间的依赖。

BeanFactory提供的高级配置机制，使得管理任何性质的对象成为可能。

ApplicationContext是BeanFactory的扩展，功能得到了进一步增强，比如更易与Spring AOP集成、消息资源处理（国际化处理）、事件传递及各种不同应用层的context实现（如针对web应用的webApplicationContext)

用的比较多的 `BeanFactory` 的子类是 `ClassPathXmlApplicationContext`，这是   `ApplicationContext`接口的一个子类，`ClassPathXmlApplicationContext`从 xml 的配置文件中获取 bean 并且管理他们。

```java
public static void main(String[] args) throws Exception {
    BeanFactory bf = new ClassPathXmlApplicationContext("student.xml");
    Student studentBean = (Student) bf.getBean("studentBean");

    studentBean.print();
}
```

XML配置如下：

```java
<bean id="studentBean" class="advanced.Student">
    <property name="name" value="Tom"/>
    <property name="age" value="18"/>
</bean>
```

## FactoryBean

Spring中为我们提供了两种类型的bean，一种就是普通的bean，我们通过getBean(id)方法获得是该bean的实例类型，另一种bean是FactoryBean，也就是FactoryBean，也就是工程bean，我们通过getBean(id)获得是该工厂锁产生的Bean的实例，而不是该FactoryBean的实例。

**FactoryBean是一个Bean，实现了FactoryBean接口的类有能力改变bean，FactoryBean希望你实现了它之后返回一些内容，Spring 会按照这些内容去注册 bean。**
`public interface FactoryBean<T>`，提供如下方法：

- `T getObject()`
- `Class<?> getObjectType()`
- `boolean isSingleton()`

**通常情况下，bean 无须自己实现工厂模式，Spring 容器担任工厂角色；但少数情况下，容器中的 bean 本身就是工厂，作用是产生其他 bean 实例。由工厂 bean 产生的其他 bean 实例，不再由 Spring 容器产生，因此与普通 bean 的配置不同，不再需要提供 class 元素。**

FactoryBean是一个Bean，实现了FactoryBean接口的类有能力改变bean，FactoryBean希望你实现了它之后返回一些内容，Spring会按照这些内容去注册bean，下面展示了FactoryBean提供的接口方法，需要注意的是，在Spring中为我们实现了大量的FactoryBean，所以可以看出FactoryBean是非常重要的。

可以看到，FactoryBean非常简单，三个方法的意义非常明确，getObject希望你返回需要注册到Spring容器中去的bean实体，getObjectType希望你返回你注册的这个Object的具体类型，isSingleton方法希望你返回这个bean是不是单例的，如果是，那么Spring容器全局将只保持一个该实例对象，否则每次getBean都将获取到一个新的该实例对象。

FactoryBean的功能貌似更像是一种代理，有一种场景是，我们使用一个通用的类来在xml文件中注册bean，我们希望通过该通用bean产生一个我们希望的bean，而这个需求FactoryBean就可以办到，你只需要拦截你需要代理的bean，然后转换成你希望的bean再注册。一个应用场景就是Rpc服务器端的bean注册，以及Rpc客户端的服务调用，都可以通过一个第三方bean来产生我们真正需要的bean。