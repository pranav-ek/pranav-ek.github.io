---
layout: post
title: Spring cache abstraction using Redis
date: '2016-12-25 07:32:14'
---

####  1. Overview
This post will focus on integrating the [Redis](//redis.io/) with the Spring Boot application to leverage the declarative annotation based caching support provided by the Spring [cache abstraction](//docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html).

#### 2. Add Redis dependency to the application
Start by adding the Redis dependency to the build script file - *build.gradle*
{% highlight java %}
compile('org.springframework.boot:spring-boot-starter-data-redis')
{% endhighlight %}
#### 3. Redis Configuration class
To make the Spring resolve the caches by annotation, extend the [CachingConfigurerSupport](//docs.spring.io/spring/docs/current/javadoc-api/org/springframework/cache/annotation/CachingConfigurerSupport.html) class and annotate the config class with [@EnableCaching](//docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/EnableCaching.html).
{% highlight java %}
@Configuration
@EnableCaching
public class RedisConfiguration extends CachingConfigurerSupport {...}
{% endhighlight %}
#####    3.1 Redis client 
Jedis is one of the popular Redis Java client, configure the JedisConnectionFactory which will create [Jedis](//github.com/xetorthio/jedis) instances for connecting to the Redis server.
{% highlight java %}
  @Bean
  public JedisConnectionFactory jedisConnectionFactory() {
    JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
    jedisConnectionFactory.setHostName("127.0.0.1");
    jedisConnectionFactory.setPort(6379);
    jedisConnectionFactory.setUsePool(true);
    return jedisConnectionFactory;
  }
{% endhighlight %}
#####    3.2 RedisTemplate
Create a [RedisTemplate](//docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/core/RedisTemplate.html) instance which helps to serialize/deserialize between the object and binary data on the Redis store.

Here StringRedisSerializer is used for serializing the key and [GenericJackson2JsonRedisSerializer](//docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/serializer/GenericJackson2JsonRedisSerializer.html) for the value. This template is a generified one, it can be wired to multiple components and reused as it's thread-safe.


{% highlight java %}
  @Bean
  public StringRedisSerializer stringRedisSerializer() {
    StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
    return stringRedisSerializer;
  }

  @Bean
  public GenericJackson2JsonRedisSerializer genericJackson2JsonRedisJsonSerializer() {
    GenericJackson2JsonRedisSerializer genericJackson2JsonRedisJsonSerializer =
        new GenericJackson2JsonRedisSerializer();
    return genericJackson2JsonRedisJsonSerializer;
  }

  @Bean
  public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
    redisTemplate.setConnectionFactory(jedisConnectionFactory());
    redisTemplate.setExposeConnection(true);
    redisTemplate.setKeySerializer(stringRedisSerializer());
    redisTemplate.setValueSerializer(genericJackson2JsonRedisJsonSerializer());
    return redisTemplate;
{% endhighlight %}
#####    3.3 RedisCacheManager
The cache abstraction does not provide an actual store and relies on abstraction materialized by the org.springframework.cache.Cache and org.springframework.cache.CacheManager interfaces.
[RedisCacheManager](//docs.spring.io/spring-data/redis/docs/current/api/org/springframework/data/redis/cache/RedisCacheManager.html) is the implimentation of the CacheManager for Redis. 
{% highlight java %}
  @Bean
  public RedisCacheManager cacheManager() {
    RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate());
    redisCacheManager.setTransactionAware(true);
    redisCacheManager.setLoadRemoteCachesOnStartup(true);
    redisCacheManager.setUsePrefix(true);
    return redisCacheManager;
  }
{% endhighlight %}

#### 4. Annotating the methods
For caching declaration, the abstraction provides a set of Java annotations:

##### 4.1 @Cacheable 
Triggers cache population
{% highlight java %}
    @Cacheable(value = "whitePaper", key = "#id")
    public WhitePaper findWhitePaperById(String id) 
    {
        return repository.findById(id);
    }
{% endhighlight %}
##### 4.2 @CachePut 
Updates the cache without interfering with the method execution
{% highlight java %}
    @CachePut(value = "whitePaper", key = "#whitePaper?.id")
    public WhitePaper updateWhitePaper(WhitePaper whitePaper)
    {
        return repository.update(whitePaper);
    }
{% endhighlight %}

##### 4.3 @CacheEvict 
Triggers cache eviction
{% highlight java %}
    @CacheEvict(value = "whitePaper", key = "#id.toString()")
    public WhitePaper saveWhitePaper(int id){
        return repository.delete(id);
    }
{% endhighlight %}
#### 5. Conclusion
Redis is not a simple key-value store, it can be called as a data structure store as it's can store data in advanced data structures. Spring Cache abstraction is a high level abstraction for interacting with the store.
