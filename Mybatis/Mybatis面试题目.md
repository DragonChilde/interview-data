## Mybatis中当实体类中的属性名和表中的字段不一样，怎么办

解决方案：

1. 写 `SQL `语句的时候 写别名

2. 在`MyBatis`的全局配置文件中开启驼峰命名规则

   ```xml
   <!-- 开启驼峰命名规则，可以将数据库中下划线映射为驼峰命名
   	列如 last_name 可以映射为 lastName
   -->
   <setting name="mapUnderscoreToCameLCase" value="true" />
   ```

   > 要求 数据库字段中含有下划线

3. 在Mapper映射文件中使用 `resultMap `自定义映射

   ```xml
   <!-- 
   	自定义映射
   -->
   <resultMap type="com.atguigu.pojo.Employee" id="myMap">
       <!-- 映射主键 -->
   	<id cloumn="id" property="id"/>
       <!-- 映射其他列 -->
       <result column="last_name" property="lastName" />
       <result column="email" property="email" />
       <result column="salary" property="salary" />
       <result column="dept_id" property="deptId" />
   </resultMap>
   ```

   
