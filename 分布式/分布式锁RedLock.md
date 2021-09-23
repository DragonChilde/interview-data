# Redis分布式锁

常见的面试题：

- Redis除了拿来做缓存，你还见过基于Redis的什么用法？
- Redis做分布式锁的时候有需要注意的问题？
- 如果是Redis是单点部署的，会带来什么问题？那你准备怎么解决单点问题呢？
- 集群模式下，比如主从模式，有没有什么问题呢？那你简单的介绍一下Redlock吧？你简历上写redisson，你谈谈。
- Redis分布式锁如何续期？看门狗知道吗？

## boot整合redis搭建超卖程序

使用场景：多个服务间 + 保证同一时刻内 + 同一用户只能有一个请求（防止关键业务出现数据冲突和并发错误）

建两个Module：boot_redis01，boot_redis02

- pom

  ```pom
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <parent>
          <artifactId>springboot-demo</artifactId>
          <groupId>org.example</groupId>
          <version>1.0-SNAPSHOT</version>
      </parent>
      <modelVersion>4.0.0</modelVersion>
  
      <artifactId>demo-redlock1</artifactId>
  
      <dependencies>
          <dependency>
              <groupId>org.redisson</groupId>
              <artifactId>redisson</artifactId>
              <version>3.16.3</version>
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
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis</artifactId>
          </dependency>
  
          <!-- jedis -->
          <dependency>
              <groupId>redis.clients</groupId>
              <artifactId>jedis</artifactId>
              <version>3.7.0</version>
          </dependency>
      </dependencies>
  </project>
  ```

- 配置

  ```
  server.port=3333
  
  spring.redis.database=0
  spring.redis.port=9379
  
  
  #连接池最大连接数（使用负值表示没有限制）默认8
  spring.redis.lettuce.pool.max-active=8
  #连接池最大阻塞等待时间（使用负值表示没有限制）默认-1
  spring.redis.lettuce.pool.max-wait=-1
  #连接池中的最大空闲连接默认8
  spring.redis.lettuce.pool.max-idle=8
  #连接池中的最小空闲连接默认0
  spring.redis.lettuce.pool.min-idle=0
  ```

- 主启动类

  ```java
  package com.redlock;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  /**
   * @title: BootRedis01Application
   * @Author Wen
   * @Date: 22/9/2021 10:42 AM
   * @Version 1.0
   */
  @SpringBootApplication
  public class BootRedis01Application {
      public static void main(String[] args) {
          SpringApplication.run(BootRedis01Application.class);
      }
  }
  ```

- 配置类

  ```java
  package com.redlock.config;
  
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
  import org.springframework.data.redis.serializer.StringRedisSerializer;
  
  import java.io.Serializable;
  
  /**
   * @title: RedisConfig
   * @Author Wen
   * @Date: 22/9/2021 10:53 AM
   * @Version 1.0
   */
  @Configuration
  public class RedisConfig {
  
      @Bean
      public RedisTemplate<String, Serializable> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
          RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();
  
          redisTemplate.setKeySerializer(new StringRedisSerializer());
          redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
          redisTemplate.setConnectionFactory(lettuceConnectionFactory);
          return redisTemplate;
      }
  }
  ```

- Controller

  ```
  package com.redlock.controller;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  /**
   * @title: GoodController
   * @Author Wen
   * @Date: 22/9/2021 11:05 AM
   * @Version 1.0
   */
  @RestController
  public class GoodController {
      @Autowired
      private RedisTemplate redisTemplate;
  
      @Value("${server.port}")
      private String serverPort;
  
      @GetMapping("/buy_goods")
      public String buyGoods() {
  
          Object result = redisTemplate.opsForValue().get("goods:001");
          int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
          String retStr = null;
  
          if (goodsNumber > 0) {
              int realNumber = goodsNumber - 1;
              redisTemplate.opsForValue().set("goods:001", realNumber + "");
              retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;
  
          } else {
              retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
          }
  
          System.out.println(retStr);
          return  retStr;
      }
  }
  ```

- 测试

  - ```
    redis：`set goods:001 100`
    ```

  - 访问:http://localhost:1111/buy_goods

- boot_redis02拷贝boot_redis01

