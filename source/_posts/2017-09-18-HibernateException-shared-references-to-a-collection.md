---
layout: post
title: HibernateException  Found shared references to a collection
date: 2017-09-18
categories: IT
tags: [IT,Hibernate]
---
# 问题现象
项目采用SpringBoot框架，数据库访问采用JPA。新增功能时出现`HibernateException: Found shared references to a collection`
具体情况是权限管理模型基于RBAC，有User，Role，Authority 三个实体以及上述三者的两两关联表，共计六张表。
当创建一个用户时，首先通过 Role 获取 Authority List，再赋值给 User的 Authority List，最后将用户写入数据库时抛出以上异常。
数据模型如下，省略部分字段。

<!-- more -->

```java
@Entity
public class User {
    @ManyToMany
    @JoinTable(name = "UserAuthority", joinColumns = { @JoinColumn(name = "userId") }, inverseJoinColumns = { @JoinColumn(name = "authorityId") })
    private List<Authority> authorities;

    @ManyToOne(fetch = FetchType.EAGER)
    private Role role;
}
```
```java
@Entity
public class Role {
    @OneToMany(mappedBy = "role")
    private List<User> user;

    @ManyToMany
    @JoinTable(name = "RoleAuthority", joinColumns = { @JoinColumn(name = "roleId") }, inverseJoinColumns = { @JoinColumn(name = "authorityId") })
    private List<Authority> authorities;
}
```
```java
@Entity
public class Authority {
    @ManyToMany(mappedBy = "authorities")
    private Set<Role> roles;

    @ManyToMany(mappedBy = "authorities")
    private Set<User> users;
}
```
问题代码如下：
```java
    private User createUser(String name, String role) {
        User user = new User();
        user.setName(name);
        user.setRole(roleService.findByRole(role));
        List<Authority> authorities = roleService.getAuthoritiesByRole(role);
        user.setAuthorities(authorities);
        return user;
    }
```

# 问题原因
参见
[Hibernate UserGuide 2.8.1. Collections as a value type](http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html) 两个实体不可以共享同一个集合，上例中 Role 和 User 共用了同一个  Authority List，最终导致以上问题。
> Two entities cannot share a reference to the same collection instance. Collection-valued properties do not support null value semantics because Hibernate does not distinguish between a null collection reference and an empty collection.

# 解决办法
新建一个集合，将原来的集合元素添加进去，并赋值给另一个实体
```java
    private User createUser(String name, String role) {
        User user = new User();
        user.setName(name);
        user.setRole(roleService.findByRole(role));
        List<Authority> authorities = roleService.getAuthoritiesByRole(role);

        List<Authority> clonedAuthorities = new ArrayList<>(authorities.size());
        authorities.forEach(authority -> clonedAuthorities.add(authority));

        user.setAuthorities(clonedAuthorities);
        return user;
    }
```
