# Java单例模式


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

