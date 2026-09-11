# Changelog

All notable changes to Ember Core Components.

## [0.1.0] — 首次发布

### Added
- **空间组件**：`LocalTransform`（位置/旋转/等比缩放，含 `Identity`、`ToMatrix`、`TransformPoint` 等数学辅助）与 `LocalToWorld`（世界矩阵，含 `Compose` 层级组合与坐标轴访问）。
- **运动组件**：`LinearVelocity`（米/秒）与 `AngularVelocity`（轴角向量，弧度/秒）。
- **时间组件**：`Lifetime`（剩余存活秒数）、`Age`（已存活秒数）与 `WorldTime` 单例（`TimeScale` / `DeltaTime` / `UnscaledDeltaTime` / `ElapsedTime` / `FrameCount`）。
- **状态标记**：`Disabled` / `Static` / `Prefab` 三个 Tag 组件及配套查询约定（`None<Disabled>` 等）。
- **随机组件**：`GlobalRandom` 单例，基于 `Unity.Mathematics.Random` 的确定性随机源。
- 全部组件为 unmanaged struct，兼容 Burst 编译的 Job 系统；由 Ember 源生成器自动注册。
- NUnit 测试项目：纯数学与确定性测试 11 项；依赖 Unity 原生容器的 World 集成测试 5 项（CLI 下自动跳过，Unity Test Runner 中执行）。
