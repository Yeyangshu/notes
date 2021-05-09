# Springboot Redis多数据源配置

## 1 pom文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency> 
```

## 2 Spring配置文件

```properties
#单点配置
spring.redis.host=192.168.1.1
spring.redis.port=6379

#哨兵配置
spring.redis.sentinel.master=common
spring.redis.sentinel.nodes=192.168.1.84:26379,192.168.1.85:26379
spring.redis.password=123456

#集群配置
spring.redis.cluster.nodes=192.168.1.24:6389,192.168.1.24:6479,192.168.1.24:6579
spring.redis.password=123456
```

所有配置文件

```properties
## Redis数据库索引（使用的数据库（0-15），默认为0）
spring.redis.database=0

## 连接URL，将覆盖主机，端口和密码（用户将被忽略），
## 例如：redis://user:password@example.com:6379
spring.redis.url=
## Redis服务器主机: 127.0.0.1
spring.redis.host=localhost
## Redis服务器连接密码（默认为空）
spring.redis.password=
## Redis服务器连接端口
spring.redis.port=6379

## 启用SSL支持
spring.redis.ssl=false
## 连接超时时间（毫秒）
spring.redis.timeout=0

## jedis连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
## 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
## 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
## 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
## 空闲连接池剔除时间
spring.redis.jedis.pool.time-between-eviction-runs

## cluster集群
spring.redis.cluster.nodes=
spring.redis.cluster.max-redirects=

## sentinel哨兵
spring.redis.sentinel.master=
spring.redis.sentinel.nodes=
spring.redis.sentinel.password

## lettuce莴苣、生菜
spring.redis.lettuce.pool.max-active=8
spring.redis.lettuce.pool.max-wait=-1
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.min-idle=0
spring.redis.lettuce.shutdown-timeout=100
spring.redis.lettuce.pool.time-between-eviction-runs
spring.redis.lettuce.shutdown-timeout

## Spring Boot 2.3.0 新特性Redis拓扑动态感应，根据redis cluster集群的变化，动态改变客户端的节点情况，完成故障转移。
spring.redis.lettuce.cluster.refresh.adaptive
spring.redis.lettuce.cluster.refresh.period
```

## 3 配置类

### 3.1 父类

```java
package com.yeyangshu.redis.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.lettuce.core.resource.ClientResources;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.data.redis.connection.*;
import org.springframework.data.redis.connection.lettuce.LettuceClientConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettucePoolingClientConfiguration;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.net.URI;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * 美国弗吉尼亚redis集群配置
 **/
public class AbstractRedisConfig {

    /**
     * 创建Lettuce连接工厂
     *
     * @param clientResources 客户端资源
     * @param properties Redis配置
     * @return Lettuce连接工厂
     */
    public LettuceConnectionFactory createLettuceConnectionFactory(ClientResources clientResources, RedisProperties properties) {
        // 1 获取客户端配置
        LettuceClientConfiguration clientConfiguration = getLettuceClientConfiguration(clientResources, properties);

        // 2 Redis各场景配置
        // 2.1 Redis集群配置
        RedisClusterConfiguration clusterConfig = getClusterConfiguration(properties);
        if (clusterConfig != null) {
            return new LettuceConnectionFactory(clusterConfig, clientConfiguration);
        }
        // 2.2 Redis哨兵配置
        RedisSentinelConfiguration sentinelConfig = getSentinelConfig(properties);
        if (sentinelConfig != null) {
            return new LettuceConnectionFactory(sentinelConfig, clientConfiguration);
        }
        // 2.3 Redis单机配置
        return new LettuceConnectionFactory(getStandaloneConfig(properties), clientConfiguration);
    }

