# Spingboot2.x

## 数据库连接池

实际上，在JDBC内核API的实现下，就已经可以实现对数据库的访问了，那么我们为什么还需要数据源呢？主要出于以下几个目的：

1. 封装关于数据库访问的各种参数，实现统一管理
2. 通过对数据库的连接池管理，节省开销并提高效率

在Java这个自由开放的生态中，已经有非常多优秀的开源数据源可以供大家选择，比如：DBCP、C3P0、Druid、HikariCP等。

而在Spring Boot 2.x中，对数据源的选择也紧跟潮流，采用了目前性能最佳的[HikariCP](https://github.com/brettwooldridge/HikariCP)。

## cache

默认ConcurrentHashMap

## 参数校验

**JSR-303定义的是什么标准？**

JSR-303 是JAVA EE 6 中的一项子规范，叫做Bean Validation，Hibernate Validator 是 Bean Validation 的参考实现 . Hibernate Validator 提供了 JSR 303 规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。

**Bean Validation中内置的constraint**

**Hibernate Validator附加的constraint**