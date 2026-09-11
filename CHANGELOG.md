# Changelog

All notable changes to Ember Core Components.

## [2.0.0] — 空间树存储迁移至 World 托管缓冲（免 Dispose）

### Breaking
- **`SpatialTree` 不再持有原生容器，也不再提供 `Dispose()` / `Initialize(Allocator)`。**
  树数据改存 World 托管 buffer（随 `World.Dispose()` 自动释放），组件本体只保留标量状态与 buffer 句柄。
  - 旧：`ref var tree = ref world.GetComponent<SpatialTree>(owner); ... tree.Dispose();`
  - 新：`if (world.TryGetSpatialTree(out var tree)) { tree.QuerySphere(center, r, ref buffer); }`
    退出时随 `ECSManager.Dispose()` 一并回收，无需任何释放调用。
- 树操作统一改经 `SpatialTreeView`（`world.GetSpatialTree()` / `world.TryGetSpatialTree`）：
  `Insert` / `Remove` / `Update` / `QueryAABB` / `QuerySphere` / `BeginTick` / `EndTick` / `Clear`。
- 实体映射由 `NativeParallelHashMap` 换为直索引稀疏映射（按 `Entity.Index` 直接寻址，
  元素内版本校验槽位复用），查找 O(1) 且常数更低。

### Added
- `SpatialTreeView`（树操作视图）与 `SpatialTreeExtensions`（`GetSpatialTree` / `TryGetSpatialTree` / `EnsureSpatialTree`）。
- 新增实体槽复用防护测试（旧 Index + 新 Version 不得命中旧映射）。

### Fixed
- 修复 `Subdivide` 中元素计数双重递减（`Unlink` 已递减后又手动递减），
  计数漂移会干扰叶容量判断与空块收缩。

### Changed
- `SpatialIndexSystem` 不再需要 `Allocator.Persistent` 初始化，首 tick 自动建树。
- 测试扩充至 52 项：CLI 25 项通过；27 项原生容器测试在 Unity Test Runner 中执行。

## [1.0.0] — 空间索引、视锥剔除与 GameObject 表现层

### Breaking
- 移除静态辅助 `LocalTransform.Identity` / `LocalTransform.FromPosition` / `LocalToWorld.Identity` / `LocalToWorld.Compose`（静态禁令）。迁移：直接构造 `new LocalTransform(position, quaternion.identity, 1f)`；层级组合用 `math.mul(parent.Value, local.ToMatrix())`。

### Added
- **空间索引**：`BoundingVolume` / `WorldBounds` 组件；`SpatialTree` 纯非托管单例组件（四叉/八叉统一，`NativeList`/`NativeParallelHashMap` 存储，稳态 0GC），`QueryAABB`/`QuerySphere` 填充调用方 `NativeList<Entity>`；消失实体经标记清扫剔除（延迟一帧）；`SpatialIndexConfig` 单例配置维度/根范围/深度/容量。
- **视锥剔除**：`CameraFrustum` 单例（桥接代码每帧写入）、`VisibilityState`（bit0=当前帧、bit1=上一帧，`EnteredView`/`ExitedView` 边沿属性）、`InView` 标签（仅边沿增删）；`FrustumMath` 纯数学（Gribb-Hartmann 平面提取、球/AABB 测试、世界盒换算）；`WorldBoundsSystem` 与 `FrustumCullingSystem` 为 Burst Job。
- **系统组**：`SpatialSystemGroup` 一行接入整条空间/剔除管线（补齐→世界包围盒→剔除→标签应用→空间索引）。
- **GameObject 表现层**：`PresentationPrefab`（预制体 Id）/ `PresentationLink` / `PresentationCommands` 单例命令通道；视口边沿驱动 Spawn/Despawn，销毁实体盖戳清扫（回收延迟一帧）；`PresentationSyncSystem`（Job·Burst）并行写 TRS 同步槽位；托管桥 `GameObjectPresentation` drain 命令并经 `TransformAccessArray` + `IJobParallelForTransform`（Burst）批量回写 GameObject Transform；`IGameObjectPool` 支持业务侧池注入，`GameObjectPool` 为默认池实现（分桶栈 + Prewarm 预热）。
- **显式 Burst 策略**：程序集级 `EmberJobCompilationMode.Burst`，作业 Burst 编译显式可见；新增 `com.unity.burst` 1.8.13 包依赖。

### Changed
- 测试扩充至 51 项：CLI 25 项通过；26 项依赖 Unity 原生容器/引擎 API 的测试在 Unity Test Runner 中执行。

## [0.1.0] — 首次发布

### Added
- **空间组件**：`LocalTransform`（位置/旋转/等比缩放，含 `Identity`、`ToMatrix`、`TransformPoint` 等数学辅助）与 `LocalToWorld`（世界矩阵，含 `Compose` 层级组合与坐标轴访问）。
- **运动组件**：`LinearVelocity`（米/秒）与 `AngularVelocity`（轴角向量，弧度/秒）。
- **时间组件**：`Lifetime`（剩余存活秒数）、`Age`（已存活秒数）与 `WorldTime` 单例（`TimeScale` / `DeltaTime` / `UnscaledDeltaTime` / `ElapsedTime` / `FrameCount`）。
- **状态标记**：`Disabled` / `Static` / `Prefab` 三个 Tag 组件及配套查询约定（`None<Disabled>` 等）。
- **随机组件**：`GlobalRandom` 单例，基于 `Unity.Mathematics.Random` 的确定性随机源。
- 全部组件为 unmanaged struct，兼容 Burst 编译的 Job 系统；由 Ember 源生成器自动注册。
- NUnit 测试项目：纯数学与确定性测试 11 项；依赖 Unity 原生容器的 World 集成测试 5 项（CLI 下自动跳过，Unity Test Runner 中执行）。
