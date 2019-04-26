## 理解Spring中的BeanFactory和FactoryBean的区别

1.实现BeanFactory接口的类表明此类是一个工厂，作用是配置、新建和管理各种Bean。

2.实现FactoryBean的类表明此类也是一个Bean，类型为工厂Bean(Spring中共有两种bean，一种为普通的bean，另一种则为工厂bean)。顾名思义，它也是用来管理Bean的，而它本身由spring管理。

一个Bean想要实现 `FactoryBean` ，必须实现以下三个接口：

```java
1. Object getObject():返回由FactoryBean创建的Bean的实例
2. boolean isSingleton():确定由FactoryBean创建的Bean的作用域是singleton还是prototype；
3. getObjectType():返回FactoryBean创建的Bean的类型。12345
```

## 注意：

1.如果将一个实现了FactoryBean的类成功配置到了Spring的上下文中，那么通过该类对象的名称从Spring的ApplicationContext或者BeanFactory获取bean时，获取到的时FactoryBean创建的Bean的实例，而不是FactoryBean的实例本身，如果想通过Spring获取FactoryBean的本事的实例，则需要在名称前加"&"符号：

```java
out.println(applicationContext.getBean("&appleFactoryBean")); //可获取到FactoryBean实例本身
```

2.Factory管理的bean实际上也是由Spring进行配置、实例化、管理，因此由FactoryBean管理的bean不能再次配置到Spring的配置上下文中（xml，Java类配置，注解均不可以）。

## 举例：

#### 1.AppleFactoryBean

```java
@Component
public class AppleFactoryBean implements FactoryBean{

    public Object getObject() throws Exception {
        return new AppleBean();
    }

    public Class<?> getObjectType() {
        return AppleBean.class;
    }

    public boolean isSingleton() {
        return false;
    }
};  //定义的时候需要实现三个方法，如上所示。
```

#### 2.因为AppleBean实例通过AppleFactoryBean进行返回，不需要再在Spring上下文中进行配置了。

```java
//@Component  这里不可以加注解 ！！！！！！
public class AppleBean{

};
```

#### 3.启动类

```java
public class Start {
    public static void main(String[] args){
        ApplicationContext applicationContext = new                AnnotationConfigApplicationContext(Configurations.class);
                 out.println(applicationContext.getBean("appleFactoryBean"));//得到的是apple
                 out.println(applicationContext.getBean("&appleFactoryBean"));//得到的是apple工厂
    }
}
```