------

### 单机版

> **单机版程序没加锁存在什么问题？**

问题：单机版程序没有加锁，在并发测试下数字不对，会出现超卖现象,并且消费端1和消费2会有重复消费问题,

```
你已经成功秒杀商品，此时还剩余：95件	 服务器端口: 2222
你已经成功秒杀商品，此时还剩余：95件	 服务器端口: 2222
```

解决：加锁，那么问题又来了，加 `synchronized `锁还是 `ReentrantLock `锁呢？

1. `synchronized`：不见不散，等不到锁就会死等

2. `ReentrantLock`：过时不候，`lock.tryLock() `提供一个过时时间的参数，时间一到自动放弃锁

如何选择：根据业务需求来选，如果非要抢到锁不可，就使用 `synchronized `锁；如果可以暂时放弃锁，等会再来强，就使用 `ReentrantLock `锁

> **使用 `synchronized` 锁保证单机版程序在并发下的安全性**

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {
    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {

        synchronized (this) {
            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);
            return  retStr;
        }
    }
}
```

> 注意事项:
>
> 1. 在单机环境下，可以使用 `synchronized `锁或 `Lock `锁来实现。
> 2. 但是在分布式系统中，因为竞争的线程可能不在同一个节点上（同一个 jvm 中），所以需要一个让所有进程都能访问到的锁来实现，比如 redis 或者 zookeeper 来构建;
> 3. 不同进程 jvm 层面的锁就不管用了，那么可以利用第三方的一个组件，来获取锁，未获取到锁，则阻塞当前想要运行的线程

------

### 分布式版

> **分布式部署之后，单机版的锁失效**

**问题**：

分布式部署之后，单机版的锁失效，单机版的锁还是会导致超卖现象，这时就需要需要分布式锁

如下，在我们的两个微服务之上，挡了一个 nginx 服务器，用于实现负载均衡的功能,nginx配置如下

```
    upstream myserver{
        server 127.0.0.1:3333;
        server 127.0.0.1:2222;
    }

    server {
        listen       90;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            # 负责用到的配置
            proxy_pass  http://myserver;
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

这时使用JMeter进行压测访问,可以看到相同的商品被出售两次，出现超卖现象

------

> **3.0 版本的代码：使用 redis 分布式锁**

使用当前请求的 UUID + 线程名作为分布式锁的 value，执行 stringRedisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value) 方法尝试抢占锁，如果抢占失败，则返回值为 false；如果抢占成功，则返回值为 true。最后记得调用 stringRedisTemplate.delete(REDIS_LOCK_KEY) 方法释放分布式锁

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();
        // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
        Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value);
        if (!lockFlag) {
            return "抢锁失败";
        }


        Object result = redisTemplate.opsForValue().get("goods:001");
        int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
        String retStr = null;

        if (goodsNumber > 0) {
            int realNumber = goodsNumber - 1;
            redisTemplate.opsForValue().set("goods:001", realNumber + "");
            retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

        } else {
            retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
        }

        System.out.println(retStr);
        redisTemplate.delete(REDIS_LOCK_KEY);
        return retStr;

    }
}
```

测试代码,可以看到加上分布式锁之后，解决了超卖现象

------

#### finally 版

> **存在的问题**

如果代码在执行的过程中出现异常，那么就可能无法释放锁，因此必须要在代码层面加上 `finally` 代码块，保证锁的释放

> **4.0 版本的代码：保证锁的释放**

加入 finally 代码块，保证锁的释放

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();
        // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
        try {
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value);
            if (!lockFlag) {
                return "抢锁失败";
            }


            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;
    
            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);
           
            return retStr;
        } finally {
            redisTemplate.delete(REDIS_LOCK_KEY);

        }

    }
}
```

------

#### 过期时间版

> **存在的问题**

假设部署了微服务 jar 包的服务器挂了，代码层面根本没有走到 finally 这块，也没办法保证解锁。这个 key 没有被删除，其他微服务就一直抢不到锁，因此我们需要加入一个过期时间限定的 key

> **5.0 版本的代码：设置带过期时间的 key**

