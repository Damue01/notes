---
title: Lua
---

# Lua初探

## Lua简单介绍

Lua可以做动态下发，如果我们用C++写功能，那么每次更新都只能发新的大版本，但是Lua可以作为资产下发，也就是说玩家可以直接在系统内更新，简单点讲就是做热更新

## UI操作

_createWidget()中，创建UI

```lua
 local item_data = Idol.CUtility.CreateListViewItemData(function(widget, dataObj, userData)
  end, userData);
```

定义带.的就是static

UI上，如果有灵活变化background的需求，可以考虑直接放置一个canvas，然后选用不同的User Widget，这样子可以变化各种形状之类的

### **ListView**

通过 **「创建服装列表」** 的示例串联整个逻辑：

---

#### **1. UE ListView 触发 Entry 的核心机制**

##### **核心流程**

```plaintext
[数据层] C++/Lua 数据源 → [UI层] ListView 生成 Entry → [绑定层] Lua 回调配置控件
```

- **触发时机**：当调用 `UListView.AddItem()` 时，ListView 会根据 `EntryClass`（预设的 Widget 蓝图）自动生成或复用 `UIdol_ListViewEntry` 实例。
- **关键步骤**：
  1. **生成 Entry**：ListView 内部通过 `UUserWidgetPool` 创建或获取一个 `UIdol_ListViewEntry` 实例。
  2. **绑定数据**：调用 `NativeOnListItemObjectSet()`，将数据对象（`UIdol_ListViewItemData`）传递到 Entry。
  3. **触发 Lua 回调**：在 `NativeOnListItemObjectSet()` 中，通过 `call(data->mScriptCallback)` 调用 Lua 函数，最终配置控件。

---

#### **2. 代码流程串联（以服装列表为例）**

##### **示例代码分解**

```lua
-- 步骤1: 获取服装配置表和数据
local cloth_table = Idol.Configs.Suit;
local cloth_list_view = IdolAPI.Call("UIdol_BlueprintFunctionLibrary_Game.FindWidget", gInstance.mUserWidgetCache.mWidget, "ListView_Cloth");

-- 步骤2: 清空列表
IdolAPI.Call("UListView.ClearListItems", cloth_list_view);

-- 步骤3: 遍历数据，动态创建列表项
for key, value in pairs(cloth_table) do
    -- 步骤3.1: 创建 ListViewItemData，绑定 Lua 回调
    local item_data = Idol.CUtility.CreateListViewItemData(
        function(widget, dataObj, userData)
            -- 步骤4: Lua 回调配置 Entry 控件
            local name = IdolAPI.Call("UKismetSystemLibrary.GetObjectName", widget);
            Idol.CUtility.Log("------------widget name: ", name);

            local text_cloth = IdolAPI.Call("UIdol_BlueprintFunctionLibrary_Game.FindWidget", widget, "Text_Cloth");
            IdolAPI.Call("UTextBlock.SetText", text_cloth, value.Name);
        end, 
        nil
    );

    -- 步骤3.2: 将数据项添加到 ListView
    IdolAPI.Call("UListView.AddItem", cloth_list_view, item_data);
end
```

---

#### **3. 关键步骤交互细节**

##### **步骤3.1: CreateListViewItemData**

- **Lua 层**：`CreateListViewItemData` 封装一个 Lua 回调函数，用于后续配置 Entry。
- **C++ 层**：
  - 生成 `UIdol_ListViewItemData` 对象，将 Lua 回调序列化为 `mScriptCallback`（如 `"function_1234"` 的持久化引用）。
  - **重要细节**：通过 `CUtility.CreatePersistentCallback` 确保 Lua 函数不会被 GC。

##### **步骤3.2: AddItem**

- **C++ 层**：ListView 收到 `AddItem` 调用后：
  1. 将 `item_data`（`UIdol_ListViewItemData*`）加入数据源列表。
  2. 触发 Entry 生成逻辑：
     - 若 EntryPool 无可用实例，根据 `EntryClass` 创建新的 `UIdol_ListViewEntry`。
     - 调用 `NativeOnListItemObjectSet(ListItemObject)`，传入 `item_data`。

##### **步骤4: Lua 回调触发**