    /**
     * 获取客户端配置
     *
     * @param clientResources 客户端资源
     * @param properties Redis配置
     * @return 客户端配置
     */
    private LettuceClientConfiguration getLettuceClientConfiguration(ClientResources clientResources, RedisProperties properties) {
        // 1 创建Lettuce客户端配置构建器
        LettuceClientConfiguration.LettuceClientConfigurationBuilder builder = createBuilder(properties);
        // 2 通过配置文件中的属性设置构建器
        setBuilderFromProperties(builder, properties);
        // 3 通过URL信息设置构建器
        if (StringUtils.isNotBlank(properties.getUrl())) {
            setBuilderFromUrl(builder, properties);
        }
        // 4 设置ClientResources
        builder.clientResources(clientResources);
        // 5 构建LettuceClientConfiguration
        return builder.build();
    }

    /**
     * 创建Lettuce客户端配置构建器
     *
     * @param properties Redis配置
     * @return Lettuce客户端配置构建器
     */
    private LettuceClientConfiguration.LettuceClientConfigurationBuilder createBuilder(RedisProperties properties) {
        // 1 获取连接池配置
        RedisProperties.Pool poolProperties = Optional.ofNullable(properties.getLettuce())
                .map(RedisProperties.Lettuce::getPool).orElse(null);
        // 2 无连接池配置，使用默认配置
        if (poolProperties == null) {
            return LettuceClientConfiguration.builder();
        }
        // 3 有连接池配置，使用连接池配置
        return LettucePoolingClientConfiguration.builder().poolConfig(getPoolConfig(poolProperties));
    }

    /**
     * 获取连接池配置
     *
     * @param poolProperties 连接池配置
     * @return 连接池配置
     */
    private GenericObjectPoolConfig<?> getPoolConfig(RedisProperties.Pool poolProperties) {
        GenericObjectPoolConfig<?> config = new GenericObjectPoolConfig<>();
        // 最大连接数（负数表示没有界限）
        config.setMaxTotal(poolProperties.getMaxActive());
        // 最大空闲连接数（负数表示没有界限）
        config.setMaxIdle(poolProperties.getMaxIdle());
        // 最小空闲连接数
        config.setMinIdle(poolProperties.getMinIdle());
        // 最大阻塞时间（负数表示没有界限）
        if (poolProperties.getMaxWait() != null) {
            config.setMaxWaitMillis(poolProperties.getMaxWait().toMillis());
        }
        // 空闲线程剔除时间
        if (poolProperties.getTimeBetweenEvictionRuns() != null) {
            config.setTimeBetweenEvictionRunsMillis(poolProperties.getTimeBetweenEvictionRuns().toMillis());
        }
        return config;
    }

    /**
     * 获取集群配置类
     *
     * @param properties Redis配置属性
     * @return Redis集群配置类
     */
    private RedisClusterConfiguration getClusterConfiguration(RedisProperties properties) {
        RedisProperties.Cluster clusterProperties = properties.getCluster();
        if (clusterProperties == null) {
            return null;
        }

        // 用于连接到Redis集群的RedisConnectionFactory配置的RedisConnection配置类。在设置高可用性Redis环境时很有用
        RedisClusterConfiguration clusterConfig = new RedisClusterConfiguration(clusterProperties.getNodes());
        // 设置最大重定向数
        if (clusterProperties.getMaxRedirects() != null) {
            clusterConfig.setMaxRedirects(clusterProperties.getMaxRedirects());
        }
        // 设置密码
        if (StringUtils.isNotBlank(properties.getPassword())) {
            clusterConfig.setPassword(RedisPassword.of(properties.getPassword()));
        }
        return clusterConfig;
    }

    /**
     * 应用配置属性
     *
     * @param builder 构建器
     * @param properties Redis配置文件
     */
    private void setBuilderFromProperties(LettuceClientConfiguration.LettuceClientConfigurationBuilder builder, RedisProperties properties) {
        // 设置启用SSL连接
        if (properties.isSsl()) {
            builder.useSsl();
        }
        // 设置连接超时时间
        if (properties.getTimeout() != null) {
            builder.commandTimeout(properties.getTimeout());
        }
        // 设置shutdownTimeout
        if (properties.getLettuce() != null) {
            RedisProperties.Lettuce lettuce = properties.getLettuce();
            boolean hasShutDownTimeout = lettuce.getShutdownTimeout() != null && !lettuce.getShutdownTimeout().isZero();
            if (hasShutDownTimeout) {
                builder.shutdownTimeout(properties.getLettuce().getShutdownTimeout());
            }
        }
    }

