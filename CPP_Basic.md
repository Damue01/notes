---
title: C++Basic
date: 2024-11-22 14:32:29
tags:
  - C++
categories: [C++]
description: C++ 基础知识点、八股和个人使用经验
---

## C++初探

### mutable关键字

`mutable` 关键字可以用来修饰类的成员变量，使其在常量成员函数中可以被修改。

### .和->

> Accessing Data Members and Member Functions
> 访问数据成员和成员函数
> The data members and member functions of the class can be accessed using the dot(‘.’) operator with the object. For example, if the name of the object is obj and you want to access the member function with the name printName() then you will have to write:
> 可以使用对象的点('.') 运算符来访问类的数据成员和成员函数。例如，如果对象的名称是obj并且您想要访问名称为printName() 的成员函数，那么您必须编写：

```cpp
obj.printName()

```

---
你提到的问题涉及到 `shared_ptr` 和 `weak_ptr` 的引用计数机制，以及它们如何影响对象的生命周期。让我详细解释一下 `weak_ptr` 在这种场景下的行为。

### 引用计数机制

1. **`shared_ptr` 的引用计数**：
   - 每个 `shared_ptr` 都会增加它所指向对象的引用计数。
   - 当引用计数降为零时，对象会被销毁。

2. **`weak_ptr` 的引用计数**：
   - `weak_ptr` 不会增加它所指向对象的引用计数。
   - `weak_ptr` 只是“弱”地观察对象的存在，不会阻止对象被销毁。
   - `weak_ptr` 有一个内部的“弱指针计数”，用于管理 `weak_ptr` 的生命周期，但这个计数不影响 `shared_ptr` 的引用计数。

### 具体例子

假设我们有以下代码：

```cpp
class B;

class A {
public:
    std::weak_ptr<B> weakPtrToB;  // 使用 weak_ptr 打破循环引用
};

class B {
public:
    std::shared_ptr<A> ptrToA;
};

int main() {
    {
        std::shared_ptr<A> a = std::make_shared<A>();
        std::shared_ptr<B> b = std::make_shared<B>();

        a->weakPtrToB = b;  // 使用 weak_ptr
        b->ptrToA = a;

        // 在这个作用域结束时，a 和 b 都会被正确销毁
    }

    // 这里 a 和 b 已经被销毁，没有内存泄漏
}
```

#### 引用计数分析

- **外部持有的 `shared_ptr`**：
  - `std::shared_ptr<A> a` 和 `std::shared_ptr<B> b` 是外部持有的 `shared_ptr`，它们分别指向 `A` 和 `B` 对象。
  - 初始时，`a` 和 `b` 的引用计数均为 1（因为它们各自有一个外部持有的 `shared_ptr`）。

- **内部持有的 `shared_ptr` 和 `weak_ptr`**：
  - `b->ptrToA = a;`：这使得 `b` 内部的 `ptrToA` 持有一个指向 `a` 的 `shared_ptr`，因此 `a` 的引用计数变为 2（一个是外部的 `a`，另一个是 `b` 内部的 `ptrToA`）。
  - `a->weakPtrToB = b;`：这使得 `a` 内部的 `weakPtrToB` 指向 `b`，但不会增加 `b` 的引用计数。因此，`b` 的引用计数仍然是 1（只有外部的 `b`）。

#### 作用域结束时的行为

当 `main` 函数的作用域结束时：

- 外部的 `std::shared_ptr<A> a` 和 `std::shared_ptr<B> b` 被销毁。
- `b` 的引用计数从 1 降为 0，因此 `b` 对象被销毁。
- `a` 的引用计数从 2 降为 1（因为 `b` 内部的 `ptrToA` 也被销毁了），然后进一步降为 0（因为外部的 `a` 也被销毁了），因此 `a` 对象也被销毁。

### 关键点总结

- **`weak_ptr` 不增加引用计数**：`weak_ptr` 只是观察对象的存在，不会阻止对象被销毁。
- **`shared_ptr` 增加引用计数**：每个 `shared_ptr` 都会增加它所指向对象的引用计数，当引用计数降为零时，对象会被销毁。
- **打破循环引用**：通过使用 `weak_ptr`，可以避免两个对象互相持有对方的 `shared_ptr` 导致的循环引用问题，从而确保对象能够被正确销毁，避免内存泄漏。

### 回答你的具体问题

- **如果 `weak_ptr` 指向自己，自己的计数不会加一**：这是正确的。`weak_ptr` 不会影响 `shared_ptr` 的引用计数。
- **如果有两个 `shared_ptr` 指向同一个对象，它的计数是 2**：这也是正确的。每个 `shared_ptr` 都会增加引用计数。

通过这种方式，你可以更好地理解 `shared_ptr` 和 `weak_ptr` 的引用计数机制，以及它们在不同场景下的行为。

__vfptr: 虚函数表指针