执行 `stringRedisTemplate.expire(REDIS_LOCK_KEY, 10L, TimeUnit.SECONDS);` 方法为分布式锁设置过期时间，保证锁的释放

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();
        try {
            // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value);
            //设置过期时间为10s
            redisTemplate.expire(lockFlag, 10L, TimeUnit.SECONDS);
            if (!lockFlag) {
                return "抢锁失败";
            }
            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {
            redisTemplate.delete(REDIS_LOCK_KEY);

        }

    }
}
```

------

#### 加锁原子版

> **存在的问题**

加锁与设置过期时间的操作分开了，假设服务器刚刚执行了加锁操作，然后宕机了，也没办法保证解锁。

> **6.0 版本的代码：保证加锁和设置过期时间为原子操作**

使用 `stringRedisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value, 10L, TimeUnit.SECONDS)` 方法，在加锁的同时设置过期时间，保证这两个操作的原子性

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();

        try {
            // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value,10L, TimeUnit.SECONDS);

            if (!lockFlag) {
                return "抢锁失败";
            }


            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {
            redisTemplate.delete(REDIS_LOCK_KEY);

        }

    }
}
```

------

> **另一个新问题**：张冠李戴，删除了别人的锁

我们无法保证一个业务的执行时间，有可能是 10s，有可能是 20s，也有可能更长。因为执行业务的时候可能会调用其他服务，我们并不能保证其他服务的调用时间。如果设置的锁过期了，当前业务还正在执行，那么就有可能出现超卖问题，并且还有可能出现当前业务执行完成后，释放了其他业务的锁

如下图，假设进程 A 在 T2 时刻设置了一把过期时间为 30s 的锁，在 T5 时刻该锁过期被释放，在 T5 和 T6 期间，Test 这把锁已经失效了，并不能保证进程 A 业务的原子性了。于是进程 B 在 T6 时刻能够获取 Test 这把锁，但是进程 A 在 T7 时刻删除了进程 B 加的锁，进程 B 在 T8 时刻删除锁的时候就蒙蔽了，我 TM 锁呢？

![](http://120.77.237.175:9080/photos/eight/java/redlock/01.jpg)

------

> **7.0 版本的代码：只允许删除自己的锁，不允许删除别人的锁**

在释放锁之前，执行 `value.equalsIgnoreCase(stringRedisTemplate.opsForValue().get(REDIS_LOCK_KEY))` 方法判断是否为自己加的锁

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();

        try {
            // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value,10L, TimeUnit.SECONDS);

            if (!lockFlag) {
                return "抢锁失败";
            }


            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {
            // 判断是否自已加的锁
                 // 判断是否自已加的锁
            if (value.equalsIgnoreCase((String) redisTemplate.opsForValue().get(REDIS_LOCK_KEY))){
                redisTemplate.delete(REDIS_LOCK_KEY);
            }

        }

    }
}
```

------

#### 解锁原子版

> 存在问题

在 finally 代码块中的判断与删除并不是原子操作，假设执行 `if` 判断的时候，这把锁还是属于当前业务，但是有可能刚执行完 `if` 判断，这把锁就被其他业务给释放了，还是会出现误删锁的情况

```java
try {
    // ...
}
finally {
    // 判断加锁与解锁是不是同一个客户端
    if (value.equalsIgnoreCase(stringRedisTemplate.opsForValue().get(REDIS_LOCK_KEY))){
        // 若在此时，这把锁突然不是这个客户端的，则会误解锁
        stringRedisTemplate.delete(REDIS_LOCK_KEY);//释放锁
    }
}
```

------

> **8.1 版本的代码：使用 redis 自身事务保证原子性操作**

开启事务不断监视 `REDIS_LOCK_KEY` 这把锁有没有被别人动过，如果已经被别人动过了，那么继续重新执行删除操作，否则就解除监视

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();

        try {
            // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value, 10L, TimeUnit.SECONDS);

            if (!lockFlag) {
                return "抢锁失败";
            }


            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {

            while (true) {
                //加事务,乐观锁
                redisTemplate.watch(REDIS_LOCK_KEY);
                // 判断是否自已加的锁
                if (value.equalsIgnoreCase((String) redisTemplate.opsForValue().get(REDIS_LOCK_KEY))) {

                    //开启事务
                    redisTemplate.setEnableTransactionSupport(true);
                    redisTemplate.multi();
                    redisTemplate.delete(REDIS_LOCK_KEY);
                    // 判断事务是否执行成功,如果等于null,就是没有删除,删除失败,再回去while循环再重新执行
                    List exec = redisTemplate.exec();
                    if (exec == null) {
                        continue;
                    }
                }
                // 如果删除成功,释放监视器,并且break跳出当前循环
                redisTemplate.unwatch();
                break;
            }
        }

    }
}
```

