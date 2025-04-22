---
title: Games104
date: 2025-04-22
tags:
  - games
categories: []
description: A collection of notes and resources for the Games104 course.
---

# 一、游戏引擎架构概述

现代游戏引擎普遍采用分层架构设计，这种设计方法的核心在于将复杂的引擎系统分解为多个相互独立但又协同工作的层次，从而实现高效的开发与便捷的维护。每个层次都专注于特定的职责，底层为上层提供必要的基础服务，而上层则依赖于底层的功能，但这种依赖是单向的，即上层不会直接调用下层的实现细节。

这种架构模式赋予游戏引擎以下优势：

- **可扩展性**：添加新功能或修改现有功能时，对其他层次的影响可最小化。
- **可维护性**：开发人员能更清晰地理解和修改特定模块的代码。
- **跨平台能力**：通过将平台相关代码隔离在底层，为跨平台奠定基础。

# 二、游戏引擎五层架构详解

## 2.1 平台层（Platform Layer）

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
   **代码示例**：

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

3. **适配输入设备和 CPU 架构**  
   - 统一处理键盘、鼠标、手柄等输入设备的差异，向上层提供统一输入事件流。  
   - 针对不同 CPU 架构（如 x86、ARM）优化，利用指令集或并行计算提升性能。

### 关键技术

- **RHI 实现**：为每个平台开发 RHI 具体实现，映射抽象接口到底层 API，保证上层代码跨平台性。  
- **跨平台编译工具链**：使用 CMake、Premake 等工具，通过统一配置文件生成不同平台的构建文件，简化编译流程。

## 2.2 核心层（Core Layer）

### 定义

核心层是引擎的基石，提供底层基础设施和服务，代码平台无关，专注于通用功能，如“瑞士军刀”般为上层模块提供工具。

### 核心功能

1. **内存管理**  
   实现自定义内存分配器，如池分配和对象池，减少内存碎片和分配开销。  
   **代码示例（C++ 对象池）**：

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

2. **容器与数据结构**  
   实现高效容器（如动态数组、哈希表）和数学库（向量、矩阵运算），针对引擎需求优化性能。

3. **多线程与任务调度**  
   - **线程池**：管理预先创建的线程，避免频繁创建/销毁开销。  
   - **异步加载**：后台线程加载资源，防止主线程阻塞。  
   - **Job 系统**：分解任务为可并行作业，提升多核处理器利用率。

### 关键技术

- **内存管理示例**：对象池通过重用空闲对象减少动态分配次数。  
- **线程池与 Job 系统**：利用无锁队列和任务依赖关系，实现并行任务调度，提升性能。

## 2.3 资源层（Resource Layer）

### 定义

资源层负责管理游戏资源（纹理、模型、音频等），涵盖加载、卸载、缓存及关联管理，确保高效访问数据。

### 核心功能

1. **资源管线**  
   将原始资源（如 .max、.psd）转换为引擎专用格式（如 .dds 纹理、二进制网格），优化文件大小和加载速度。

2. **资产管理**  
   - **GUID**：为资源分配唯一标识符（128 位数字），避免路径依赖冲突。  
   - **Handle**：通过句柄间接引用资源，解耦路径依赖，支持资源重命名/移动。

3. **生命周期管理**  
   - **延迟加载**：仅在需要时加载资源（如玩家进入特定区域时）。  
   - **引用计数与垃圾回收**：跟踪资源引用，自动释放不再使用的资源。

### 关键技术

- **GUID 生成**：使用 Python 的 `uuid` 库生成全局唯一标识符。  
- **资源加载器设计**：缓存已加载资源，处理不同类型资源的加载逻辑（如 GPU 可识别的纹理格式）。

## 2.4 功能层（Function Layer）

### 定义

功能层实现游戏核心玩法逻辑，划分为 **逻辑层（Tick Logic）** 和 **渲染层（Tick Render）**：
- 逻辑层处理游戏规则、状态更新、物理模拟、AI 等。  
- 渲染层将逻辑状态可视化，实现渲染管线（裁剪、光照、阴影等）。

### 核心功能

#### 逻辑层（Tick Logic）

- **核心职责**：处理角色移动、战斗逻辑、输入响应等，以固定时间间隔（Tick）更新。  
- **性能挑战与解决方案**：  
  - **Tick 时间过长**：采用固定时间步长补偿（如 Unity 的 FixedUpdate），将物理模拟与渲染帧率解耦。  
  - **突发负载**：延迟处理高耗时任务（如分帧处理爆炸事件中的物体逻辑）。

#### 渲染层（Tick Render）

- **核心职责**：将逻辑层状态渲染为图像，处理裁剪、光照、阴影等。  
- **同步问题与解决方案**：  
  - **线程同步**：使用双缓冲与事件栅栏（Fence）确保逻辑与渲染线程数据一致。  
  - **预测渲染**：根据历史数据预测下一帧状态，降低输入延迟（如射击游戏弹道预测）。

### 关键技术

- **Tick 驱动机制**：主循环按固定频率执行逻辑更新（Tick）和渲染更新。  
- **逻辑与渲染分离**：解耦代码，便于独立优化（如分别调优 AI 算法和绘制调用）。

