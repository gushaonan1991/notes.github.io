---
title: 勿要滥用 Optional
date: 2020-10-15
categories: 
    - 开发技术
    - Java
    - 语言技巧
tags: 
    - Optional
    - Java
    - Null
---

- [Optional 个人理解](#optional-个人理解)
- [Optional 滥用问题](#optional-滥用问题)
  - [滥用表现](#滥用表现)
  - [解决思路](#解决思路)
- [参考](#参考)

# Optional 个人理解
Optional 是 Java 语言从 JDK8 版本开始提供的一种容器类，它可以承接、包裹任意类型的对象（包括 null），并在其上提供 getter 方法及 null 值处理方法。

个人理解，Optional 主要解决两方面问题：
1. 解决传统工程中 null 值判断过于频繁所造成处理逻辑繁复的问题。
2. 解决分散在工程各处的、隐藏的 NullPointerException 异常频发的问题。
                           
# Optional 滥用问题
## 滥用表现
1. 类中声明 Optional 类型成员变量。
2. 方法中声明 Optional 类型参数。
3. 私有方法声明 Optional 类型返回值。
4. 所有 getter 类型方法都声明 Optional 类型返回值。
5. 本应返回非 null 对象的业务处理方法声明了一个 Optional 类型返回值。

## 解决思路
要理解为什么上述几种使用方式属于滥用，不仅要清楚 Optional 的功能和用法，更要理清 Optional 在代码中展现的或显然或隐含的含义：  
* 接口方法返回了 Optional 对象时，我们可以认为接口提供方明确声明该接口可能返回无效（null）结果，如何应对就交由调用方来决策。
* 接口方法返回非 Optional 对象时，我们认为接口提供方隐性声明该接口仅会返回有效（非 null）结果，验证和保障结果有效性的能力由方法或承载类来提供。
  * 保障 getter 类型方法不会返回 null：在构造函数或成员 setter 方法中就对传入的参数做 null 检测，提前抛出 NPE。
  * 保障业务处理方法不会返回 null：处理逻辑执行结束后、返回结果前做 null 检测。

示例：
```java
/**
 * 用户认证信息。
 */
public class UserIdentifyInfo {
    private String idNumber; // 身份证号，不会是 null
    private String userName; // 用户名，不会是 null
    private String address;  // 住址，可能是 null

    public UserIdentifyInfo(String idNumber, String userName, String address) {
        this.idNumber = Preconditions.checkNotNull(idNumber);
        this.userName = Preconditions.checkNotNull(userName);
        this.address = address;
    }

    public void setIdNumber(String idNumber) {
        this.idNumber = Preconditions.checkNotNull(idNumber);
    }

    public String getIdNumber() {
        return this.idNumber;
    }

    public void setUserName(String userName) {
        this.userName = Preconditions.checkNotNull(userName);
    }

    public String getUserName() {
        return this.userName;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public Optional<String> getAddress() {
        return Optional.ofNullable(this.address);
    }
}

/**
 * 用户认证服务。
 */
public class UserIdentifyService {

    public Optional<String> findUserById(String idNumber) {
        // 业务逻辑代码
        String userName = ...;
        return Optional.ofNullable(userName);
    }
}
```

综上可以看出，仅涉及对外交互的接口返回 Optional 才有意义，这也是 1、2、3 点属于滥用的原因。  
不过针对第三点（私有方法返回 Optional）可以根据实际场景适当放宽限定，比如为实现更优雅的流式调用，可以让部分细粒度、业务逻辑简单的 private 方法也返回 Optional。  

即使是对外交互接口，也要看业务逻辑是否支持返回 Optional，如果结果不是可选而是必需的，在方法内抛出异常更合理。

# 参考
* [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
* [Java SE 8 Optional, a pragmatic approach](https://blog.joda.org/2015/08/java-se-8-optional-pragmatic-approach.html)
* [Intention Revealing Code With Optional](https://nipafx.dev/intention-revealing-code-java-8-optional#)
* [Should Java 8 getters return optional type?](https://stackoverflow.com/questions/26327957/should-java-8-getters-return-optional-type)
* [26 Reasons Why Using Optional Correctly Is Not Optional](https://dzone.com/articles/using-optional-correctly-is-not-optional)