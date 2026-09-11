# Ember Core Components Manual

[中文](README.md)

Foundational component library for the Ember ECS framework. Provides spatial transforms,
motion, timing, state tags, and deterministic random as unmanaged structs stored directly in
chunk columns and compatible with Burst-compiled job systems. Components are registered
automatically at compile time by the Ember source generator — no manual registration.

## Installation

This package depends on `com.ember.ecs` (the Ember ECS framework). Install both via the
Unity Package Manager:

1. Open your project's `Packages/manifest.json`
2. Add the dependencies (use the actual released versions):

```json
{
  "dependencies": {
    "com.ember.ecs": "1.10.1",
    "com.ember.core": "0.1.0"
  }
}
```

3. When installing via Git URL: UPM does not resolve transitive Git dependencies, so make sure
   `com.ember.ecs` is declared explicitly in the manifest.
4. Save and let Unity resolve; `Unity.Mathematics` is pulled in transitively.

Requires Unity 2022.3 or newer.

## Component Reference

All components live in the `Ember.Core` namespace. Entity hierarchy (parent/child) is provided
by the framework's built-in `ParentComponent` / `ChildEntity` and is intentionally not
duplicated here.

### Spatial

| Component | Kind | Description |
|---|---|---|
| `LocalTransform` | Data | Local transform: `Position` (float3) / `Rotation` (quaternion) / `Scale` (uniform float). Acts as world transform when the entity has no parent. Helpers: `Identity`, `FromPosition`, `ToMatrix()`, `TransformPoint`, `InverseTransformPoint`, `TransformDirection` |
| `LocalToWorld` | Data | Local-to-world `float4x4` matrix. Helpers: `Identity`, `Position` / `Right` / `Up` / `Forward`, `TransformPoint`, `Compose(local, parent)` |

### Motion

| Component | Kind | Description |
|---|---|---|
| `LinearVelocity` | Data | Linear velocity `Value` (float3) in meters/second, integrated into `LocalTransform.Position` by movement systems |
| `AngularVelocity` | Data | Angular velocity `Value` (float3) as an axis-angle vector: direction = rotation axis, magnitude = radians/second |

### Timing

| Component | Kind | Description |
|---|---|---|
| `Lifetime` | Data | Remaining seconds `Remaining`, counted down by a lifetime system; entity is destroyed at zero (bullets, VFX, etc.) |
| `Age` | Data | Elapsed seconds `Elapsed` since creation; useful for fade in/out interpolation and growth stages |
| `WorldTime` | Singleton | Global time state: `TimeScale` / `DeltaTime` / `UnscaledDeltaTime` / `ElapsedTime` / `FrameCount`. Read it from jobs or anywhere `SystemContext.DeltaTime` is unavailable |

### State Tags

| Component | Kind | Description |
|---|---|---|
| `Disabled` | Tag | Disabled marker. Convention: systems add `None<Disabled>` to skip disabled entities; cheaper than destruction for temporary deactivation |
| `Static` | Tag | Static marker. Convention: movement/transform systems add `None<Static>` to skip immovable entities, saving integration and matrix recomputation |
| `Prefab` | Tag | Prefab template marker. Template entities stay out of regular logic (queries add `None<Prefab>`) and serve only as instantiation sources |

### Random

| Component | Kind | Description |
|---|---|---|
| `GlobalRandom` | Singleton | Global deterministic random source (`Unity.Mathematics.Random`). It is a value type whose state advances per draw — access it by `ref`. For parallel random streams, give each entity its own random component instead of sharing this singleton |

## Usage Example

```csharp
using Ember;
using Ember.Core;
using Unity.Mathematics;

public sealed class SpawnSystem : SystemBase
{
    protected override void DeclareAccess(AccessBuilder access)
    {
        access.Write<LocalTransform>().Write<LinearVelocity>().Write<Lifetime>()
              .StructuralChanges();
    }

    protected override void OnTick(SystemContext ctx)
    {
        var entity = ctx.ECB.CreateEntity(new ComponentMask());
        ctx.ECB.AddComponent(entity, LocalTransform.FromPosition(new float3(0f, 1f, 0f)));
        ctx.ECB.AddComponent(entity, new LinearVelocity(new float3(0f, 0f, 5f)));
        ctx.ECB.AddComponent(entity, new Lifetime(3f));
    }
}
```

Reading the global time singleton:

```csharp
var owner = World.GetOrCreateSingleton<WorldTime>();
ref var time = ref World.GetComponent<WorldTime>(owner);
time.ElapsedTime += time.DeltaTime;
```

Movement query convention that skips disabled and static entities:

```csharp
var query = EntityQuery.With<LocalTransform, LinearVelocity>().None<Disabled, Static>();
```

## Component Design Conventions

1. Components are unmanaged structs implementing **exactly one** component interface:
   `IDataComponent` / `ITagComponent` / `ISingletonComponent` / `IBufferElement`.
2. Tag components carry no instance fields; they occupy archetype mask bits only.
3. Components hold data and pure math helpers only — all behavior lives in systems.
4. Component assemblies must be loaded before the first `World` is created; the registry
   seals afterwards.

## Testing

The source repository ships an NUnit test project under `tests/Ember.Core.Tests`. Pure math
and determinism tests run on any .NET environment; World integration tests that require
`Unity.Collections` native containers execute inside the Unity Test Runner and skip
automatically under a plain .NET CLI — the same convention as the Ember framework tests.
