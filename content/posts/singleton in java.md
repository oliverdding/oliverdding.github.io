---
title: "Java单例模式"
subtitle: "单例模式的代码实现"
date: 2021-01-03T16:25:21+08:00
lastmod: 2021-01-03T16:25:21+08:00
draft: true
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "单例模式是设计模式中一种很常见的模式, 本文以代码的形式记录了单例模式的实现方式"
images: []

tags: ["Java", "基础", "设计模式"]
categories: ["Java"]
featuredImage: ""
featuredImagePreview: ""

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
mapbox:
  accessToken: ""
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # 位于 "assets/"
    # 或者
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # 位于 "assets/"
    # 或者
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---

首先一点, 只有枚举饿汉保证不会被反射的方式生成多个实例。

## 饿汉模式

### 普通饿汉

```java
public class HungrySingleton {

    private static final HungrySingleton instance = new HungrySingleton();
    
    private HungrySingleton() {
    }
    
    public static HungrySingleton getInstance() {
        return instance;
    }
}
```

### 枚举饿汉

```java
public enum HungrySingleton {
    INSTANCE{
        // code here
    };
}
```

## 懒汉模式

### DCL(Double Check Lock)

```java
public class LazySingleton {

    private static volatile LazySingleton instance = null; // 必须加上volatile防止指令重排序与保证内存可见性
    
    private LazySingleton() {
    }
    
    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

### Holder模式

```java
public class LazySingleton {

    private static class Holder {
        private static LazySingleton instance = null;
    }
    
    private LazySingleton() {
    }
    
    /**
     * 只有访问了Holder内部类JVM才会加载class
     * 也就实现了懒汉单例
     */
    public static LazySingleton getInstance() {
        return Holder.instance;
    }
}
```