- **C++ 层**：在 `NativeOnListItemObjectSet` 中：

  ```cpp
  void UIdol_ListViewEntry::NativeOnListItemObjectSet(UObject* ListItemObject) {
      auto data = static_cast<UIdol_ListViewItemData*>(ListItemObject);
      // 构造 Lua 参数：mEntry=当前Entry实例，mData=绑定的数据
      LuaTable table_wrapper;
      table_wrapper->set("mEntry", this);  // this 是 UIdol_ListViewEntry*
      table_wrapper->set("mData", data);
      // 调用 Lua 回调（data->mScriptCallback 即步骤3.1保存的函数）
      LuaState->call(data->mScriptCallback, table_wrapper);
  }
  ```

- **Lua 层**：回调函数被调用，参数解析：
  - `widget`: 对应 `mEntry`，即当前 Entry 的 UserWidget 实例。
  - `dataObj`: 对应 `mData`，即 `item_data`（此处未使用）。
  - `userData`: 创建时传入的额外数据（此处为 `nil`）。

---

#### **4. 参数与数据流动图解**

```plaintext
Lua 数据层: cloth_table (服装表)
  │
  │ for循环迭代 value = { Name="服装A", ... }
  ▼
C++ 数据层: UIdol_ListViewItemData (item_data)
  │   ▲
  │   │ AddItem(item_data)
  ▼   │
C++ UI层: ListView 生成 Entry (UIdol_ListViewEntry)
  │   ▲
  │   │ NativeOnListItemObjectSet(item_data)
  ▼   │
Lua 回调层: function(widget, dataObj, userData)
         │
         │ FindWidget(widget, "Text_Cloth") → 操作 TextBlock
         ▼
       UI显示: Text_Cloth 显示 "服装A"
```

---

#### **5. 关键问题解答**

##### **Q1: `widget` 如何对应到正确的 Entry？**

- **机制**：ListView 为每个数据项生成独立的 Entry 实例，`NativeOnListItemObjectSet` 在设置数据时，通过 `mEntry` 传递当前 Entry 的实例到 Lua。

##### **Q2: `value.Name` 为何能正确捕获？**

- **闭包特性**：Lua 回调在 `for key, value in pairs(cloth_table)` 循环中创建，闭包捕获了当前迭代的 `value`，即使回调在后续渲染时触发，仍能访问正确的值。

##### **Q3: 为何要调用 `FindWidget` 查找子控件？**

- **控件嵌套**：`UIdol_ListViewEntry` 可能是一个复合 Widget（如包含 Text_Cloth、Image_Icon 等子控件），需通过名称查找具体控件实例。

##### **Q4: 为什么在`CreateListViewItemData`的时候就传入`widget`，此时如何知道对应哪个类型的？**

- **数据绑定**：这个时候其实并不知道该widget对应的类型，里面的函数是假定我们知道其对应衣服entry的蓝图的，但是为什么能正常运行呢？因为只做这一步并不会在显示时运行，只有在add以后，设置了对应的蓝图和类型，才将我们设定的蓝图作为entry设置到了对应的listview上，这样，该listview显示的时候就知道对应的entry是什么类型了，从而选到正确的widget

---

通过以上流程，你可以清晰地看到从 Lua 数据配置到 C++ 控件生成，再回到 Lua 动态配置 UI 的完整链路。

初始化时，函数根据Count创建对应数量的ListViewItemData，填充到ListView里，进入待命状态。（创建出所有行了，但是没有显示）
只有在列表项滚动到可见区域时，UE才会调用对应的ItemData的回调函数，生成和更新列表现里的内容。（一部分列表现可见，可见的部分去调用绑定的回调函数，参数在创建时候也存入了？）
CreateListViewItemData(function(widget, dataObj, userData) end, nil);中的function就是回调函数，参数的含义为：
widget: 列表项的ItemData，每一行是一个Item
这里的additem其实就是将创建出来的itemdata绑定到对应的listiew里，这样子对应的listview里有可见部分时，就知道调用哪个回调函数了

这里手动加载class类和卸载class类，因为这个类是蓝图资源，所以我们要手动去找到

```lua
  local cls = IdolAPI.Call("UIdol_BlueprintFunctionLibrary_Game.ReferenceResource", v.mConfig.AssetPath);
  self.mAttachments[k] = IdolAPI.Call("AActor.AddComponentByClass", self.mRole, cls, false, Idol.CUtility.ConstructionCPPTransform(), false);
  IdolAPI.Call("UIdol_BlueprintFunctionLibrary_Game.ReleaseResource", v.mConfig.AssetPath);
```

#### 调试操作

ZeroBraneStudio调试完成后不要直接点他的结束进程，这样子UE端会收不到回包，导致卡死