------

> **8.1 版本的代码：使用 lua 脚本保证原子性操作**

redis 可以通过 `eval` 命令保证代码执行的原子性

```java
package com.redlock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.RedisScript;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Collections;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();

        try {
            // setIfAbsent() 就相当于 setnx,如果不存在就新建锁
            Boolean lockFlag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK_KEY, value, 10L, TimeUnit.SECONDS);

            if (!lockFlag) {
                return "抢锁失败";
            }


            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {


            // lua 脚本，摘自官网
            String script = "if redis.call(\"get\",KEYS[1]) == ARGV[1] then\n" +
                    "    return redis.call(\"del\",KEYS[1])\n" +
                    "else\n" +
                    "    return 0\n" +
                    "end";
            Object result = redisTemplate.execute(RedisScript.of(script, Long.class), Collections.singletonList(REDIS_LOCK_KEY), value);
            if (result.equals(1)) {
                System.out.println("------del REDIS_LOCK_KEY success");
            } else {
                System.out.println("------del REDIS_LOCK_KEY error");
            }
        }

    }
}
```

------

#### 自动续期版

> **存在的问题**

前面已经讲过了：我们无法保证一个业务的执行时间，有可能是 10s，有可能是 20s，也有可能更长。因为执行业务的时候可能会调用其他服务，我们并不能保证其他服务的调用时间。如果设置的锁过期了，当前业务还正在执行，那么之前设置的锁就失效了，就有可能出现超卖问题。

因此我们需要确保 redisLock 过期时间大于业务执行时间的问题，即面临如何对 Redis 分布式锁进行续期的问题

> **redis 与 zookeeper 在 CAP 方面的对比**

redis

redis 异步复制造成的锁丢失， 比如：主节点没来的及把刚刚 set 进来这条数据给从节点，就挂了，那么主节点和从节点的数据就不一致。此时如果集群模式下，就得上 Redisson 来解决

zookeeper

zookeeper 保持强一致性原则，对于集群中所有节点来说，要么同时更新成功，要么失败，因此使用 zookeeper 集群并不存在主从节点数据丢失的问题，但丢失了速度方面的性能

> **9.0 版本的代码：使用 Redisson 实现自动续期功能**

------

1. **注入 `Redisson` 对象**

   在 `RedisConfig` 配置类中注入 `Redisson` 对象

   ```java
   package com.redlock.config;
   
   import org.redisson.Redisson;
   import org.redisson.config.Config;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
   import org.springframework.data.redis.serializer.StringRedisSerializer;
   
   import java.io.Serializable;
   
   /**
    * @title: RedisConfig
    * @Author Wen
    * @Date: 22/9/2021 10:53 AM
    * @Version 1.0
    */
   @Configuration
   public class RedisConfig {
   
       @Value("${spring.redis.host}")
       private String redisHost;
   
       @Value("${spring.redis.port}")
       private String redisPort;
   
       @Bean
       public RedisTemplate<String, Serializable> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
           RedisTemplate<String, Serializable> redisTemplate = new RedisTemplate<>();
           redisTemplate.setKeySerializer(new StringRedisSerializer());
           redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
           redisTemplate.setConnectionFactory(lettuceConnectionFactory);
           return redisTemplate;
       }
   
       @Bean
       public Redisson redisson() {
           Config config = new Config();
           config.useSingleServer().setAddress("redis://"+redisHost+":"+redisPort+"").setDatabase(0);
           return (Redisson) Redisson.create(config);
       }
   }
   ```

