---
title: Singleton
date: 2024-11-22 11:07:15
tags:
    - UE
categories: [UE]
description: Singleton
---

## 单例模式

- 类只能有一个实例
- 必须自行创建这个实例，自己创建自己的唯一实例
- 必须自行向整个系统提供这个实例

### 为什么用单例

单例模式是一种设计模式，它确保一个类只有一个实例，并提供一个全局访问点。使用单例模式可以节省资源，避免频繁创建和销毁对象，特别适用于需要频繁访问的资源密集型对象。

### 如何实现单例

以下是实现单例模式的步骤和示例代码：

#### 新建UObject

   1. 新建一个C++文件，继承自UObject。
   2. 新建一个Actor用于测试能否正确调用该单例。

#### 示例代码

```C++
// MySingleton.h

#pragma once

#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MySingleton.generated.h"

/**
 * 单例类UMySingleton，继承自UObject
 */
UCLASS(Blueprintable)
class MYPROJECT_API UMySingleton : public UObject
{
    GENERATED_BODY()

public:
    // 构造函数
    UMySingleton();

    // 获取单例实例的静态方法
    UFUNCTION(BlueprintCallable, Category = "Singleton")
    static UMySingleton* GetInstance();

    // 测试调用方法
    UFUNCTION(BlueprintCallable, Category = "Singleton")
    void TestCall();

private:
    // 单例实例指针
    static UMySingleton* SingletonInstance;
};
```

```C++
// MySingleton.cpp

#include "MySingleton.h"

// 初始化单例实例指针
UMySingleton* UMySingleton::SingletonInstance = nullptr;

// 构造函数
UMySingleton::UMySingleton()
{
    // 构造函数逻辑
}

// 获取单例实例的静态方法
UMySingleton* UMySingleton::GetInstance()
{
    if (SingletonInstance == nullptr)
    {
        SingletonInstance = NewObject<UMySingleton>();
        SingletonInstance->AddToRoot(); // 防止垃圾回收
    }
    return SingletonInstance;
}

// 测试调用方法
void UMySingleton::TestCall()
{
    UE_LOG(LogTemp, Warning, TEXT("TestCall"));
}
```

#### 添加到GameInstance

```C++
// MyGameInstance.h

private:
    UPROPERTY(BlueprintCallable)
    class UMysingleton* GlobalSingleton;

// MyGameInstance.cpp

void UMyGameInstance::Init()
{
    Super::Init();
    GlobalSingleton = UMysingleton::GetInstance();
}
```

#### 优点

- 保证一个类只有一个实例
- 提供一个全局访问点
- 节省资源，避免频繁创建和销毁对象
- 适用于需要频繁访问的资源密集型对象
