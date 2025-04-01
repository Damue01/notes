---
title: Code_Convention
date: 2025-2-20 16:53:12
tags:
  - Code Convention
categories: [Code, C++, lua]
description: 代码命名规范
---

## 代码命名规范

### 命名规范总览

| 元素类型 | 命名规则 | 示例 |
| --- | --- | --- |
| **类（Class）** | 以 `C` 开头，每个单词首字母大写 | `class CTestWork` |
| **结构体（Struct）（table）** | 每个单词首字母大写 | `struct TestWork` |
| **公有静态函数** | 每个单词首字母大写 | `static void TestWork()` |
| **受保护/私有静态函数** | 以 `_` 开头，每个单词首字母大写 | `static void _TestWork()` |
| **静态变量** | 以 `s` 开头，每个单词首字母大写 | `static float sTestWork` |
| **静态常量** | 全字母大写，单词用 `_` 分开 | `static const float TEST_WORK` |
| **成员变量** | 以 `m` 开头，每个单词首字母大写 | `float mTestWork` |
| **公有成员函数** | 首个单词首字母小写，后续单词首字母大写 | `void testWork()` |
| **受保护/私有成员函数** | 以 `_` 开头，首个单词首字母小写，后续单词首字母大写 | `void _testWork()` |
| **函数参数** | 首字母小写，后续单词首字母大写 | `void testWork(float paramArg0)` |
| **函数体内临时变量** | 全字母小写，单词用 `_` 分开 | `float test_param_0` |

静态变量 = Lua中函数体外的变量