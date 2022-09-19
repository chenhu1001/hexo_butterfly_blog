---
title: 自定义Spring Cache的key
date: 2019-06-22 15:38:10
categories: Java
tags: [Java,Spring Cache]
---
在多租户系统中，为了统一处理系统缓存，需在缓存组件中加上租户Id，以下是自定义自定义Spring Cache的key步骤。
## 1、继承RedisCacheManager

```
public class RedisAutoCacheManager extends RedisCacheManager {
    public RedisAutoCacheManager(RedisCacheWriter cacheWriter, RedisCacheConfiguration defaultCacheConfiguration) {
        super(cacheWriter, defaultCacheConfiguration);
    }

    /**
     * 从上下文中获取租户ID，重写@Cacheable value值
     * @param name
     * @return
     */
    @Override
    public Cache getCache(String name) {
        return super.getCache( name + StrUtil.COLON + TenantContextHolder.getTenantId());
    }
}
```
## 2、在Spring Boot启动类中注入容器
```
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
    // 初始化一个RedisCacheWriter
    RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory);

    // 设置CacheManager的值序列化方式为json序列化
    RedisSerializer<Object> jsonSerializer = new GenericFastJsonRedisSerializer();
    RedisSerializationContext.SerializationPair<Object> pair = RedisSerializationContext.SerializationPair.fromSerializer(jsonSerializer);
    RedisCacheConfiguration defaultCacheConfig = RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(pair);

    // 初始化RedisCacheManager
    return new RedisAutoCacheManager(redisCacheWriter, defaultCacheConfig);
}
```
## 3、租户类
```
public class TenantContextHolder {

    private static final ThreadLocal<Integer> CONTEXT = new ThreadLocal<>();

    public static void setTenantId(Integer tenantId) {
        CONTEXT.set(tenantId);
    }

    public static Integer getTenantId() {
        return CONTEXT.get();
    }

    public static void clear() {
        CONTEXT.remove();
    }
}
```
