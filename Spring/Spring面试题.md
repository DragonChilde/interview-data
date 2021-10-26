# Spring中支持的常用数据库事务传播属性和事务隔离级别

## 事务的传播行为

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播，列如方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行，事务传播的行为有传播属性指定，Spring定义了7中类传播行为

| **传播属性** |                           **描述**                           |
| :----------: | :----------------------------------------------------------: |
|   REQUIRED   | 如果有事务在运行，当前的方法就在这个事务内运行，否则就启动一个新的事务，并在自己的事务内运行 |
| REQUIRED_NEW | 当前方法必须启动事务，并在它自己的事务内运行，如果有事务正在运行，应该将他挂起 |
|   SUPPORTS   | 如果有事务在运行，当前的方法就在这个事务内运行，否则他可以不运行在事务中 |
| NOT_SUPPORTE |   当前的方法不应该运行在事务中，如果有运行的事务，将他挂起   |
|  MANDATORY   | 当前的方法必须运行在事务内部，如果没有正在运行的事务，就抛出异常 |
|    NEVER     |   当前方法不应该运行在事务中，如果有运行的事务，就抛出异常   |
|    NESTED    | 如果有事务在运行，当前的方法就应该在这个事物的嵌套事务内运行，否则，就启动一个新的事务，并在它自己的事务内运行 |

> 事务传播属性可以在@Transactional注解的propagation属性中定义

------

## 事务隔离级别

### 数据库事务并发问题

假设现在有两个事务：`Transaction01`和`Transaction02`并发执行。

- 脏读
  1. `Transaction01`将某条记录的AGE值从20修改为30
  2. `Transaction02`读取了`Transaction01`更新后的值：30
  3. `Transaction01`回滚，AGE值恢复到了20
  4. `Transaction02`读取到的30就是一个无效的值
- 不可重复读
  1. `Transaction01`读取了AGE值为20
  2. `Transaction02`将AGE值修改为30
  3. `Transaction01`再次读取AGE值为30，和第一次读取不一致
- 幻读
  1. `Transaction01`读取了STUDENT表中的一部分数据
  2. `Transaction02`向STUDENT表中插入了新的行
  3. `Transaction01`读取了STUDENT表时，多出了一些行

------

### 隔离级别

数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题。**一个事务与其他事务隔离的程度称为隔离级别**。SQL标准中规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱

1. **读未提交**：READ UNCOMMITTED

   允许Transaction01读取Transaction02未提交的修改。

2. **读已提交**：READ COMMITTED

    要求Transaction01只能读取Transaction02已提交的修改。

3. **可重复读**：REPEATABLE READ

   确保Transaction01可以多次从一个字段中读取到相同的值，即Transaction01执行期间禁止其它事务对这个字段进行更新

4. **串行化**：SERIALIZABLE

   确保Transaction01可以多次从一个表中读取到相同的行，在Transaction01执行期间，禁止其它事务对这个表进行添加、更新、删除操作。可以避免任何并发问题，但性能十分低下。

