from: https://www.baeldung.com/spring-null-safety-annotations
by [Nguyen Nam Thai](https://www.baeldung.com/author/namthai-nguyen/)

## 1. 概览

Spring 5开始，通过*null-safety*的方式帮助我们编写安全代码，一批注解像卫兵一样帮我们监视潜在的空引用。

这并意味着是我们彻底的摆脱不安全的代码，*null-safety*特性会在编译时产生警告。这类警告防止我们在运行时发生灾难性的NPEs。



## 2. @NonNull 注释

`@NonNull`注释是*null-safety*特性中至关重要的一个注释。我们可以在任何一个对象期望引用的地方，利用这个注释声明非空约束：域，方法参数，方法返回值。

如`Person`的类：

```java
public class Person {
    private String fullName;
    
    void setFullName(String fullName) {
        if (fullName != null && fullName.isEmpty) {
            fullName = null;
        }
        this.fullName = fullName;
    }
    //getter
}
```

该类定义有效，但是有个缺陷 - *fullName*这个域可能被置为空，这就可能引发NPE当我们在别处使用该域时。

Spring *null-safety*特性能够通过IDE向我们汇报警告。如，当我们使用Intellij IDEA，对*fullName*使用`NonNull`注解，我们能看到如下警告:

![e1](pic/nonnul-annotation.png)

有了这个提示，我们能及早注意到并采取一些手段避免在运行时发生故障。



## 3. @NonNullFields 注解

`@NonNull`注解能保证*null-safety*。然而，过分依赖这个注解在操作*non-full*域时，导致代码污染。

可以通过`@NonNullFields`避免`NonNull`的滥用。这个注解用于包作用级别（package-level）,用于通知IDE该包下的所有域为非空。

通过*package-info.java*文件加上`@NonNullFields`使用，如：

```java
@NonNullFields
package org.example.nullibility;
```



## 4. @Nullable 注解

`@NonNullFields`比起`@NonNull`注解能有效减少样板代码。通过`@Nullable`能避免*non-null*带来的约束。



## 5. @NonNullAPi 注解

`NonNullFields`仅作用在方法参数上，如果我们想要针对方法返回值获得相同的功能，我们需要使用`NonNullApi`。

该注解通用是在*package-info.java*中声明，如：

```java
@NonNullAPi
package org.example.nullbility;
```



## 6. 结论

使用Spring *null-safety*尽可能的帮助我们减少NPE。然而，我们仍需注意以下亮点当我们使用该特性时：

* 该功能只有在IDE中能获得支持，如Intellij IDEA
* 并不会强制在运行时进行非空检查 - 我们仍需要在自己编写代码时避免NPE





*注：NPE = NullPointerException，空指针异常