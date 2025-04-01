---
title: Python
date: 2024-12-3 15:25:00
tags:
  - Python
categories: [Python]
description: Python简单介绍
---

### Python初探

#### [f-strings](https://www.geeksforgeeks.org/formatted-string-literals-f-strings-python/)

```python
# Python3 program introducing f-string
val = 'Geeks'
print(f"{val}for{val} is a portal for {val}.")

# 三引号可以用来创建多行字符串
message = f"""
姓名：{name}
职业：{profession}
语言：{language}
"""

# 引号有一个问题
# To use any type of quotation marks with the f-string in Python we have to make sure that the quotation marks used inside the expression are not the same as quotation marks used with the f-string.

print(f"'GeeksforGeeks'")
# 想要正确输出，就要用双引号包含单引号，或者单引号包含双引号

Geek = { 'Id': 100,
         'Name': 'Om'}

print(f"Id of {Geek['Name']} is {Geek['Id']}")
```

#### [openpyxl](https://openpyxl.readthedocs.io/en/stable/tutorial.html)

<!--
This document provides a brief introduction to the openpyxl library.
openpyxl is a Python library used for reading and writing Excel 2010 xlsx/xlsm/xltx/xltm files.
It allows you to create, modify, and extract data from Excel files in a programmatic way.
-->
<!-- 简单介绍一下openpyxl -->


