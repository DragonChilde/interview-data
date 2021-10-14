# Spring循环依赖

- 你解释下spring中的三级缓存？
- 三级缓存分别是什么？三个Map有什么异同？
- 什么是循环依赖？请你谈谈？看过spring源码吗？
- 如何检测是否存在循环依赖？实际开发中见过循环依赖的异常吗？
- 多例的情况下，循环依赖问题为什么无法解决？

------

什么是循环依赖？

多个bean之间相互依赖，形成了一个闭环。比如：A依赖于B、B依赖于C、C依赖于A。

通常来说，如果问Spring容器内部如何解决循环依赖，一定是指默认的单例Bean中，属性互相引用的场景。

![](http://120.77.237.175:9080/photos/eight/spring/02.png)

两种注入方式对循环依赖的影响

循环依赖官网说明

> Circular dependencies
>
> If you use predominantly constructor injection, it is possible to create an unresolvable circular dependency scenario.
>
> For example: Class A requires an instance of class B through constructor injection, and class B requires an instance of class A through constructor injection. If you configure beans for classes A and B to be injected into each other, the Spring IoC container detects this circular reference at runtime, and throws a `BeanCurrentlyInCreationException`.
>
> One possible solution is to edit the source code of some classes to be configured by setters rather than constructors. Alternatively, avoid constructor injection and use setter injection only. In other words, although it is not recommended, you can configure circular dependencies with setter injection.
>
> Unlike the typical case (with no circular dependencies), a circular dependency between bean A and bean B forces one of the beans to be injected into the other prior to being fully initialized itself (a classic chicken-and-egg scenario).

结论

我们AB循环依赖问题只要A的注入方式是setter且singleton ，就不会有循环依赖问题。

------

## spring循环依赖纯java代码验证案例

Spring容器循环依赖报错演示``BeanCurrentlylnCreationException`

循环依赖现象在``spring`容器中注入依赖的对象，有2种情况

- 构造器方式注入依赖（不可行）

  ```java
  @Component
  public class ServiceA {
      private ServiceB serviceB;
  
      public ServiceA(ServiceB serviceB){
          this.serviceB = serviceB;
      }
  }
  ```

  ```java
  @Component
  public class ServiceB {
  
      private ServiceA serviceA;
  
      public ServiceB(ServiceA serviceA){
          this.serviceA = serviceA;
      }
  }
  ```

  ```java
  public class ClientConstructor {
  
      public static void main(String[] args){
          new ServiceA(new ServiceB(new ServiceA()));//这会抛出编译异常
      }
  }
  ```

- 以set方式注入依赖（可行）

  ```java
  @Component
  public class ServiceAA {
  
      private ServiceAA serviceAA;
  
      public void setServiceAA(ServiceAA serviceAA){
          this.serviceAA = serviceAA;
          System.out.println("B里面设置了A");
      }
  }
  ```

  ```java
  @Component
  public class ServiceBB {
  
    private ServiceAA serviceAA;
  
    public void setServiceAA(ServiceAA serviceAA) {
      this.serviceAA = serviceAA;
      System.out.println("B里面设置了A");
    }
  }
  ```

  ```java
  public class ClientSet {
  
    public static void main(String[] args) {
      //创建serviceAA
      ServiceAA a = new ServiceAA();
      //创建serviceBB
      ServiceBB b = new ServiceBB();
      //将serviceA入到serviceB中
      b.setServiceAA(a);
      //将serviceB法入到serviceA中
      a.setServiceBB(b);
    }
  }
  ```

  上面代码输出结果

  ```
  B里面设置了A
  A里面设置了B
  ```

------

## spring循环依赖bug演示

Bean类

```java
public class A {
  private B b;

  public B getB() {
    return b;
  }

  public void setB(B b) {
    this.b = b;
    System.out.println("A call setB.");
  }
}

```

```java
public class B {
  private A a;

  public A getA() {
    return a;
  }

  public void setA(A a) {
    this.a = a;
    System.out.println("B call setA.");
  }
}
```

执行类

```java
public class ClientSpringContainer {

  public static void main(String[] args) {
    //
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    A a = context.getBean("a", A.class);
    B b = context.getBean("b", B.class);
  }
}
```

默认的单例(Singleton)的场景是**支持**循环依赖的，不报错

`beans.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="a" class="com.example.demo.bean.A">
        <property name="b" ref="b"></property>
    </bean>

    <bean id="b" class="com.example.demo.bean.B">
        <property name="a" ref="a"></property>
    </bean>
</beans>
```

输出结果如下

```
15:10:07.126 [main] DEBUG org.springframework.context.support.ClassPathXmlApplicationContext - Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@5fdef03a
15:10:07.348 [main] DEBUG org.springframework.beans.factory.xml.XmlBeanDefinitionReader - Loaded 2 bean definitions from class path resource [beans.xml]
15:10:07.389 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'a'
15:10:07.406 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'b'
B call setA.
A call setB.
```

> > **scope = “singleton”，默认的单例(Singleton)的场景是支持循环依赖的，不报错**

每个 bean 的 `scope` 实行默认不写就是 `singleton`

```xml
<bean id="a" class="com.example.demo.bean.A" scope="singleton">
	<property name="b" ref="b"></property>
</bean>

<bean id="b" class="com.example.demo.bean.B" scope="singleton">
	<property name="a" ref="a"></property>
</bean>
```

将 bean 的生命周期改为 `prototype`

```xml
    <bean id="a" class="com.example.demo.bean.A" scope="prototype">
        <property name="b" ref="b"></property>
    </bean>

    <bean id="b" class="com.example.demo.bean.B" scope="prototype">
        <property name="a" ref="a"></property>
    </bean>
```

抛出了以下异常

```
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'a' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'b' while setting bean property 'b'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'b' defined in class path resource [beans.xml]: Cannot resolve reference to bean 'a' while setting bean property 'a'; nested exception is org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference
```

------

## 循环依赖的解决办法

> **重要结论：Spring 内部通过 3 级缓存来解决循环依赖**

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

	/** Maximum number of suppressed exceptions to preserve. */
	private static final int SUPPRESSED_EXCEPTIONS_LIMIT = 100;


	/** Cache of singleton objects: bean name to bean instance. */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);

	...
}
```

- 第一级缓存：`Map<String, Object> singletonObjects`，我愿称之为成品单例池，常说的 `Spring `容器就是指它，我们获取单例 `bean `就是在这里面获取的，存放已经经历了完整生命周期的Bean对象
- 第二级缓存：`Map<String, Object> earlySingletonObjects`，存放早期暴露出来的`Bean`对象，`Bean`的生命周期未结束（属性还未填充完整，可以认为是半成品的 bean）
- 第三级缓存：`Map<String, ObiectFactory<?>> singletonFactories`，存放可以生成Bean的工厂，用于生产（创建）对象

------

## spring循环依赖debug前置知识

### 实例化和初始化的区别

1. 实例化：堆内存中申请一块内存空间，如同租赁好房子，自己的家当还未搬来。

   ![](http://120.77.237.175:9080/photos/eight/spring/04.jpg)

2. 初始化：完成属性的填充,完成属性的各种赋值，如同装修，家具，家电进场。

   ![](http://120.77.237.175:9080/photos/eight/spring/05.jpg)

------

### 3个Map & 4个方法

**三级缓存**

![](http://120.77.237.175:9080/photos/eight/spring/03.png)

- 第一级缓存：存放的是已经初始化好了的`Bean`，`bean`名称与`bean`实例相对应，即所谓的单例池。表示已经经历了完整生命周期的`Bean`对象
- 第二级缓存：存放的是实例化了，但是未初始化的`Bean`，`bean`名称与`bean`实例相对应。表示`Bean`的生命周期还没走完（`Bean`的属性还未填充）就把这个`Bean`存入该缓存中。也就是实例化但未初始化的`bean`放入该缓存里
- 第三级缓存：表示存放生成`bean`的工厂，存放的是`FactoryBean`，`bean`名称与`bean`工厂对应。假如A类实现了FactoryBean，那么依赖注入的时候不是A类，而是A类产生的`Bean`

**四大方法**

1. `getSingleton()`：从容器里面获得单例的`bean`，没有的话则会创建 `bean`
2. `doCreateBean()`：执行创建 `bean` 的操作（在 `Spring `中以 `do `开头的方法都是干实事的方法）
3. `populateBean()`：创建完 `bean `之后，对 `bean `的属性进行填充
4. `addSingleton()`：`bean `初始化完成之后，添加到单例容器池中，下次执行 `getSingleton()` 方法时就能获取到

> 注：关于三级缓存 `Map<String, ObjectFactory<?>> singletonFactories`的说明，`singletonFactories `的 `value `为 `ObjectFactory `接口实现类的实例。`ObjectFactory `为函数式接口，在该接口中定义了一个` getObject()` 方法用于获取 `bean`，这也正是工厂思想的体现（工厂设计模式）
>
> ------
>
> ```java
> @FunctionalInterface
> public interface ObjectFactory<T> {
> 
> 	/**
> 	 * Return an instance (possibly shared or independent)
> 	 * of the object managed by this factory.
> 	 * @return the resulting instance
> 	 * @throws BeansException in case of creation errors
> 	 */
> 	T getObject() throws BeansException;
> 
> }
> ```

------

### 对象在三级缓存中的迁移

1. A创建过程中需要B，于是A将自己放到三级缓存里面，去实例化B
2. B实例化的时候发现需要A，于是B先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了A，然后把三级缓存里面的这个A放到二级缓存里面，并删除三级缓存里面的A
3. B顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中状态），然后回来接着创建A，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，并将A自己放到一级缓存里面。

------

## 详细 Debug 流程

### beanA 的实例化

> **技巧：如何阅读框架源码？答：打断点 + 看日志**

在 `ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");` 代码处打上断点，逐步执行（Step Over），发现执行 `new ClassPathXmlApplicationContext("applicationContext.xml") `操作时，`beanA `和 `beanB `都已经被创建好了，因此我们需要进入` new ClassPathXmlApplicationContext("applicationContext.xml")` 中

