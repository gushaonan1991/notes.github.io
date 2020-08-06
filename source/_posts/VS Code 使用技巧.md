---
title: VS Code 使用技巧
date: 2020-08-06
categories: [日常应用, 编辑工具]
tags: [VS Code, 技巧, 编辑, 工具]
---
- [修改默认语言支持](#修改默认语言支持)
  - [需求](#需求)
  - [方法](#方法)
  - [参考](#参考)

## 修改默认语言支持
### 需求
VS Code 默认将新建的匿名文件视为普通文本格式，我希望它能直接以 Markdown 格式解析。

### 方法
1. 临时方案：针对当前文件
   * 打开全局查询（F1）
   * 找到 `Change Language Mode` 选项
   * 选择目标格式 `Markdown`
   * 注意：打开 `Change Language Mode` 项的快捷键为 *cmd&k -> m*

2. 长期方案：修改默认配置
   * 打开 `User Settings`
   * 修改 `files.defaultLanguage` 项的值为 `markdown`

### 参考
* [Change VsCode default language](https://stackoverflow.com/questions/35904221/change-vscode-default-language-for-new-files)