## 2.5 工具层（Tool Layer）

### 定义

工具层提供编辑器和辅助工具，提升开发效率，为开发者和美术师提供友好的工作环境。

### 核心功能

1. **编辑器**  
   - **关卡编辑器**：可视化设计场景，摆放对象。  
   - **材质编辑器**：调整材质属性（颜色、光泽度等）。  
   - **动画编辑器**：创建角色动画，支持实时预览。

2. **资产管线**  
   通过 DCC 工具插件（如 3ds Max、Maya）实现资源导入/导出，转换为引擎格式。

3. **脚本系统**  
   支持蓝图可视化编程或 Python/C# 脚本，扩展引擎功能。

### 关键技术

- **编辑器架构**：基于 GUI 框架（如 Qt）构建，代码示例：

  ```python
  # 使用 PyQt 的简单关卡编辑器
  class LevelEditor(QMainWindow):
      def __init__(self):
          super().__init__()
          self.scene_graph = SceneGraphWidget()
          self.toolbar = QToolBar()
          self.save_btn = QAction("Save", self)
          self.toolbar.addAction(self.save_btn)
  ```

- **资产导入管线**：解析文件格式，优化资源（如压缩、生成 Mipmap），生成元数据。

# 三、关键概念与技术解析

## 3.1 ECS 架构（Entity-Component-System）

### 定义

ECS 是数据驱动的设计模式，核心概念：
- **实体（Entity）**：唯一标识符，无数据/行为，如容器或 ID。  
- **组件（Component）**：附加数据（如位置、血量），无逻辑。  
- **系统（System）**：处理特定组件的逻辑（如移动系统更新含位置和速度组件的实体）。

### 优势

- **数据与逻辑分离**：修改数据或逻辑不影响其他部分。  
- **内存访问高效**：同类组件连续存储，提升 CPU 缓存命中率。  
- **易于扩展**：新增功能只需创建新组件和系统，符合开闭原则。

### 实现对比

| **传统组件模式（OOP）**         | **ECS 架构**                          |
|----------------------------------|--------------------------------------|
| `player.GetComponent<Health>().TakeDamage();` | 系统遍历所有含 Health 组件的实体执行逻辑 |
| 组件间直接调用接口               | 仅通过事件总线或全局状态通讯         |
| 单次组件查询开销高               | 批量处理减少查询开销                 |

**代码示例**：

```cpp
// 实体
class Entity {
public:
    std::unordered_map<std::type_index, ComponentID> components;
};

// 组件
struct PositionComponent { float x, y; };

// 系统
class MovementSystem {
public:
    void update(EntityManager& entities) {
        for (auto& [entity, position] : entities.view<PositionComponent>()) {
            // 更新位置逻辑
        }
    }
};
```

## 3.2 多线程与 Job 系统

### 挑战

单线程处理复杂逻辑易卡顿，多核处理器需并行任务，但线程同步和竞态条件（Race Conditions）是主要挑战。

### 解决方案

- **任务拆分**：将物理模拟、资源加载等分配到不同线程。  
- **Job 系统**：分解任务为独立作业，通过无锁队列调度到多核执行，自动处理依赖关系。  

**代码示例（C++ Job 系统伪代码）**：

```cpp
class JobSystem {
public:
    void enqueue_job(std::function<void()> job) {
        std::lock_guard<std::mutex> lock(queue_mutex);
        jobs.push(std::move(job));
        condition.notify_one();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> jobs;
    std::mutex queue_mutex;
    std::condition_variable condition;
};
```

## 3.3 跨平台兼容性

### 常见问题

- **文件路径差异**：Windows 使用 `\`，macOS/Linux 使用 `/`。  
- **图形 API 差异**：各平台支持不同 API（如 DirectX、Metal、Vulkan）。

### 解决方法

- **统一路径处理**：提供平台无关的路径组合函数（如 `Path::combine("assets", "models", "character.fbx")`）。  
- **RHI 抽象图形 API**：上层代码面向 RHI 接口，底层根据平台选择具体 API。  
- **预编译宏**：使用 `#ifdef _WIN32` 等宏包裹平台特定代码。

# 四、开发实践要点

## 4.1 资源管理优化

- **延迟加载**：仅加载当前区域资源，动态卸载不再需要的资源，减少初始加载时间和内存占用。  
- **内存池技术**：预分配内存块，快速分配/回收频繁创建的对象（如粒子、子弹），减少碎片和 GC 压力。  
- **资源压缩**：使用 BC 格式压缩纹理、简化网格降低模型复杂度、压缩音频文件，减小安装包和内存占用。

## 4.2 工具链集成

- **DCC 插件开发**：开发 3ds Max、Maya 插件，支持直接导出引擎格式资源并生成元数据，提升协作效率。  
- **热更新机制**：运行时同步编辑器修改（如调整关卡对象位置），无需重启引擎，加快迭代速度。

## 4.3 性能调优

- **逻辑与渲染帧率解耦**：逻辑层固定帧率（如 60Hz），渲染层根据硬件性能动态调整，保证稳定性和流畅度。  
- **GPU 驱动优化**：提前编译 Shader 避免运行时卡顿，针对 NVIDIA/AMD GPU 驱动特性优化渲染性能。



