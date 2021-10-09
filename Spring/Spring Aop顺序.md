# Spring Aop顺序

## Aop 常用注解

> **Spring 中的 5 个通知**

1. `@Before` 前置通知: 目标方法之前执行
2. `@After` 后置通知: 目标方法之后执行（始终执行）
3. `@AfterReturning` 返回后通知: 执行方法结束前执行(异常不执行)
4. `@AfterThrowing` 异常通知: 出现异常时候执行
5. `@Around` 环绕通知: 环绕目标方法执行

## Spring Aop 面试题

> > **面试官对线环节**

1. 你肯定知道 Spring，那说说 Aop 的全部通知顺序 Springboot 或 Springboot2 对 Aop 的执行顺序影响？
2. 说说你使用 Aop 中碰到的坑

## 测试前的准备工作

### 业务类

> > **创建业务接口类：`CalcService`**

```java
package com.example.demo.inter;

/**
 * @title: CalcService
 * @Author Wen
 * @Date: 8/10/2021 下午5:55
 * @Version 1.0
 */
public interface CalcService {
    public int div(int x, int y);
}
```

> > **创建业务接口的实现类：`CalcServiceImpl`**

```java
package com.example.demo.inter.impl;

import com.example.demo.inter.CalcService;
import org.springframework.stereotype.Service;

/**
 * @title: CalcServiceImpl
 * @Author Wen
 * @Date: 8/10/2021 下午5:56
 * @Version 1.0
 */
@Service
public class CalcServiceImpl implements CalcService {
    @Override
    public int div(int x, int y) {
        int result = x / y;
        System.out.println("=========>CalcServiceImpl被调用了,我们的计算结果：" + result);
        return result;
    }
}
```

### 切面类

> **想在除法方法前后各种通知，引入切面编程**

 

1. `@Aspect`：指定一个类为切面类
2. `@Component`：纳入 Spring 容器管理

> **创建切面类 `MyAspect`**

```java
package com.example.demo.aspcet;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @title: MyAspect
 * @Author Wen
 * @Date: 8/10/2021 下午6:02
 * @Version 1.0
 */
@Aspect
@Component
public class MyAspect {
    @Before("execution(public int com.example.demo.inter.impl.CalcServiceImpl.*(..))")
    public void beforeNotify() {
        System.out.println("******** @Before我是前置通知MyAspect");
    }

    @After("execution(public int com.example.demo.inter.impl.CalcServiceImpl.*(..))")
    public void afterNotify() {
        System.out.println("******** @After我是后置通知");
    }

    @AfterReturning("execution(public int com.example.demo.inter.impl.CalcServiceImpl.*(..))")
    public void afterReturningNotify() {
        System.out.println("********@AfterReturning我是返回后通知");
    }

    @AfterThrowing("execution(public int com.example.demo.inter.impl.CalcServiceImpl.*(..))")
    public void afterThrowingNotify() {
        System.out.println("********@AfterThrowing我是异常通知");
    }

    @Around("execution(public int com.example.demo.inter.impl.CalcServiceImpl.*(..))")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        Object retValue = null;
        System.out.println("我是环绕通知之前AAA");
        retValue = proceedingJoinPoint.proceed();
        System.out.println("我是环绕通知之后BBB");
        return retValue;
    }

}
```

## Spring4 下的测试

### POM 文件

> **在 POM 文件中导入 SpringBoot 1.5.9.RELEASE 版本**

SpringBoot 1.5.9.RELEASE 版本的对应的 Spring 版本为 4.3.13 Release

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>demo</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- <version>1.5.9.RELEASE</version〉 ch/qos/Logback/core/joran/spi/JoranException解决方案-->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.1.3</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-access</artifactId>
            <version>1.1.3</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.1.3</version>
        </dependency>

        <!-- web+actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>

        <!-- Spring Boot AOP技术-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <!-- 一般通用基础配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

### 创建测试类

