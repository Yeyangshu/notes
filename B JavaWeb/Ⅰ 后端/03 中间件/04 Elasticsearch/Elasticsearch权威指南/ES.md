## 1 maven

```xml
<dependencies>
    <!-- Java High Level REST Client -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-high-level-client</artifactId>
        <version>${elasticseach-version}</version>
    </dependency>
    
    <!-- 高级客户端依赖下面两个 -->
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>${elasticseach-version}</version>
    </dependency>
    <!-- Java REST Client -->
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>elasticsearch-rest-client</artifactId>
        <version>${elasticseach-version}</version>
    </dependency>
</dependencies>
```

查询

```java
GET /execute_log/_search?index=_wwMkngBqcKy8eqbieus
```

