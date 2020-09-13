---
title: Springboot项目中Redis的简单使用
tags:
  - redis
category:
  - 后端
abbrlink: 6867333c
date: 2020-04-12 20:22:32
---
## 简介
> Redis是一个使用ANSI C编写的开源、支持网络、基于内存、可选持久性的键值对存储数据库。——维基百科

## 安装与配置

### 安装

首先到github上下载你需要的版本 [下载地址](https://github.com/MicrosoftArchive/redis/releases) 

下载之后的文件解压缩，复制到安装目录。本文以*D:/redis*为例。

在安装目录中双击打开redis-server.exe，出现如下字样，代表启动成功。
    
    
    [15008] 14 Mar 10:09:21.250 # Server started, Redis version 3.2.100
    [15008] 14 Mar 10:09:21.250 * DB loaded from disk: 0.000 seconds
    [15008] 14 Mar 10:09:21.250 * The server is now ready to accept connections on port 6379


### 配置
在pom.xml中添加redis依赖
```xml
     <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
     </dependency>
```
打开application.properties配置文件，将以下配置复制

    # Redis数据库索引（默认为0）
    spring.redis.database=0
    # Redis服务器地址
    spring.redis.host=localhost
    # Redis服务器连接端口
    spring.redis.port=6379
    # Redis服务器连接密码（默认为空）
    spring.redis.password=
    # 连接池最大连接数（使用负值表示没有限制）
    spring.redis.jedis.pool.max-active=8
    # 连接池最大阻塞等待时间（使用负值表示没有限制）
    spring.redis.jedis.pool.max-wait=-1
    # 连接池中的最大空闲连接
    spring.redis.jedis.pool.max-idle=8
    # 连接池中的最小空闲连接
    spring.redis.jedis.pool.min-idle=0
    # 连接超时时间（毫秒）
    spring.redis.timeout=5000

有些博主给出的配置中*spring.redis.timeout=0*，我使用时会导致**RedisCommandTimeoutException: Command timed out after no timeout**异常，所以请注意自行修改。

## 使用
在项目的service文件夹新建RedisService.java,放入增删改查等常用命令
```java    
    package com.daysappserver.daysofourlives.service;

		/**
	 	* @Author: z
		* @Date: 2020/3/12 21:45
	    * @Version 1.0
	    */
	public interface RedisService {
	    /**
	     * 存储数据
	     */
	    void set(String key, String value);
	
	    /**
	     * 获取数据
	     */
	    String get(String key);
	
	    /**
	     * 删除数据
	     */
	    void remove(String key);
	
	    /**
	     * 设置过期时间
	     */
	    boolean expire(String key, long expire);
	
	    /**
	     * 判断key是否存在
	     */
	    Boolean hasKey(String key);

	}
```

在impl中新建RedisServiceImpl，实现service中的接口
```java
    package com.daysappserver.daysofourlives.service.impl;

	import com.daysappserver.daysofourlives.service.RedisService;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.redis.core.StringRedisTemplate;
	import org.springframework.stereotype.Service;
	
	import java.util.concurrent.TimeUnit;
	
	/**
	 * @Author: z
	 * @Date: 2020/3/12 21:47
	 * @Version 1.0
	 */
	
	@Service
	public class RedisServiceImpl implements RedisService {
	    @Autowired
	    private StringRedisTemplate stringRedisTemplate;
	
	    @Override
	    public void set(String key, String value) {
	        stringRedisTemplate.opsForValue().set(key, value);
	    }
	
	    @Override
	    public String get(String key) {
	        return stringRedisTemplate.opsForValue().get(key);
	    }
	
	    @Override
	    public boolean expire(String key, long expire) {
	        return stringRedisTemplate.expire(key, expire, TimeUnit.SECONDS);
	    }
	
	    @Override
	    public void remove(String key) {
	        stringRedisTemplate.delete(key);
	    }
	
	    @Override
	    public Boolean hasKey(String key) {
	        return stringRedisTemplate.hasKey(key);
	    }
	
	}
```
通过一个单元测试来检测是否缓存成功
```java
	package com.daysappserver.daysofourlives;
	
	import com.daysappserver.daysofourlives.service.RedisService;
	import org.junit.Assert;
	import org.junit.jupiter.api.Test;
	import org.junit.jupiter.api.extension.ExtendWith;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.test.context.junit.jupiter.SpringExtension;
	import org.springframework.test.context.junit4.SpringRunner;
	
	/**
	 * @Author: z
	 * @Date: 2020/3/14 11:39
	 * @Version 1.0
	 */
	@RunWith(SpringRunner.class)
	@SpringBootTest
	@ExtendWith(SpringExtension.class)
	class RedisTest {
	
	    @Autowired
	    private RedisService redisService;
	    @Test
	    public void redisTest() throws Exception {
	        redisService.set("一个测试用的key", "key的值");
	        Assert.assertEquals(redisService.get("一个测试用的key"),"key的值");
	    }
	}
```

运行之后显示*Tests Passed*,说明缓存成功，且能拿到缓存值。