5. 各个隔离级别解决并发问题的能力见下表

   ![](http://120.77.237.175:9080/photos/eight/spring/117.png)

总结

```
//1.请简单介绍Spring支持的常用数据库事务传播属性和事务隔离级别？

	/**
	 * 事务的属性：
	 * 	1.★propagation：用来设置事务的传播行为
	 * 		事务的传播行为：一个方法运行在了一个开启了事务的方法中时，当前方法是使用原来的事务还是开启一个新的事务
	 * 		-Propagation.REQUIRED：默认值，使用原来的事务
	 * 		-Propagation.REQUIRES_NEW：将原来的事务挂起，开启一个新的事务
	 * 	2.★isolation：用来设置事务的隔离级别
	 * 		-Isolation.REPEATABLE_READ：可重复读，MySQL默认的隔离级别
	 * 		-Isolation.READ_COMMITTED：读已提交，Oracle默认的隔离级别，开发时通常使用的隔离级别
	 */
```

------

## Spring MVC 如果解决 POST 请求中文乱码问题？

1. 解决 POST 请求中文乱码问题

   修改项目中`web.xml`文件

   ```xml
     <filter>
       <filter-name>CharacterEncodingFilter</filter-name>
       <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
       <init-param>
         <param-name>encoding</param-name>
         <param-value>UTF-8</param-value>
       </init-param>
       <init-param>
         <param-name>forceEncoding</param-name>
         <param-value>true</param-value>
       </init-param>
     </filter>
     <filter-mapping>
       <filter-name>CharacterEncodingFilter</filter-name>
       <url-pattern>/*</url-pattern>
     </filter-mapping>
   ```

2. 解决 Get 请求中文乱码问题

   修改`tomcat`中`server.xml`文件

   ```xml
   <Connector URIEncoding="UTF-8" port="8080" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443" />
   
   ```

------

# SpringMvc工作流程

整体流程

`SpringMVC`框架是一个基于请求驱动的Web框架，并且使用了‘前端控制器’模型来进行设计，再根据‘请求映射规则’分发给相应的页面控制器进行处理

![](http://120.77.237.175:9080/photos/eight/spring/118.png)

具体步骤：

1. 首先用户发送请求到前端控制器，前端控制器根据请求信息（如 URL）来决定选择哪一个页面控制器进行处理并把请求委托给它，即以前的控制器的控制逻辑部分；图中的 1、2 步骤；

2. 页面控制器接收到请求后，进行功能处理，首先需要收集和绑定请求参数到一个对象，这个对象在 `Spring Web MVC` 中叫命令对象，并进行验证，然后将命令对象委托给业务对象进行处理；处理完毕后返回一个 `ModelAndView`（模型数据和逻辑视图名）；图中的 3、4、5 步骤；

3. 前端控制器收回控制权，然后根据返回的逻辑视图名，选择相应的视图进行渲染，并把模型数据传入以便视图渲染；图中的步骤 6、7；

4. 前端控制器再次收回控制权，将响应返回给用户，图中的步骤 8；至此整个结束。

   ![](http://120.77.237.175:9080/photos/eight/spring/119.png)

具体步骤：

第一步：发起请求到前端控制器(`DispatcherServlet`)

第二步：前端控制器请求`HandlerMapping`查找 Handler （可以根据`xml`配置、注解进行查找）

第三步：处理器映射器`HandlerMapping`向前端控制器返回Handler，`HandlerMapping`会把请求映射为`HandlerExecutionChain`对象（包含一个Handler处理器（页面控制器）对象，多个`HandlerInterceptor`拦截器对象），通过这种策略模式，很容易添加新的映射策略

第四步：前端控制器调用处理器适配器去执行Handler

第五步：处理器适配器`HandlerAdapter`将会根据适配的结果去执行Handler

第六步：Handler执行完成给适配器返回`ModelAndView`

第七步：处理器适配器向前端控制器返回`ModelAndView `（`ModelAndView`是`springmvc`框架的一个底层对象，包括 `Model`和view）

第八步：前端控制器请求视图解析器去进行视图解析 （根据逻辑视图名解析成真正的视图(`jsp`)），通过这种策略很容易更换其他视图技术，只需要更改视图解析器即可

第九步：视图解析器向前端控制器返回`View`

第十步：前端控制器进行视图渲染 （视图渲染将模型数据(在`ModelAndView`对象中)填充到request域）

第十一步：前端控制器向用户响应结果

> 总结核心开发步骤

1. `DispatcherServlet `在` web.xml` 中的部署描述，从而拦截请求到 `Spring Web MVC`

2. `HandlerMapping `的配置，从而将请求映射到处理器

3. `HandlerAdapter `的配置，从而支持多种类型的处理器

   注：处理器映射求和适配器使用纾解的话包含在了注解驱动中，不需要在单独配置

4. `ViewResolver `的配置，从而将逻辑视图名解析为具体视图技术

5. 处理器（页面控制器）的配置，从而进行功能处理

   View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf…）

![](http://120.77.237.175:9080/photos/eight/spring/120.png)

