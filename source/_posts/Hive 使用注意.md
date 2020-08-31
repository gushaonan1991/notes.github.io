---
title: Hive 使用注意
date: 2020-08-31
tags: [Hive, 应用, 技巧, 注意]
categories: [开发技术, Data, Hive]
---
- [Number 与 NULL 比较结果总为 false](#number-与-null-比较结果总为-false)
  - [说明](#说明)
  - [参考](#参考)

## Number 与 NULL 比较结果总为 false
### 说明
不同类型的值做比较时，Hive 默认先进行类型转换，具体转换规则参考官方文档：[Hive Data Types](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Types#LanguageManualTypes-NumericTypes)。

需要关注的是 `Number` 类型与 `NULL` 值比较或 `Number` 类型与 `Non-Number` 类型比较时，比较结果总是 `NULL`，也就等同与 `false`。

### 参考
* [What happens when compare string to int in hive?](https://stackoverflow.com/questions/50849151/what-happens-when-compare-string-to-int-in-hive)