2. `Controller`

   ```java
   package com.redlock.controller;
   
   import org.redisson.Redisson;
   import org.redisson.api.RLock;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.data.redis.core.RedisTemplate;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   import java.util.UUID;
   
   /**
    * @title: GoodController
    * @Author Wen
    * @Date: 22/9/2021 11:05 AM
    * @Version 1.0
    */
   @RestController
   public class GoodController {
   
       private static final String REDIS_LOCK_KEY = "redis_lock";
   
       @Autowired
       private RedisTemplate redisTemplate;
   
       @Value("${server.port}")
       private String serverPort;
   
       @Autowired
       private Redisson redisson;
   
       @GetMapping("/buy_goods")
       public String buyGoods() {
           // 当前请求的UUID+线程名
           String value = UUID.randomUUID() + Thread.currentThread().getName();
           //获取锁
           RLock lock = redisson.getLock(REDIS_LOCK_KEY);
           //上锁
           lock.lock();
           try {
   
               Object result = redisTemplate.opsForValue().get("goods:001");
               int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
               String retStr = null;
   
               if (goodsNumber > 0) {
                   int realNumber = goodsNumber - 1;
                   redisTemplate.opsForValue().set("goods:001", realNumber + "");
                   retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;
   
               } else {
                   retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
               }
   
               System.out.println(retStr);
   
               return retStr;
           } finally {
                   //解锁
                   lock.unlock();
           }
   
       }
   }
   ```

3. 测试,并没有出现超卖现象

------

> **9.1 版本的代码：Bug 的完善**

在超高并发的情况下，可能会抛出如下异常，原因是解锁 lock 的线程并不是当前线程

```
attempt to unlock lock,not locked by current thread by node id 
```

在释放锁之前加一个判断：还在持有锁的状态，并且是当前线程持有的锁再解锁

```java
package com.redlock.controller;

import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

/**
 * @title: GoodController
 * @Author Wen
 * @Date: 22/9/2021 11:05 AM
 * @Version 1.0
 */
@RestController
public class GoodController {

    private static final String REDIS_LOCK_KEY = "redis_lock";

    @Autowired
    private RedisTemplate redisTemplate;

    @Value("${server.port}")
    private String serverPort;

    @Autowired
    private Redisson redisson;

    @GetMapping("/buy_goods")
    public String buyGoods() {
        // 当前请求的UUID+线程名
        String value = UUID.randomUUID() + Thread.currentThread().getName();
        //获取锁
        RLock lock = redisson.getLock(REDIS_LOCK_KEY);
        //上锁
        lock.lock();
        try {

            Object result = redisTemplate.opsForValue().get("goods:001");
            int goodsNumber = result == null ? 0 : Integer.parseInt(result.toString());
            String retStr = null;

            if (goodsNumber > 0) {
                int realNumber = goodsNumber - 1;
                redisTemplate.opsForValue().set("goods:001", realNumber + "");
                retStr = "你已经成功秒杀商品，此时还剩余：" + realNumber + "件" + "\t 服务器端口: " + serverPort;

            } else {
                retStr = "商品已经售罄/活动结束/调用超时，欢迎下次光临" + "\t 服务器端口: " + serverPort;
            }

            System.out.println(retStr);

            return retStr;
        } finally {
          // 还在持有锁的状态，并且是当前线程持有的锁再解锁
            if (lock.isLocked() && lock.isHeldByCurrentThread()){
                //解锁
                lock.unlock();
            }
        }
    }
}
```

------

# 分布式锁总结

1. synchronized 锁：单机版 OK，上 nginx分布式微服务，单机锁就不 OK,
2. 分布式锁：取消单机锁，上 redis 分布式锁 SETNX
3. 如果出异常的话，可能无法释放锁， 必须要在 finally 代码块中释放锁
4. 如果宕机了，部署了微服务代码层面根本没有走到 finally 这块，也没办法保证解锁，因此需要有设置锁的过期时间
5. 除了增加过期时间之外，还必须要 SETNX 操作和设置过期时间的操作必须为原子性操作
6. 规定只能自己删除自己的锁，你不能把别人的锁删除了，防止张冠李戴，可使用 lua 脚本或者事务
7. 判断锁所属业务与删除锁的操作也需要是原子性操作
8. redis 集群环境下，我们自己写的也不 OK，直接上 RedLock 之 Redisson 落地实现

