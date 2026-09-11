# Changelog

All notable changes to Ember Core Components.

## [1.0.0] — Spatial Index, Frustum Culling & GameObject Presentation

### Breaking
- Removed static helpers `LocalTransform.Identity` / `LocalTransform.FromPosition` / `LocalToWorld.Identity` / `LocalToWorld.Compose` (no-statics rule). Migration: construct directly via `new LocalTransform(position, quaternion.identity, 1f)`; combine hierarchies with `math.mul(parent.Value, local.ToMatrix())`.

### Added
- **Spatial index**: `BoundingVolume` / `WorldBounds` components; `SpatialTree` as a fully unmanaged singleton component (unified quadtree/octree, backed by `NativeList`/`NativeParallelHashMap`, zero steady-state GC); `QueryAABB`/`QuerySphere` fill a caller-provided `NativeList<Entity>`; vanished entities are removed by mark-and-sweep (one frame latency); `SpatialIndexConfig` singleton configures dimension/root extent/depth/capacity.
- **Frustum culling**: `CameraFrustum` singleton (written per frame by bridge code), `VisibilityState` (bit0 = current frame, bit1 = previous frame, with `EnteredView`/`ExitedView` edge properties), `InView` tag (added/removed on edges only); `FrustumMath` pure math (Gribb-Hartmann plane extraction, sphere/AABB tests, world-bounds transform); `WorldBoundsSystem` and `FrustumCullingSystem` run as Burst jobs.
- **System groups**: `SpatialSystemGroup` wires the entire spatial/culling pipeline in one registration (setup → world bounds → culling → tag apply → spatial index).
- **GameObject presentation**: `PresentationPrefab` (prefab id) / `PresentationLink` / `PresentationCommands` singleton command channel; viewport edges drive Spawn/Despawn, destroyed entities are reclaimed via mark-and-sweep (one frame latency); `PresentationSyncSystem` (Burst job) writes TRS sync slots in parallel; the managed `GameObjectPresentation` bridge drains commands and applies transforms in batch via `TransformAccessArray` + `IJobParallelForTransform` (Burst); `IGameObjectPool` allows injecting a business-side pool, with `GameObjectPool` as the default implementation (bucketed stacks + Prewarm).
- **Explicit Burst policy**: assembly-level `EmberJobCompilationMode.Burst`; added `com.unity.burst` 1.8.13 package dependency.

### Changed
- Test suite grew to 51 tests: 25 pass on CLI; 26 tests requiring Unity native containers/engine APIs run in the Unity Test Runner.

## [0.1.0] — Initial Release

### Added
- **Spatial components**: `LocalTransform` (position/rotation/uniform scale with math helpers such as `Identity`, `ToMatrix`, `TransformPoint`) and `LocalToWorld` (world matrix with `Compose` hierarchy combination and axis accessors).
- **Motion components**: `LinearVelocity` (meters/second) and `AngularVelocity` (axis-angle vector, radians/second).
- **Timing components**: `Lifetime` (remaining seconds), `Age` (elapsed seconds), and the `WorldTime` singleton (`TimeScale` / `DeltaTime` / `UnscaledDeltaTime` / `ElapsedTime` / `FrameCount`).
- **State tags**: `Disabled` / `Static` / `Prefab` tag components with query conventions (`None<Disabled>`, etc.).
- **Random component**: `GlobalRandom` singleton, a deterministic random source based on `Unity.Mathematics.Random`.
- All components are unmanaged structs compatible with Burst-compiled job systems and are registered automatically by the Ember source generator.
- NUnit test project: 11 pure math/determinism tests plus 5 World integration tests requiring Unity native containers (auto-skipped on CLI, executed in the Unity Test Runner).
