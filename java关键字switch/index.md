# Java关键字switch


## 基本概念

switch语句和if else类似是一种选择语句。根据整数表达式的值，switch语句从一系列代码中选出一段执行。

```java
switch (key) {
case 1:
    //do something...
    break;
case 2:
    //do something...
    break;
default:
    //do something...
    break;
}
```

<!-- more -->

#### 注意：

1. switch 语句可以处理 int，short，byte，char 类型的值，但是不能处理 long。因为 short，byte，char都会转换成 int 进行处理，这一点也可以从生成的字节码看出。
2. 在 JDK 5 中加入枚举 Enum 类型也可以作为 case 值的。
3. ** JDK 7 中加入字符串 String 类型作为 case 值的，** 这是Java提供的语法糖，本质上switch语句处理还是整型。

## 原理分析  



#### switch对字符串支持的实现

先上段demo

```java
/**
 * Java Program to demonstrate how string in switch functionality is implemented in * Java SE 7 release.
 */
public class StringInSwitchCase {
    public static void main(String[] args) {
        String mode = "ACTIVE";
        switch (mode) {
            case "ACTIVE":
                System.out.println("Application is running on Active mode");
                break;
            case "PASSIVE":
                System.out.println("Application is running on Passive mode");
                break;
            case "SAFE":
                System.out.println("Application is running on Safe mode");
            default:
                    System.out.println("Application unknow");
                break;
        }
    }
}
```

利用IDEA打开StringInSwitchCase.class文件，反编译后，结果如下：

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

public class StringInSwitchCase {
    public StringInSwitchCase() {
    }

    public static void main(String[] args) {
        String mode = "ACTIVE";
        byte var3 = -1;
        switch(mode.hashCode()) {
        case -74056953:
            if (mode.equals("PASSIVE")) {
                var3 = 1;
            }
            break;
        case 2537357:
            if (mode.equals("SAFE")) {
                var3 = 2;
            }
            break;
        case 1925346054:
            if (mode.equals("ACTIVE")) {
                var3 = 0;
            }
        }

        switch(var3) {
        case 0:
            System.out.println("Application is running on Active mode");
            break;
        case 1:
            System.out.println("Application is running on Passive mode");
            break;
        case 2:
            System.out.println("Application is running on Safe mode");
        default:
            System.out.println("Application unknow");
        }

    }
}
```

看到反编译后的代码，可知原来**字符串的switch是通过`equals()`和`hashCode()`方法来实现的。**  

> **记住，本质上switch中只能使用整型**，比如`byte`、`short`、`char`(ackii码是整型)以及`int`。`hashCode()`方法返回的是`int`，而不是`long`。通过这个很容易记住`hashCode`返回的是`int`这个事实。 

> 仔细看下可以发现，进行`switch`的实际是哈希值，然后通过使用equals方法比较进行安全检查，这个检查是必要的，因为哈希可能会发生碰撞。因此它的性能是不如使用枚举进行switch或者使用纯整数常量，但这也不是很差。因为Java编译器只增加了一个`equals`方法，如果你比较的是字符串字面量的话会非常快，比如”abc” ==”abc”。如果你把`hashCode()`方法的调用也考虑进来了，那么还会再多一次的调用开销，因为字符串一旦创建了，它就会把哈希值缓存起来。因此如果这个`siwtch`语句是用在一个循环里的，比如逐项处理某个值，或者游戏引擎循环地渲染屏幕，这里`hashCode()`方法的调用开销其实不会很大。

### 参考文章：

1. [http://www.importnew.com/14597.html](http://www.importnew.com/14597.html)
2. [https://blog.csdn.net/u012420654/article/details/59707677](https://blog.csdn.net/u012420654/article/details/59707677)
3. [https://www.jianshu.com/p/9103ab536bbe](https://www.jianshu.com/p/9103ab536bbe)




