---
title: useful third package.md
date: 2019-02-14 10:42:24
categories: IT
tags: [IT,java]
---
# 1. JaVers
  Object auditing and diff framework for Java;JaVers是一个轻量级的对象比较/审计框架,非常易于使用。支持持久化，支持springboot。
  > 如果你需要实时比较生产环境的处理结果和备份环境的处理结果，或是在新系统中重放生产环境的请求，或者像代码一样对对象进行版本管理，那么JaVers就可以成为你的好朋友。它不仅可以比较对象，也可以将比较结果存储在数据库中以便审计。审计是这样的一种需求：作为用户，我希望知道谁改变了状态;是什么时候改变的，以及原先的状态是什么。
[官网地址](https://javers.org/)
  ```xml
    <dependency>
      <groupId>org.javers</groupId>
      <artifactId>javers-core</artifactId>
      <version>5.1.3</version>
    </dependency>
  ```

# 2. MapStruct
MapStruct是一种类型安全的bean映射类生成java注释处理器。
> 我们要做的就是定义一个映射器接口，声明任何必需的映射方法。在编译的过程中，MapStruct会生成此接口的实现。该实现使用纯java方法调用的源和目标对象之间的映射，MapStruct节省了时间，通过生成代码完成繁琐和容易出错的代码逻辑。
> [官网地址](http://mapstruct.org/)
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-jdk8</artifactId>
    <version>1.3.0.Final</version>
</dependency>

```
