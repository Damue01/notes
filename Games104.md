---
title: Games104
last_updated: 2025-04-22
tags:
  - games
categories: []
description: A collection of notes and resources for the Games104 course.
---

# 游戏引擎架构概述

现代游戏引擎普遍采用分层架构设计，这种设计方法的核心在于将复杂的引擎系统分解为多个相互独立但又协同工作的层次，从而实现高效的开发与便捷的维护。每个层次都专注于特定的职责，底层为上层提供必要的基础服务，而上层则依赖于底层的功能，但这种依赖是单向的，即上层不会直接调用下层的实现细节。

## 架构优势

- **可扩展性**：添加新功能或修改现有功能时，对其他层次的影响可最小化。
- **可维护性**：开发人员能更清晰地理解和修改特定模块的代码。
- **跨平台能力**：通过将平台相关代码隔离在底层，为跨平台奠定基础。

# 游戏引擎五层架构详解

## 平台层（Platform Layer）

### 定义

平台层是不同硬件设备（如 PC、游戏主机、移动设备）和软件平台（如 Windows、macOS、Android、iOS 及应用商店）之间的桥梁。它通过统一接口屏蔽底层差异，使上层模块无需关心平台细节，是实现跨平台兼容性的关键。

### 核心功能

1. **处理文件系统差异**  
   不同操作系统的文件路径格式、存储方式和权限管理存在差异。例如：
   - Windows 使用反斜杠 `\` 作为路径分隔符。
   - macOS 和 Linux 使用正斜杠 `/`。  
   平台层需统一处理，为上层提供一致的文件操作接口。

2. **封装图形 API**  
   不同平台支持不同图形 API（如 Windows 的 DirectX、macOS 的 Metal、跨平台的 Vulkan）。平台层通过 **RHI（渲染硬件接口）** 封装底层 API，定义抽象渲染接口，上层面向 RHI 开发，平台层负责具体实现。

### 示例代码

```python
# 抽象 RHI 接口
class RHI:
    def init_device(self):
        raise NotImplementedError

    def draw_frame(self):
        raise NotImplementedError

# Windows 平台 DirectX 实现
class DirectXRHI(RHI):
    def init_device(self):
        # 初始化 DirectX 设备
        pass

    def draw_frame(self):
        # 调用 DirectX 渲染逻辑
        pass
```

## 核心层（Core Layer）

### 定义

核心层是引擎的基石，提供底层基础设施和服务，代码平台无关，专注于通用功能，如“瑞士军刀”般为上层模块提供工具。

### 核心功能

- **内存管理**：实现自定义内存分配器，如池分配和对象池，减少内存碎片和分配开销。
- **容器与数据结构**：实现高效容器（如动态数组、哈希表）和数学库（向量、矩阵运算），针对引擎需求优化性能。
- **多线程与任务调度**：通过线程池和 Job 系统提升多核处理器利用率。

### 示例代码

```cpp
template <typename T>
class ObjectPool {
public:
    T* allocate() {
        if (free_list.empty()) {
            return new T();
        }
        T* obj = free_list.back();
        free_list.pop_back();
        return obj;
    }

    void deallocate(T* obj) {
        free_list.push_back(obj);
    }
private:
    std::vector<T*> free_list;
};
```

## 资源层（Resource Layer）

### 定义

资源层负责管理游戏资源（纹理、模型、音频等），涵盖加载、卸载、缓存及关联管理，确保高效访问数据。

### 核心功能

- **资源管线**：将原始资源转换为引擎专用格式，优化文件大小和加载速度。
- **资产管理**：通过 GUID 和 Handle 解耦路径依赖，支持资源重命名/移动。
- **生命周期管理**：延迟加载和引用计数机制确保资源高效使用。

## 功能层（Function Layer）

### 定义

功能层实现游戏核心玩法逻辑，划分为逻辑层和渲染层：
- **逻辑层**：处理游戏规则、状态更新、物理模拟、AI 等。
- **渲染层**：将逻辑状态可视化，实现渲染管线（裁剪、光照、阴影等）。

### 核心技术

- **Tick 驱动机制**：主循环按固定频率执行逻辑更新和渲染更新。
- **逻辑与渲染分离**：解耦代码，便于独立优化。

## 工具层（Tool Layer）

### 定义

工具层提供编辑器和辅助工具，提升开发效率，为开发者和美术师提供友好的工作环境。

### 核心功能

- **编辑器**：关卡编辑器、材质编辑器、动画编辑器等。
- **资产管线**：通过 DCC 工具插件实现资源导入/导出。
- **脚本系统**：支持蓝图可视化编程或 Python/C# 脚本扩展引擎功能。

# 关键技术解析

## ECS 架构（Entity-Component-System）

### 定义

ECS 是数据驱动的设计模式，核心概念包括实体、组件和系统。

### 优势

- 数据与逻辑分离，易于扩展。
- 内存访问高效，提升 CPU 缓存命中率。

### 示例代码

```cpp
class Entity {
public:
    std::unordered_map<std::type_index, ComponentID> components;
};

struct PositionComponent { float x, y; };

class MovementSystem {
public:
    void update(EntityManager& entities) {
        for (auto& [entity, position] : entities.view<PositionComponent>()) {
            // 更新位置逻辑
        }
    }
};
```

# 总结

游戏引擎的分层架构通过职责分离实现高效开发、便捷维护和跨平台能力。掌握 RHI、ECS、Job 系统等关键技术，结合资源管理、多线程、跨平台实践，是构建高性能引擎的关键。