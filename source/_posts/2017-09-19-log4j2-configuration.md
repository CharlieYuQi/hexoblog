---
layout: post
title: 自定义log4j2配置文件地址
date: 2017-09-18
categories: IT
tags: [IT,Log4j]

---
默认情况下，只要把log4j配置文件放在 CLASSPATH 环境变量所指定的目录， JAVA 启动时会制动加载。实际项目中经常需要把配置文件与打包分离，方便修改，所以需要自定义配置文件加载地址。
# SpringMvc
采用spring mvc框架时需要一般在web.xml里配置
```
<context-param>
    <param-name>log4jConfigLocation</param-name>
    <param-value>classpath:log4j.properties</param-value>
</context-param>
<context-param>
    <param-name>log4jRefreshInterval</param-name>
    <param-value>6000</param-value>
</context-param>
```

# Spring Boot
采用spring boot轻量级框架后，大部分自定义配置都放在application.properties里了
```
logging.config=file:config/log4j2.xm
```

# 无框架
本节为本文重点，发现很多人离了框架就不知道该怎么写代码了。
## log4j
键值对格式
```
PropertyConfigurator.config(filepath) 
```
XML 格式
```
DOMConfigurator.config(filepath) 
```
## log4j2
log4j2里一般不建议使用properties文件，而是使用xml文件。log4j2的版本里自定义配置文件加载地址的接口改为Configurator.initialize了
```java
    static {
        try {
            ConfigurationSource source;
            String relativePath =
                    "config" + System.getProperty("file.separator") + "log4j2.xml";
            File log4jFile = new File(
                    System.getProperty("user.dir") + System.getProperty("file.separator")
                            + relativePath);
            if (log4jFile.exists()) {
                source = new ConfigurationSource(new FileInputStream(log4jFile), log4jFile);
                Configurator.initialize(null, source);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