**注意**：SpringBoot 1.5.9 版本在测试类上需要加上 `@RunWith(SpringRunner.class)` 注解，单元测试需要导入的包名为 `import org.junit.Test;`

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class AopTests {

  @Autowired private CalcService calcService;

  @Test
  public void testAop4() {
    System.out.println("spring版本：" + SpringVersion.getVersion() + "\t" + "SpringBoot版本：" + SpringBootVersion.getVersion());
    System.out.println();
    calcService.div(10, 2);
    // calcService.div(10, 0);
  }
}
```

### Aop 测试结果

> **正常执行的结果**

环绕通知将前置通知与目标方法包裹住，执行完 `@After` 才执行 `@AfterReturning`

```
spring版本：4.3.13.RELEASE	SpringBoot版本：1.5.9.RELEASE

我是环绕通知之前AAA
******** @Before我是前置通知MyAspect
=========>CalcServiceImpl被调用了,我们的计算结果：5
我是环绕通知之后BBB
******** @After我是后置通知
********@AfterReturning我是返回后通知
```

> **异常执行的结果**

由于抛出了异常，因此环绕通知后半部分没有执行，执行完 `@After` 才执行 `@AfterThrowing`

```
spring版本：4.3.13.RELEASE	SpringBoot版本：1.5.9.RELEASE

我是环绕通知之前AAA
******** @Before我是前置通知MyAspect
******** @After我是后置通知
********@AfterThrowing我是异常通知

java.lang.ArithmeticException: / by zero
```

> **注**：Spring4 默认用的是 JDK 的动态代理

## Spring 5 下的测试

### POM 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.12.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>demo</description>
	<properties>
		<java.version>1.8</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>

        <!-- springboot-aop 技术 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

	</dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>


</project>

```

### 创建测试类

> **在 Spring4 的测试类下修改代码**

> 注意：``SpringBoot 2.3.3 `版本下，不需要在测试类上面添加 `@RunWith(SpringRunner.class)`` 直接，单元测试需要导入的包名为 `import org.junit.jupiter.api.Test`;，不再使用`` import org.junit.Test`;

```java
package com.example.demo;

import com.example.demo.inter.CalcService;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringBootVersion;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.core.SpringVersion;

@SpringBootTest
//@RunWith(SpringRunner.class)
public class AopTests {

  @Autowired private CalcService calcService;

  @Test
  public void testAop4() {

    System.out.println("spring版本：" + SpringVersion.getVersion() + "\t" + "SpringBoot版本：" + SpringBootVersion.getVersion());
    System.out.println();
    //calcService.div(10, 2);
     calcService.div(10, 0);

  }

  @Test
  public void testAop5() {

    System.out.println("spring版本：" + SpringVersion.getVersion() + "\t" + "SpringBoot版本：" + SpringBootVersion.getVersion());
    System.out.println();
    calcService.div(10, 5);
    //calcService.div(10, 0);

  }
}

```

### Aop 测试结果

> **正常执行的结果**

感觉 Spring5 的环绕通知才是真正意义上的华绕通知，它将其他通知和方法都包裹起来了，而且 `@AfterReturning` 和 `@After` 之前，合乎逻辑！

```
spring版本：5.2.15.RELEASE	SpringBoot版本：2.3.12.RELEASE

我是环绕通知之前AAA
******** @Before我是前置通知MyAspect
=========>CalcServiceImpl被调用了,我们的计算结果：2
********@AfterReturning我是返回后通知
******** @After我是后置通知
我是环绕通知之后BBB
```

> **异常执行的结果**

由于方法抛出了异常，因此环绕通知后半部分没有执行，并且 `@AfterThrowing` 和 `@After` 之前

```
spring版本：5.2.8.RELEASE	SpringBoot版本：2.3.3.RELEASE

我是环绕通知之前AAA
******** @Before我是前置通知MyAspect
********@AfterThrowing我是异常通知
******** @After我是后置通知

java.lang.ArithmeticException: / by zero
```

## Aop 执行顺序总结

![](http://120.77.237.175:9080/photos/eight/spring/01.png)