    /**
     * 从url信息中设置构建器
     *
     * @param builder 构建器
     * @param properties 配置文件
     */
    private void setBuilderFromUrl(LettuceClientConfiguration.LettuceClientConfigurationBuilder builder, RedisProperties properties) {
        ConnectionInfo connectionInfo = parseUrl(properties.getUrl());
        if (connectionInfo.isUseSsl()) {
            builder.useSsl();
        }
    }

    /**
     * 解析url
     *
     * @param url url配置
     * @return Redis连接信息
     */
    private ConnectionInfo parseUrl(String url) {
        try {
            URI uri = new URI(url);
            boolean useSsl = (url.startsWith("redis://"));
            String password = null;
            if (uri.getUserInfo() != null) {
                password = uri.getUserInfo();
                int index = password.indexOf(':');
                if (index >= 0) {
                    password = password.substring(index + 1);
                }
            }
            return new ConnectionInfo(uri, useSsl, password);
        } catch (URISyntaxException ex) {
            throw new IllegalArgumentException("Malformed url '" + url + "'", ex);
        }
    }

    /**
     * 连接信息
     */
    private static class ConnectionInfo {
        /** url */
        private final URI uri;

        /** 是否启用ssl */
        private final boolean useSsl;

        /** 密码 */
        private final String password;

        public ConnectionInfo(URI uri, boolean useSsl, String password) {
            this.uri = uri;
            this.useSsl = useSsl;
            this.password = password;
        }

        public boolean isUseSsl() {
            return this.useSsl;
        }

        public String getHostName() {
            return this.uri.getHost();
        }

        public int getPort() {
            return this.uri.getPort();
        }

        public String getPassword() {
            return this.password;
        }

    }

    /**
     * 获取哨兵配置
     *
     * @param properties Redis配置
     * @return 哨兵配置
     */
    private RedisSentinelConfiguration getSentinelConfig(RedisProperties properties) {
        RedisProperties.Sentinel sentinelProperties = properties.getSentinel();
        if (sentinelProperties == null) {
            return null;
        }

        RedisSentinelConfiguration config = new RedisSentinelConfiguration();
        config.master(sentinelProperties.getMaster());
        config.setSentinels(createSentinels(sentinelProperties));
        if (properties.getPassword() != null) {
            config.setPassword(RedisPassword.of(properties.getPassword()));
        }
        config.setDatabase(properties.getDatabase());
        return config;
    }

    /**
     * 创建哨兵节点列表
     *
     * @param sentinel 哨兵配置信息
     * @return Redis节点列表
     */
    private List<RedisNode> createSentinels(RedisProperties.Sentinel sentinel) {
        List<RedisNode> nodes = new ArrayList<>();
        for (String node : sentinel.getNodes()) {
            try {
                String[] parts = StringUtils.split(node, ":");
                nodes.add(new RedisNode(parts[0], Integer.valueOf(parts[1])));
            } catch (RuntimeException e) {
                throw new IllegalStateException("Invalid redis sentinel " + "property '" + node + "'", e);
            }
        }
        return nodes;
    }

    /**
     * 获取单机配置
     *
     * @param properties Redis配置
     * @return 单机配置
     */
    private RedisStandaloneConfiguration getStandaloneConfig(RedisProperties properties) {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
        if (StringUtils.isNotBlank(properties.getUrl())) {
            ConnectionInfo connectionInfo = parseUrl(properties.getUrl());
            config.setHostName(connectionInfo.getHostName());
            config.setPort(connectionInfo.getPort());
            config.setPassword(RedisPassword.of(connectionInfo.getPassword()));
        } else {
            config.setHostName(properties.getHost());
            config.setPort(properties.getPort());
            config.setPassword(RedisPassword.of(properties.getPassword()));
        }
        config.setDatabase(properties.getDatabase());
        return config;
    }

