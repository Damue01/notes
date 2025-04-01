---
title: Lua_Logic
date: 
tags:
  - Lua
categories: []
description: 
---

## Lua

### Lua初始化

### 模块开发逻辑（以UI调用Lua逻辑为例）

1. 在`Idol_BlueprintFunctionLibrary.h`里，我们有定义`OnGenericEvent`函数

    ```cpp
    UFUNCTION(BlueprintCallable, Category = "Idol|UI")
    static void OnGenericEvent(UWidget* widget, const FString& luaFunction);

    void UUIdol_UIBlueprintFunctionLibrary::OnGenericEvent(UWidget* widget, const FString& luaFunction)
      {
          NSIdol::NSClient::LuaVariable variable(
              NSIdol::NSClient::NSLua::IAPI::Get()->queryInterface_ILuaState(), 
              NSIdol::NSClient::NSLua::IAPI::Get()->queryInterface_ILuaState()->createVariable(widget)
          );
          NSIdol::NSClient::NSLua::IAPI::Get()->queryInterface_ILuaState()->call(luaFunction, variable);
      }
    ```

2. 函数具体实现中，我们传参`widget`和`luaFunction`，其中`widget`是UI控件，`luaFunction`是Lua函数名，注意最后调用Lua的方式，是 `NSIdol::NSClient::NSLua::IAPI::Get()->queryInterface_ILuaState()->call();`。

3. 然后Lua里我们就去实现具体的逻辑，最后把完整的函数路径传给call。


#### 注意Logic里的初始化

#### Lua也会出现调用顺序的问题，比如说：A里调用B，结果B还没初始化好