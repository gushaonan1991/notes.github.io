---
title: Vim 使用技巧
date: 2020-09-01
tags: [Vim, 编辑器, 技巧, 使用]
categories: [日常应用, 编辑工具]
---
- [编辑技巧](#编辑技巧)
  - [如何改变大小写？](#如何改变大小写)
- [替换技巧](#替换技巧)
  - [如何实现零宽断言？](#如何实现零宽断言)
- [重复操作](#重复操作)
  - [如何录制宏（Macro）？](#如何录制宏macro)

# 编辑技巧
## 如何改变大小写？
1. 可视化模式选中目标。
2. `gu`：将目标变为小写；`gU`：将目标变为大写。

# 替换技巧
## 如何实现零宽断言？
Vim 的零宽断言规范与常用正则规范不同，具体如下：
|     |前向|后向|
|:---:|:---:|:---:|
|**肯定**| `(atom)\@<=` | `(atom)\@=` |
|**否定**| `(atom)\@<!` | `(atom)\@!` |

* 注意
  * 在实际应用中，`(atom)` 左右的括号需要加反斜杠转义，即最终形式 `\(atom\)`，原因是 Vim 默认将括号等计为常规字符，转义后才具有功能特性。

* 参考
  * [Regex lookahead and lookbehind](https://vim.fandom.com/wiki/Regex_lookahead_and_lookbehind)

# 重复操作
## 如何录制宏（Macro）？
流程示例：
  ```
  qa    // 开始录制，其中 a 代表寄存器序号，可选值 [a-zA-Z0-9]
  ...   // 录制中：一系列实际操作
  q     // 结束录制
  @a    // 运行一次 a 寄存器内宏操作
  @@    // 重复运行上次宏操作
  ```

* 参考
  * [Vim 重复操作的宏录制](https://www.cnblogs.com/ini_always/archive/2011/09/21/2184446.html)
  * [Vim Macros Tutorial](https://vim.fandom.com/wiki/Macros)
  * [Vim Help: complex-repeat](https://vimhelp.org/repeat.txt.html#complex-repeat)