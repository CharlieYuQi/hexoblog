---
layout: post
title: opentsdb特殊字符问题
date: 2017-09-18
categories: IT
tags: [IT,OpenTSDB]
---

# Opentsdb对特殊字符规范如下;


> The following rules apply to metric and tag values:
Strings are case sensitive, i.e. "Sys.Cpu.User" will be stored separately from "sys.cpu.user"
Spaces are not allowed
Only the following characters are allowed: a to z, A to Z, 0 to 9, -, _, ., / or Unicode letters (as per the specification)

# 代码实现如下：
```java
public final class net.opentsdb.core.Tags
  /**
   * Ensures that a given string is a valid metric name or tag name/value.
   * @param what A human readable description of what's being validated.
   * @param s The string to validate.
   * @throws IllegalArgumentException if the string isn't valid.
   */
  public static void validateString(final String what, final String s) {
    if (s == null) {
      throw new IllegalArgumentException("Invalid " + what + ": null");
    } else if ("".equals(s)) {
      throw new IllegalArgumentException("Invalid " + what + ": empty string");
    }
    final int n = s.length();
    for (int i = 0; i < n; i++) {
      final char c = s.charAt(i);
      if (!(('a' <= c && c <= 'z') || ('A' <= c && c <= 'Z')
          || ('0' <= c && c <= '9') || c == '-' || c == '_' || c == '.'
          || c == '/' || Character.isLetter(c) || isAllowSpecialChars(c))) {
        throw new IllegalArgumentException("Invalid " + what
            + " (\"" + s + "\"): illegal character: " + c);
      }
    }
  }
```

# 解决办法
1. 从上述代码里可以看到Opentsdb支持自定义允许的特殊字符，*isAllowSpecialChars*，

```java
public final class net.opentsdb.core.TSDB

    if (config.getString("tsd.core.tag.allow_specialchars") != null) {
      Tags.setAllowSpecialChars(config.getString("tsd.core.tag.allow_specialchars"));
    }
```

所以只需要在opentsdb的配置文件里加上该属性配置就可以，例如 *tsd.core.tag.allow_specialchars = :* 就将“:”设置为允许的字符了。
很奇怪的在于opentsdb的官方文档里没有列出该配置属性 [OpenTSDB Configuration](http://opentsdb.net/docs/build/html/user_guide/configuration.html)

2、写入opentsdb之前进行数据转换。
```java
    private static String convert(String in) {
        return in == null ? null : in.toLowerCase().replaceAll("[^-_a-z0-9.\\u4e00-\\u9fa5]", "_");
    }
```
