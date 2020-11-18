---
title: Mockito 使用入门
date: 2020-08-18
categories: [开发技术, Java, 工具应用]
tags: [工具, 应用, 入门, 测试]
---
- [基础入门](#基础入门)
- [应用进阶](#应用进阶)
- [官方文档](#官方文档)
- [接口文档](#接口文档)
- [常见问题](#常见问题)

# 基础入门
* [Learn Mockito](https://www.tutorialspoint.com/mockito/index.htm)

# 应用进阶
* [Mockito Tutorial Series](https://www.baeldung.com/mockito-series)

# 官方文档
* [Mockito Github Home](https://github.com/mockito/mockito/wiki)

# 接口文档
* [Mockito Core JavaDoc](https://javadoc.io/static/org.mockito/mockito-core/3.5.0/org/mockito/Mockito.html)

# 常见问题
1. 怎样联合使用 `@RunWith(SpringJunit4ClassRunner.class)` 和 `@RunWith(MockitoJunitRunner.class)` ？
   
   目前无法联合使用，如果强行在父类加 Spring 注解，并在子类加 Mockito 注解，可能导致由 Spring 主动注入的 Bean 没有完成初始化。

   解决方案：在 `@Before` 方法中执行 `MockitoAnnotations.init(this)` 主动初始化测试类中各 Mockito 注解。

   参考：
   * [Multiple RunWith Statements in JUnit](https://stackoverflow.com/questions/24431427/multiple-runwith-statements-in-junit)

2. 仅希望 mock 特定方法，其他均调用实际方法，如何实现？
   
   1. 以 `Mockito.spy()` 替换 `Mockito.mock()`，两者区别是 spy 方式默认调用实际方法，mock 方式默认不做处理。

   2. 使用 `Mockito.doCallRealMethod()`。