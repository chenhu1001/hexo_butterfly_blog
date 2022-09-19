---
title: Spring Cloud Gateway实现指定路由跳过全局过滤器
date: 2022-04-25 09:49:14
categories: Java
tags: [Java,Spring Cloud,Spring Cloud Gateway]
---
在Spring Cloud Gateway中GlobalFilter可以方便的全局拦截或统计，有时候希望在某些路由中可以跳过GlobalFilter，可以通过GatewayFilter与GlobalFilter组合来实现。
## 1、全局过滤器GlobalFilter
详细代码如下：

```java
package com.chanjet.dsf.web.filter;

import com.alibaba.fastjson.JSON;
import com.chanjet.dsf.cache.RedisUtil;
import com.chanjet.dsf.constant.AttrbuteConstant;
import com.chanjet.dsf.constant.ResultCodeEnum;
import com.chanjet.dsf.result.ResultFactory;
import com.chanjet.dsf.web.models.AuthTokenPo;
import com.chanjet.dsf.web.util.Constant;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.core.io.buffer.DataBuffer;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.nio.charset.StandardCharsets;

@Component
public class AuthFilter implements GlobalFilter, Ordered {

    Logger logger = LoggerFactory.getLogger(AuthFilter.class);

    private static final String TOKEN_PREFIX = "DSFTOKEN::";

    @Autowired
    RedisUtil redisUtil;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getQueryParams().getFirst(Constant.QUERY_PARAMS.TOKEN);

        logger.info("=======AuthFilter:token======" + token);
        // 不判断token的过滤器，比如登录接口
        if (exchange.getAttribute(AttrbuteConstant.ATTRIBUTE_IGNORE_TEST_GLOBAL_FILTER) != null
            && exchange.getAttribute(AttrbuteConstant.ATTRIBUTE_IGNORE_TEST_GLOBAL_FILTER).equals(true)) {
            return chain.filter(exchange);
        }

        if (StringUtils.isEmpty(token)) {
            // 返回一个错误码 前端去做跳转淘宝授权登录页面 （前端处理）
            return generateJSON(exchange, ResultFactory.getErrorResult(ResultCodeEnum.TOKEN_IS_NULL));
        }
        
        String rediskey = TOKEN_PREFIX + token;
        // 通过token取缓存
        try {
            AuthTokenPo authTokenPo = (AuthTokenPo) redisUtil.get(rediskey);
            //判断token过期
            // 校验缓存的账号信息是否为空
            if (authTokenPo == null) {
                return generateJSON(exchange, ResultFactory.getErrorResult(ResultCodeEnum.TOKEN_IS_INVALID));
            }

        } catch (Exception e) {
            logger.error("AuthFilter.filter 错误信息{}", e.getMessage(), e);
        }
        
        return chain.filter(exchange);
    }
    
    private <T>Mono generateJSON(ServerWebExchange exchange, T t){
        ServerHttpResponse response = exchange.getResponse();
        byte[] bits = JSON.toJSONString(t).getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = response.bufferFactory().wrap(bits);
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        //指定编码，否则在浏览器中会中文乱码
        response.getHeaders().add("Content-Type", "text/plain;charset=UTF-8");
        return response.writeWith(Mono.just(buffer));
    }

    @Override
    public int getOrder() {
        return -900;
    }
}
```
## 2、AbstractGatewayFilterFactory 局部过滤器
内部类写法，是为了指定过滤器的优先级，要优先于全局过滤器，否则
容易造成全局过滤器拦截到指定局部过滤器的配置内容，从而导致局
部过滤器失效。
```java
package com.chanjet.dsf.web.filter;

import com.chanjet.dsf.constant.AttrbuteConstant;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * 设置忽略路由
 */
@Component
public class IgnoreAuthFilter extends AbstractGatewayFilterFactory<IgnoreAuthFilter.Config> {

    Logger logger = LoggerFactory.getLogger(IgnoreAuthFilter.class);

    public IgnoreAuthFilter() {
        super(Config.class);
        logger.info("IgnoreFilter 进入 IgnoreAuthGatewayFilterFactory ");
    }

    @Override
    public GatewayFilter apply(Config config) {
        logger.info("IgnoreFilter 进入  apply");
        // ===============================注意=================================
        // 下面的内部类写法，是为了指定过滤器的优先级，要优先于全局过滤器，否则
        // 容易造成全局过滤器 拦截到指定 局部过滤器的配置内容。
        return new InnerFilter(config);
    }

    /**
     * 创建一个内部类，来实现2个接口，指定顺序
     * 这里通过Ordered指定优先级
     */
    private class InnerFilter implements GatewayFilter, Ordered {

        private Config config;

        InnerFilter(Config config) {
            this.config = config;
        }

        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            logger.info("进入innerFilter=====" + config.isIgnoreGlobalFilter());
            if (config.isIgnoreGlobalFilter() == true) {
                exchange.getAttributes().put(AttrbuteConstant.ATTRIBUTE_IGNORE_TEST_GLOBAL_FILTER, true);
            }
            return chain.filter(exchange);
        }

        @Override
        public int getOrder() {
            return -1000;
        }
    }

    public static class Config{
        boolean ignoreGlobalFilter;

        public boolean isIgnoreGlobalFilter() {
            return ignoreGlobalFilter;
        }

        public void setIgnoreGlobalFilter(boolean ignoreGlobalFilter) {
            this.ignoreGlobalFilter = ignoreGlobalFilter;
        }
    }

	// 这个name方法 用来在yml配置中指定对应的过滤器名称
    @Override
    public String name() {
        return "IgnoreAuthFilter";
    }
}
```
## 3、yml配置

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      routes:
        - id: dsf-report-ser
          uri: lb://dsf-report-ser
          predicates:
            - Path=/report/**
        - id: dsf-account-ser
          uri: lb://dsf-account-ser
          predicates:
            - Path=/account/**,/setting/**
          filters:
            - name: IgnoreAuthFilter #本路由跳过全局过滤器
              args:
                ignoreGlobalFilter: true
        - id: dsf-finance-ser
          uri: lb://dsf-finance-ser
          predicates:
            - Path=/shopTrade/**,/goods/**,/cost/**
```