    /**
     * 构建StringRedisTemplate
     *
     * @param redisConnectionFactory redisConnectionFactory
     * @return StringRedisTemplate
     */
    protected StringRedisTemplate buildStringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    /**
     * 构建StringRedisTemplate
     *
     * @param redisConnectionFactory redisConnectionFactory
     * @return StringRedisTemplate
     */
    protected RedisTemplate<String, Object> buildRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // key采用String的序列化方式
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(jackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

### 3.2 美国多机房配置

```java
package com.yeyangshu.redis.config;

import io.lettuce.core.resource.ClientResources;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;

/**
 * 美国弗吉尼亚Redis集群配置
 **/
@Configuration
public class UsRedisConfig extends AbstractRedisConfig {

    @Bean("usStringRedisTemplate")
    @Primary
    public StringRedisTemplate usStringRedisTemplate(@Qualifier("usLettuceConnectionFactory") RedisConnectionFactory redisConnectionFactory) {
        return buildStringRedisTemplate(redisConnectionFactory);
    }

    @Bean("usRedisTemplate")
    @Primary
    public RedisTemplate usRedisTemplate(@Qualifier("usLettuceConnectionFactory") RedisConnectionFactory redisConnectionFactory) {
        return buildRedisTemplate(redisConnectionFactory);
    }

    @Bean("usLettuceConnectionFactory")
    @Primary
    public LettuceConnectionFactory usLettuceConnectionFactory(@Qualifier("usRedisProperties") RedisProperties properties, ClientResources clientResources) {
        return createLettuceConnectionFactory(clientResources, properties);
    }

    /** 配置文件属性注入 */
    @Bean("usRedisProperties")
    @Primary
    @ConfigurationProperties(prefix = "spring.redis.us")
    public RedisProperties usRedisProperties() {
        return new RedisProperties();
    }

}
```

### 3.3 新加坡机房配置

```java
package com.yeyangshu.redis.config;

import io.lettuce.core.resource.ClientResources;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.data.redis.RedisProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;

/**
 * 新加坡Redis集群配置
 **/
@Configuration
public class SgRedisConfig extends AbstractRedisConfig {

    @Bean("sgStringRedisTemplate")
    public StringRedisTemplate sgStringRedisTemplate(@Qualifier("sgLettuceConnectionFactory") RedisConnectionFactory redisConnectionFactory) {
        return buildStringRedisTemplate(redisConnectionFactory);
    }

    @Bean("sgRedisTemplate")
    public RedisTemplate sgRedisTemplate(@Qualifier("sgLettuceConnectionFactory") RedisConnectionFactory redisConnectionFactory) {
        return buildRedisTemplate(redisConnectionFactory);
    }

    @Bean("sgLettuceConnectionFactory")
    public LettuceConnectionFactory sgLettuceConnectionFactory(@Qualifier("sgRedisProperties") RedisProperties properties, ClientResources clientResources) {
        return createLettuceConnectionFactory(clientResources, properties);
    }

    /** 配置文件属性注入 */
    @Bean("sgRedisProperties")
    @ConfigurationProperties(prefix = "spring.redis.sg")
    public RedisProperties sgRedisProperties() {
        return new RedisProperties();
    }

}
```

## 4 使用样例

```java
@Resource(name = "usRedisTemplate")
RedisTemplate<String, String> usRedisTemplate;

@Resource(name = "sgRedisTemplate")
RedisTemplate<String, String> sgRedisTemplate;
```

## 5 资料

- https://blog.csdn.net/moakun/article/details/104060304
- https://blog.csdn.net/maxi1234/article/details/113883800
- 源码：https://blog.csdn.net/u013412772/article/details/80315120