![](http://120.77.237.175:9080/photos/eight/spring/06.jpg)

> > **进入 `new ClassPathXmlApplicationContext("applicationContext.xml")` 中**

点击 Step Into，首先进入了静态代码块中，不管我们的事，使用 Step Out 退出此方法

```java
	static {
		// Eagerly load the ContextClosedEvent class to avoid weird classloader issues
		// on application shutdown in WebLogic 8.1. (Reported by Dustin Woods.)
		ContextClosedEvent.class.getName();
	}
```

再次 Step Into，进入 `ClassPathXmlApplicationContext` 类的构造函数，该构造函数使用 `this `调用了另一个重载构造函数

```java
	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}
```

继续 Step Into，进入重载构造函数后单步 Step Over，发现执行完 `refresh()` 方法后输出如下日志，于是我们将断点打在 `refresh()` 那一行

![](http://120.77.237.175:9080/photos/eight/spring/07.jpg)

> **进入 `refresh()` 方法**

Step Into 进入` refresh()` 方法，发现执行完 `finishBeanFactoryInitialization(beanFactory) `方法后输出日志，于是我们将断点打在 `finishBeanFactoryInitialization(beanFactory) `那一行

从注释也可以看出本方法完成了非懒加载单例 bean的初始化`（Instantiate all remaining (non-lazy-init) singletons.）`

![](http://120.77.237.175:9080/photos/eight/spring/08.jpg)

> **进入 `finishBeanFactoryInitialization(beanFactory)` 方法**

Step Into 进入 `finishBeanFactoryInitialization(beanFactory) `方法，发现执行完 `beanFactory.preInstantiateSingletons()` 方法后输出日志，于是我们将断点打在 `beanFactory.preInstantiateSingletons()` 那一行

从注释也可以看出本方法完成了非懒加载单例 `bean`的初始化`（Instantiate all remaining (non-lazy-init) singletons.）`

![](http://120.77.237.175:9080/photos/eight/spring/09.jpg)

> **进入 `beanFactory.preInstantiateSingletons()` 方法**

Step Into 进入 `beanFactory.preInstantiateSingletons()` 方法，发现执行完 `getBean(beanName)` 方法后输出日志，于是我们将断点打在 `getBean(beanName)` 那一行

![](http://120.77.237.175:9080/photos/eight/spring/10.jpg)

> **进入 `getBean(beanName)` 方法**

`getBean(beanName)` 调用了 `doGetBean(name, null, null, false)` 方法，也就是前面说过的：在 `Spring `里面，以`do `开头的方法都是干实事的方法

```java
	@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
```

> > **进入 `doGetBean(name, null, null, false)` 方法**

我们可以给 `bean `配置别名，这里的 `transformedBeanName(name)` 方法就是将用户别名转换为 bean 的真实名称

![](http://120.77.237.175:9080/photos/eight/spring/11.jpg)

> **进入 `getSingleton(beanName)` 方法**

![](http://120.77.237.175:9080/photos/eight/spring/12.jpg)

调用了其重载的方法，`allowEarlyReference == true` 表示可以从三级缓存 `earlySingletonObjects `中获取 `bean`，`allowEarlyReference == false` 表示不可以从三级缓存 `earlySingletonObjects `中获取 `bean`，

![](http://120.77.237.175:9080/photos/eight/spring/13.jpg)

`getSingleton(beanName, true)` 方法尝试从一级缓存 `singletonObjects `中获取 `beanA`，`beanA `现在还没有开始造呢（`isSingletonCurrentlyInCreation(beanName)` 返回 `false`），获取不到返回 `null`

![](http://120.77.237.175:9080/photos/eight/spring/14.jpg)

> **回到 `doGetBean(name, null, null, false)` 方法中**

`getSingleton(beanName)` 方法返回 `null`

![](http://120.77.237.175:9080/photos/eight/spring/15.jpg)

我们所说的 `bean `对于 `Spring `来说就是一个个的 `RootBeanDefinition` 实例

![](http://120.77.237.175:9080/photos/eight/spring/16.jpg)

这个 `dependsOn` 变量对应于 bean 的 `depends-on=""` 属性，我们没有配置过，因此为 `null`

![](http://120.77.237.175:9080/photos/eight/spring/17.jpg)

转了一圈发现并没有 `beanA`，终于要开始准备创建 `beanA `啦

![](http://120.77.237.175:9080/photos/eight/spring/18.jpg)

> **进入 `getSingleton(beanName, () -> {... }` 方法**

首先尝试从一级缓存 `singletonObjects `获取 `beanA`，那肯定是获取不到的啦，因此 s`ingletonObject == null`，那么就需要创建 `beanA`，此时日志会输出：【Creating shared instance of singleton bean ‘a’】

![](http://120.77.237.175:9080/photos/eight/spring/19.jpg)

执行完 `singletonObject = singletonFactory.getObject();` 时，会输出【—A created success】，这说明执行 `singletonFactory.getObject()` 方法时将会实例化 `beanA`，并且根据代码变量名可得知单例工厂创建的，这个单例工厂就是我们传入的 `Lambda `表达式

![](http://120.77.237.175:9080/photos/eight/spring/20.jpg)

> **进入 `createBean(beanName, mbd, args)` 方法**

我们 Step Into 进入 `createBean(beanName, mbd, args)` 方法中，`mbdToUse` 将用于创建 `beanA`

![](http://120.77.237.175:9080/photos/eight/spring/21.jpg)

来了，终于要执行 `doCreateBean(beanName, mbdToUse, args)` 实例化 beanA 啦

![](http://120.77.237.175:9080/photos/eight/spring/22.jpg)

> **进入 `doCreateBean(beanName, mbdToUse, args)` 方法**

Step Into 进入 `doCreateBean(beanName, mbdToUse, args)` 方法，在 `factoryBeanInstanceCache `中并不存在 `beanA `对应的 `Wrapper `缓存，`instanceWrapper == null`，因此我们要去创建 `beanA `对应的 `instanceWrapper`，`Wrapper `有包裹之意思，`instanceWrapper `翻译过来为实例包裹器的意思，形象理解为：`beanA `实例化需要经过 `instanceWrapper `之手，`beanA `实例被 `instanceWrapper `包裹在其中

![](http://120.77.237.175:9080/photos/eight/spring/23.jpg)

> **进入 `createBeanInstance(beanName, mbd, args)` 方法**

这一看就是反射的操作啊

![](http://120.77.237.175:9080/photos/eight/spring/24.jpg)

这里有个 `resolved `变量，写着注释：Shortcut when re-creating the same bean…，我个人理解是 `resolved `标志该 `bean `是否已经被实例化了，如果已经被实例化了，那么` resolved == true`，这样就不用重复创建同一个 `bean `了

![](http://120.77.237.175:9080/photos/eight/spring/25.jpg)

Candidate constructors for autowiring? 难道是构造器自动注入？在 `return `的时候调用 `instantiateBean(beanName, mbd)` 方法实例化 `beanA`，并将其返回

![](http://120.77.237.175:9080/photos/eight/spring/26.jpg)

> **进入 `instantiateBean(beanName, mbd)` 方法**

`getInstantiationStrategy().instantiate(mbd, beanName, this)` 方法完成了 beanA 的实例化

![](http://120.77.237.175:9080/photos/eight/spring/27.jpg)

> **进入 `getInstantiationStrategy().instantiate(mbd, beanName, this)` 方法**

首先获取已经解析好的构造器 `bd.resolvedConstructorOrFactoryMethod`，这是第一次创建，当然还没有啦，因此 `constructorToUse == null`。然后获取 A 的类型，如果发现是接口则直接抛异常。最后获取 A 的公开构造器，并将其赋值给 `bd.resolvedConstructorOrFactoryMethod`,获取构造器的目的当然是为了实例化 `beanA `啦

![](http://120.77.237.175:9080/photos/eight/spring/28.jpg)

> **进入 `BeanUtils.instantiateClass(constructorToUse)` 方法**

通过构造器创建 `beanA `实例，Step Over 后会输出：【—A created success】，并且会回到 `getInstantiationStrategy().instantiate(mbd, beanName, this)` 方法中

![](http://120.77.237.175:9080/photos/eight/spring/29.jpg)

> **回到 `getInstantiationStrategy().instantiate(mbd, beanName, this)` 方法中**

在 `BeanUtils.instantiateClass(constructorToUse)` 方法中创建好了 beanA 实例，不过还没有进行初始化，可以看到属性 `b = null`，Step Over 后会回到 `instantiateBean(beanName, mbd)` 方法中

![](http://120.77.237.175:9080/photos/eight/spring/30.jpg)

> **回到 `instantiateBean(beanName, mbd)` 方法中**

得到刚才创建的 `beanA `实例，但其属性并未被初始化

![](http://120.77.237.175:9080/photos/eight/spring/31.jpg)

将实例化的 `beanA `装进 `BeanWrapper `中并返回 `bw`

![](http://120.77.237.175:9080/photos/eight/spring/32.jpg)

> **回到 `createBeanInstance(beanName, mbd, args)` 方法中**

![](http://120.77.237.175:9080/photos/eight/spring/33.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

在 `doCreateBean(beanName, mbdToUse, args)` 方法获得 `BeanWrapper instanceWrapper`，用于封装 `beanA `实例

![](http://120.77.237.175:9080/photos/eight/spring/34.jpg)

获取并记录 A 的全类名

![](http://120.77.237.175:9080/photos/eight/spring/35.jpg)

执行 `BeanPostProcessor`

![](http://120.77.237.175:9080/photos/eight/spring/36.jpg)

如果该 `bean `是单例 `bean（mbd.isSingleton()）`，并且允许循环依赖`（this.allowCircularReferences）`，并且当前 `bean `正在创建过程中`（isSingletonCurrentlyInCreation(beanName)）`，那么就就允许提前暴露该单例 `bean``（earlySingletonExposure = true）`，则会执行` addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean))` 方法将该 `bean `放到三级缓存 `singletonFactories `中

![](http://120.77.237.175:9080/photos/eight/spring/37.jpg)

> **进入 `addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean))` 方法**

首先去一级缓存 `singletonObjects `中找一下有没有 `beanA`，肯定没有啦~然后将 `beanA `添加到三级缓存 `singletonFactories `中，并将 `beanA `从二级缓存 `earlySingletonObjects `中移除，最后将 `beanName` 添加至 `registeredSingletons `中，表示该 `bean `实例已经被注册

![](http://120.77.237.175:9080/photos/eight/spring/38.jpg)

------

### beanA 的属性填充

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

接着回到 `doCreateBean(beanName, mbdToUse, args)` 方法中，需要执行 `populateBean(beanName, mbd, instanceWrapper)` 方法对 beanA 中的属性进行填充

![](http://120.77.237.175:9080/photos/eight/spring/39.jpg)

> **进入 `populateBean(beanName, mbd, instanceWrapper)` 方法**

获取 `beanA` 的属性列表

![](http://120.77.237.175:9080/photos/eight/spring/40.jpg)

执行 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法完成 beanA 属性的填充

![](http://120.77.237.175:9080/photos/eight/spring/41.jpg)

> **进入 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法**

获取到 `beanA `的属性列表，发现有个属性为 `b`

![](http://120.77.237.175:9080/photos/eight/spring/42.jpg)

遍历每一个属性，并对每一个属性进行注入，`valueResolver.resolveValueIfNecessary(pv, originalValue)` 的作用：Given a PropertyValue, return a value, resolving any references to other beans in the factory if necessary.

![](http://120.77.237.175:9080/photos/eight/spring/43.jpg)

> **进入 `valueResolver.resolveValueIfNecessary(pv, originalValue)` 方法**

通过 `resolveReference(argName, ref)` 解决依赖注入的问题

![](http://120.77.237.175:9080/photos/eight/spring/44.jpg)

> **进入 `resolveReference(argName, ref)` 方法**

先获得属性 `b` 的名称，再通过 `this.beanFactory.getBean(resolvedName)` 方法获取 `beanB `的实例

![](http://120.77.237.175:9080/photos/eight/spring/45.jpg)

------

### beanB 的实例化

> **进入 `this.beanFactory.getBean(resolvedName)` 方法**

哦，这熟悉的 `doGetBean(name, null, null, false)` 方法，这就开始递归了呀

![](http://120.77.237.175:9080/photos/eight/spring/46.jpg)

> **再次执行 `doGetBean(name, null, null, false)` 方法**

`beanB `还没有实例化，因此 `getSingleton(beanName)` 方法返回 `null`

![](http://120.77.237.175:9080/photos/eight/spring/47.jpg)

呐，又来到了这个熟悉的地方，先尝试获取 `beanB `实例，获取不到就执行 `createBean()` 的操作

![](http://120.77.237.175:9080/photos/eight/spring/48.jpg)

> **进入 `getSingleton(beanName, () -> {... }` 方法**

首先尝试从一级缓存 `singletonObjects` 中获取 beanB，那肯定是获取不到的呀

![](http://120.77.237.175:9080/photos/eight/spring/49.jpg)

然后就调用 `singletonFactory.getObject()` 创建 beanB

![](http://120.77.237.175:9080/photos/eight/spring/50.jpg)

> **进入 `createBean(beanName, mbd, args)` 方法**

获取到 `beanB `的类型为 `com.example.demo.bean.B`

![](http://120.77.237.175:9080/photos/eight/spring/51.jpg)

之前创建 `beanA `的时候没有看到，现在看到挺有趣的：Give BeanPostProcessors a chance to return a proxy instead of the target bean instance. 也就是说我们可以通过 `BeanPostProcessors `返回 `bean `的代理，而非 `bean `本身。然后喜闻乐见，又来到了` doCreateBean(beanName, mbdToUse, args)` 环节

![](http://120.77.237.175:9080/photos/eight/spring/52.jpg)

> **进入 `doCreateBean(beanName, mbdToUse, args)` 方法**

老样子，创建 beanB 对应的 `BeanWrapper instanceWrapper`

![](http://120.77.237.175:9080/photos/eight/spring/53.jpg)

> **进入 `createBeanInstance(beanName, mbd, args)` 方法**

调用 `instantiateBean(beanName, mbd)` 创建 `beanWrapper`

![](http://120.77.237.175:9080/photos/eight/spring/54.jpg)

> **进入 `instantiateBean(beanName, mbd)` 方法**

调用 `getInstantiationStrategy().instantiate(mbd, beanName, this)` 创建 `beanWrapper`

![](http://120.77.237.175:9080/photos/eight/spring/55.jpg)

> **进入 `getInstantiationStrategy().instantiate(mbd, beanName, this)` 方法**

获取 `com.example.demo.bean.B` 的构造器，并将构造器信息记录在 `bd.resolvedConstructorOrFactoryMethod` 字段中

![](http://120.77.237.175:9080/photos/eight/spring/56.jpg)

调用 `BeanUtils.instantiateClass(constructorToUse)` 方法创建 beanB 实例

![](http://120.77.237.175:9080/photos/eight/spring/57.jpg)

> **进入 `BeanUtils.instantiateClass(constructorToUse)` 方法**

通过调用 `B` 类的构造器创建 beanB 实例，此时控制台会输出：【—B created success】

![](http://120.77.237.175:9080/photos/eight/spring/58.jpg)

> **回到 `instantiateBean(beanName, mbd)` 方法中**

在 `instantiateBean(beanName, mbd)` 方法中得到创建好的 `beanB `实例，并将其丢进 `beanWrapper` 中，封装为 `BeanWrapper bw` 对象

![](http://120.77.237.175:9080/photos/eight/spring/59.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

`createBeanInstance(beanName, mbd, args)` 方法将返回包装着 `beanB `的 `beanWrapper`

![](http://120.77.237.175:9080/photos/eight/spring/60.jpg)

执行 `BeanPostProcessor` 的处理过程

![](http://120.77.237.175:9080/photos/eight/spring/61.jpg)

`beanB `由于满足单例并且正在被创建，因此 `beanB `可以被提前暴露出去（在属性还未初始化的时候可以提前暴露出去），于是执行 `addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean))` 方法将其添加至三级缓存 `singletonFactory `中

![](http://120.77.237.175:9080/photos/eight/spring/62.jpg)

> **进入 `addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean))` 方法**

将 beanB 实例添加至三级缓存 `singletonFactory` 中，从二级缓存 `earlySingletonObjects` 中移除，并注册其 beanName

![](http://120.77.237.175:9080/photos/eight/spring/63.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

执行 `populateBean(beanName,mbd,instancewrapper)` 方法填充 beanB 的属性

![](http://120.77.237.175:9080/photos/eight/spring/64.jpg)

------

### beanB 的属性填充

> **进入 `populateBean(beanName, mbd, instanceWrapper)` 方法**

执行 `mbd.getPropertyValues()` 方法获取 beanB 的属性列表

![](http://120.77.237.175:9080/photos/eight/spring/65.jpg)

执行 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法完成 beanB 属性的填充

![](http://120.77.237.175:9080/photos/eight/spring/66.jpg)

> **进入 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法**

执行 `mpvs.getPropertyValuelist()` 方法获取 beanB 的属性列表

![](http://120.77.237.175:9080/photos/eight/spring/67.jpg)

遍历每一个属性，并对每一个属性进行注入，`valueResolver.resolveValueIfNecessary(pv, originalValue) `的作用：Given a PropertyValue, return a value, resolving any references to other beans in the factory if necessary.

![](http://120.77.237.175:9080/photos/eight/spring/68.jpg)

> **进入 `valueResolver.resolveValueIfNecessary(pv, originalValue)` 方法**

执行 `resolveReference(argName, ref)` 方法为 `beanB `注入名为 `a` 属性

![](http://120.77.237.175:9080/photos/eight/spring/69.jpg)

> **进入 `resolveReference(argName, ref)` 方法**

执行 `this.beanFactory.getBean(resolvedName)` 方法获取 beanA 实例，其实就是执行 `doGetBean(name, null, null, false)` 方法

![](http://120.77.237.175:9080/photos/eight/spring/70.jpg)

> **进入 `doGetBean(name, null, null, false)` 方法**

![](http://120.77.237.175:9080/photos/eight/spring/71.jpg)

关键来了，这里执行 `getSingleton(beanName)` 是够能够获取到 `beanA `实例呢？答案是可以

![](http://120.77.237.175:9080/photos/eight/spring/72.jpg)

> **进入 `getSingleton(beanName, true)` 方法**

`getSingleton(beanName) `调用了其重载方法 `getSingleton(beanName, true)`，接下来的逻辑很重要，让我们来唠嗑唠嗑

![](http://120.77.237.175:9080/photos/eight/spring/73.jpg)

1. `beanA `并没有存放在一级缓存 `singletonObjects `中，因此执行 `Object singletonObject = this.singletonObjects.get(beanName) `后，`singletonObject == null`，再加上 `beanA `正在满足创建的条件`（isSingletonCurrentlyInCreation(beanName) == true）`，因此可以进入第一层 if 判断

   ![](http://120.77.237.175:9080/photos/eight/spring/74.jpg)

2. `beanA `被存放在三级缓存 `singletonFactories` 中，从二级缓存 `earlySingletonObjects` 中获取也是 `null`，因此可以进入第二层 `if` 判断

   ![](http://120.77.237.175:9080/photos/eight/spring/75.jpg)

3. 根据图中所示Quick check for existing instance without full singleton lock,先快速检查已存在的实例，在没有完整的单例锁,然后再Consistent creation of early reference within full singleton lock,在完整的单例锁中一致地创建早期引用,以加锁的方式再重新检查一次一级缓存和二级缓存是否存放`beanA`,答案也是没有的

   ![](http://120.77.237.175:9080/photos/eight/spring/76.jpg)

4. 从三级缓存中获取 `beanA `肯定不为空啦~，因此可以进入第三层 `if` 判断

   ![](http://120.77.237.175:9080/photos/eight/spring/77.jpg)

   - 从单例工厂 `singletonFactory` 中获取 beanA；
   - 将 beanA 添加至二级缓存 `earlySingletonObjects` 中；
   - 将 beanA 从三级缓存 `singletonFactories` 中移除

   ![](http://120.77.237.175:9080/photos/eight/spring/78.jpg)

> **回到 `doGetBean(name, null, null, false)` 方法中**

执行 `Object sharedInstance = getSingleton(beanName)` 将获得之前存入三级缓存 `singletonFactories` 中的 `beanA`

![](http://120.77.237.175:9080/photos/eight/spring/79.jpg)

好家伙，获取到 `beanA `后就直接返回了

![](http://120.77.237.175:9080/photos/eight/spring/80.jpg)

> **回到 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法中**

执行 `valueResolver.resolveValueIfNecessary(pv, originalValue)` 方法获取到 `beanA `实例

![](http://120.77.237.175:9080/photos/eight/spring/81.jpg)

将属性 `beanA `添加到 `deepCopy `集合中（`List<PropertyValue> deepCopy = new ArrayList<>(original.size())`）

![](http://120.77.237.175:9080/photos/eight/spring/82.jpg)

执行 `bw.setPropertyValues(new MutablePropertyValues(deepCopy))` 方法将会填充 `beanB `中的 `a` 属性

![](http://120.77.237.175:9080/photos/eight/spring/83.jpg)

> **进入 `bw.setPropertyValues(new MutablePropertyValues(deepCopy))` 方法**

调用了其重载方法 `setPropertyValues(pvs, false, false)`

![](http://120.77.237.175:9080/photos/eight/spring/84.jpg)

> **进入 `setPropertyValues(pvs, false, false)` 方法**

在该方法中会对 `bean `的每一个属性进行填充（通过 `setPropertyValues(pvs, false, false)` 方法对属性进行赋值）

![](http://120.77.237.175:9080/photos/eight/spring/85.jpg)

> **回到 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法中**

此时 `bw` 包裹着 `beanB`，执行 `bw.setPropertyValues(new MutablePropertyValues(deepCopy))` 方法会将 `deepCopy` 中的元素依次赋值给 `beanB `的各个属性，此时 `beanB `中的 `a` 属性已经赋值为 `beanA`

![](http://120.77.237.175:9080/photos/eight/spring/86.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

因为 `instanceWrapper `封装了 `beanB`，所以执行了 `populateBean(beanName, mbd, instanceWrapper) `方法后，`beanB `中的 a 属性就已经被填充啦~可以看到 `beanB `中有 `beanA`，但 `beanA `中没有 `beanB`

![](http://120.77.237.175:9080/photos/eight/spring/87.jpg)

执行 `getSingleton(beanName, false)` 方法，传入的参数 `allowEarlyReference = false`，表示不允许从三级缓存 `singletonFactories` 中获取 beanB

![](http://120.77.237.175:9080/photos/eight/spring/88.jpg)

> **进入 `getSingleton(beanName, false)` 方法**

由于传入的参数 `allowEarlyReference = false`，因此第三层 `if` 判断铁定进不去，而 `beanB `在三级缓存 `singletonFactories` 中存着，因此返回的 `singletonObject` 为 `null`

![](http://120.77.237.175:9080/photos/eight/spring/89.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

这里应该是执行 `bean `的 `destroy-method `，应该只会在工厂销毁的时候并且 `bean `为单例的条件下，其内部逻辑才会执行。`registerDisposableBeanIfNecessary(beanName, bean, mbd) `方法的注释如下：Add the given bean to the list of disposable beans in this factory, registering its DisposableBean interface and/or the given destroy method to be called on factory shutdown (if applicable). Only applies to singletons. 最后将 `beanB `返回（属性 a 已经填充完毕）

![](http://120.77.237.175:9080/photos/eight/spring/90.jpg)

> **回到 `createBean(beanName, mbd, args)` 方法**

执行 `doCreateBean(beanName, mbdToUse, args)` 方法得到包装 `beanB `实例（属性 `a` 已经填充完毕），并将其返回

![](http://120.77.237.175:9080/photos/eight/spring/91.jpg)

> **回到 `getSingleton(beanName, () -> { ... }` 方法中**

执行` singletonFactory.getObject() `方法获取到 `beanB `实，这里的 `singletonFactory `是之前调用 `getSingleton(beanName, () -> { ... }` 方法传入的 `Lambda `表达式，然后将 `newSingleton `设置为 `true`

![](http://120.77.237.175:9080/photos/eight/spring/92.jpg)

执行 `addSingleton(beanName, singletonObject)` 方法将 beanB 实例添加到一级缓存 `singletonObjects` 中

![](http://120.77.237.175:9080/photos/eight/spring/93.jpg)

> **进入 `addSingleton(beanName, singletonObject)` 方法**

1. 将 beanB 放入一级缓存 singletonObjects 中

2. 将 beanB 从三级缓存 singletonFactories 中删除（beanB 确实在三级缓存中）

3. 将 beanB 从二级缓存 earlySingletonObjects 中删除（beanB 并不在二级缓存中）

4. 将 beanB 的 beanName 注册到 registeredSingletons 中（之前添加至三级缓存的时候已经注册过啦~）

   ![](http://120.77.237.175:9080/photos/eight/spring/94.jpg)

> **回到 `getSingleton(beanName, () -> { ... }` 方法中**

执行 `addSingleton(beanName, singletonObject)` 将 `beanB `添加到一级缓存 `singletonObjects` 后，将 `beanB `返回

![](http://120.77.237.175:9080/photos/eight/spring/95.jpg)

> **回到 `doGetBean(name, null, null, false)` 方法中**

执行完 `getSingleton(beanName, () -> { ... }` 方法后，得到属性已经填充好的 `beanB`，并且已经将其添加至一级缓存 `singletonObjects` 中

![](http://120.77.237.175:9080/photos/eight/spring/96.jpg)

将 `beanB`返回，想想返回到哪儿去了呢？当初时因为 `beanA `要填充其属性 `b`，才执行了创建 `beanB `的操作，现在返回肯定是将 `beanB `返回给 `beanA`

![](http://120.77.237.175:9080/photos/eight/spring/97.jpg)

------

### beanA 的属性填充

> **回到 `resolveReference(argName, ref)` 方法中**

执行完 `this.beanFactory.getBean(resolvedName)` 方法后，获得了属性填充好的 beanB 实例，并将其实例返回

![](http://120.77.237.175:9080/photos/eight/spring/98.jpg)

> **回到 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法中**

执行完 `valueResolver.resolveValueIfNecessary(pv, originalValue)` 方法后，将获得属性填充好的 `beanB `实例

![](http://120.77.237.175:9080/photos/eight/spring/99.jpg)

将 `b` 属性添加至 `deepCopy` 集合中

![](http://120.77.237.175:9080/photos/eight/spring/100.jpg)

执行 `bw.setPropertyValues(new MutablePropertyValues(deepCopy))` 方法对 beanA 的 `b` 属性进行填充

![](http://120.77.237.175:9080/photos/eight/spring/101.jpg)

> **进入 `setPropertyValues(pvs, false, false)` 方法**

在` bw.setPropertyValues(new MutablePropertyValues(deepCopy)) `方法中调用了 `setPropertyValues(pvs, false, false) `方法，在该方法中会对 `bean `的每一个属性进行填充（通过 `setPropertyValues(pvs, false, false) `方法对属性进行赋值）

![](http://120.77.237.175:9080/photos/eight/spring/102.jpg)

> **回到 `applyPropertyValues(beanName, mbd, bw, pvs)` 方法中**

此时 `bw `中包裹着 `beanA`，，执行 `bw.setPropertyValues(new MutablePropertyValues(deepCopy))` 方法会将 `deepCopy `中的元素依次赋值给 `beanA `的各个属性，此时 `beanA `中的 b 属性已经赋值为 `beanA`，又加上之前 `beanB `中的 a 属性已经赋值为 `beanA`，此时可开启无限套娃模式

![](http://120.77.237.175:9080/photos/eight/spring/103.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

执行完 `populateBean(beanName, mbd, instanceWrapper)` 方法后，可以开启无限套娃模式

![](http://120.77.237.175:9080/photos/eight/spring/104.jpg)

这次执行 `getSingleton(beanName, false)` 方法能获取到 beanA 吗？

![](http://120.77.237.175:9080/photos/eight/spring/105.jpg)

> **进入 `getSingleton(beanName, false)` 方法**

之前 `beanB `中注入 `a` 属性时，将 `beanA `从三级缓存 `singletonFactories` 移动到了二级缓存 `earlySingletonObjects` 中，因此可以从二级缓存 `earlySingletonObjects` 中获取到 `beanA`

![](http://120.77.237.175:9080/photos/eight/spring/106.jpg)

> **回到 `doCreateBean(beanName, mbdToUse, args)` 方法中**

最终将获取到的 beanA 返回

![](http://120.77.237.175:9080/photos/eight/spring/107.jpg)

> **回到 `createBean(beanName, mbd, args)` 方法中**

执行 `doCreateBean(beanName, mbdToUse, args)` 方法后得到 beanA 实例，并将此实例返回

![](http://120.77.237.175:9080/photos/eight/spring/108.jpg)

> **回到 `getSingleton(beanName, () -> { ... }` 方法**

执行 `singletonFactory.getObject()` 方法后将获得 beanA 实例，这里的 `singletonFactory` 是我们传入的 Lambda 表达式（专门用于创建 bean 实例）

![](http://120.77.237.175:9080/photos/eight/spring/109.jpg)

执行 `addSingleton(beanName, singletonObject)` 方法将 beanA 添加到一级缓存 `singletonObjects` 中

![](http://120.77.237.175:9080/photos/eight/spring/110.jpg)

> **进入 `addSingleton(beanName, singletonObject)` 方法**

![](http://120.77.237.175:9080/photos/eight/spring/111.jpg)

1. 将 `beanA `放入一级缓存 `singletonObjects `中
2. 将 `beanA `从三级缓存 `singletonFactories `中删除（beanA 并不在三级缓存中）
3. 将 `beanA `从二级缓存 `earlySingletonObjects `中删除（beanA 确实在二级缓存中）
4. 将 `beanA `的 beanName 注册到 `registeredSingletons `中（之前添加至三级缓存的时候已经注册过啦~）

> **回到 `getSingleton(beanName, () -> { ... }` 方法中**

将 beanA 添加至一级缓存 `singletonObjects` 后，将其返回

![](http://120.77.237.175:9080/photos/eight/spring/112.jpg)

> **回到 `doGetBean(name, null, null, false)` 方法中**

执行 `getSingleton(beanName, () -> { ... }` 方法得到 beanA 实例后，将其返回

![](http://120.77.237.175:9080/photos/eight/spring/113.jpg)

> **回到 `preInstantiateSingletons()` 方法中**

终于要结束了。。。执行完 `getBean(beanName)` 方法后，将得到无限套娃版本的 beanA 和 beanB 实例

![](http://120.77.237.175:9080/photos/eight/spring/114.jpg)

------

## 循环依赖总结

全部 Debug 断点

![](http://120.77.237.175:9080/photos/eight/spring/115.jpg)

------

## Debug 步骤总结

1. 调用`doGetBean()`方法，想要获取`beanA`，于是调用`getSingleton()`方法从缓存中查找`beanA`
2. 在`getSingleton()`方法中，从一级缓存中查找，没有，返回null
3. `doGetBean()`方法中获取到的`beanA`为`null`，于是走对应的处理逻辑，调用`getSingleton()`的重载方法（参数为`ObjectFactory`的)
4. 在`getSingleton()`方法中，先将`beanA_name`添加到一个集合中，用于标记该`bean`正在创建中。然后回调匿名内部类的`creatBean()`方法
5. 进入`AbstractAutowireCapableBeanFactory#doCreateBean()`，先反射调用构造器创建出`beanA`的实例，然后判断。是否为单例、是否允许提前暴露引用(对于单例一般为true)、是否正在创建中〈即是否在第四步的集合中)。判断为`true`则将`beanA`添加到【三级缓存】中
6. 对`beanA`进行属性填充，此时检测到`beanA`依赖于`beanB`，于是开始查找`beanB`
7. 调用`doGetBean()`方法，和上面`beanA`的过程一样，到缓存中查找`beanB`，没有则创建，然后给`beanB`填充属性
8. 此时`beanB`依赖于beanA，调用`getsingleton()`获取beanA，依次从一级、二级、三级缓存中找，此时从三级缓存中获取到`beanA`的创建工厂，通过创建工厂获取到`singletonObject`，此时这个`singletonObject`指向的就是上面在`doCreateBean()`方法中实例化的`beanA`
9. 这样`beanB`就获取到了`beanA`的依赖，于是`beanB`顺利完成实例化，并将`beanA`从三级缓存移动到二级缓存中
10. 随后`beanA`继续他的属性填充工作，此时也获取到了`beanB`，`beanA`也随之完成了创建，回到`getsingleton()`方法中继续向下执行，将`beanA`从二级缓存移动到一级缓存中

![](http://120.77.237.175:9080/photos/eight/spring/116.jpg)

------

## 三级缓存总结

> **Spring 创建 Bean 的两大步骤**

Spring创建`bean`主要分为两个步骤，创建原始`bean`对象，接着去填充对象属性和初始化

每次创建`bean`之前，我们都会从缓存中查下有没有该`bean`，因为是单例，只能有一个。当我们创建 `beanA`的原始对象后，并把它放到三级缓存中，接下来就该填充对象属性了，这时候发现依赖了`beanB`，接着就又去创建`beanB`，同样的流程，创建完 `beanB`填充属性时又发现它依赖了`beanA`又是同样的流程。

不同的是：这时候可以在三级缓存中查到刚放进去的原始对象`beanA`，所以不需要继续创建，用它注入`beanB`，完成`beanB`的创建既然 `beanB`创建好了，所以`b`eanA就可以完成填充属性的步骤了，接着执行剩下的逻辑，闭环完成

Spring解决循环依赖依靠的是`Bean`的“中间态"这个概念，而这个中间态指的是已经实例化但还没初始化的状态，也即半成品。

实例化的过程又是通过构造器创建的，如果A还没创建好出来怎么可能提前曝光，所以构造器的循环依赖无法解决。

> **Spring 的三级缓存**

Spring为了解决单例的循环依赖问题，使用了三级缓存：

1. 其中一级缓存为单例池〈 `singletonObjects`)
2. 二级缓存为提前曝光对象( `earlySingletonObjects`)
3. 三级缓存为提前曝光对象工厂( `singletonFactories`）

```java
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// Quick check for existing instance without full singleton lock
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				synchronized (this.singletonObjects) {
					// Consistent creation of early reference within full singleton lock
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
                        // 一级缓存没有,就去二级缓存找
						singletonObject = this.earlySingletonObjects.get(beanName);
						if (singletonObject == null) {
                            // 二级缓存也没有,就去三级找
							ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
							if (singletonFactory != null) {
                                // 三级缓存有的话,就把它移动到二级缓存
								singletonObject = singletonFactory.getObject();
								this.earlySingletonObjects.put(beanName, singletonObject);
								this.singletonFactories.remove(beanName);
							}
						}
					}
				}
			}
		}
		return singletonObject;
	}
```

假设A、B循环引用，实例化A的时候就将其放入三级缓存中，接着填充属性的时候，发现依赖了B，同样的流程也是实例化后放入三级缓存，接着去填充属性时又发现自己依赖A，这时候从缓存中查找到早期暴露的A，没有AOP代理的话，直接将A的原始对象注入B，完成B的初始化后，进行属性填充和初始化，这时候B完成后，就去完成剩下的A的步骤，如果有AOP代理，就进行AOP处理获取代理后的对象A，注入B，走剩下的流程。
