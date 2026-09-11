# Changelog

All notable changes to Ember Core Components.

## [0.1.0] — Initial Release

### Added
- **Spatial components**: `LocalTransform` (position/rotation/uniform scale with math helpers such as `Identity`, `ToMatrix`, `TransformPoint`) and `LocalToWorld` (world matrix with `Compose` hierarchy combination and axis accessors).
- **Motion components**: `LinearVelocity` (meters/second) and `AngularVelocity` (axis-angle vector, radians/second).
- **Timing components**: `Lifetime` (remaining seconds), `Age` (elapsed seconds), and the `WorldTime` singleton (`TimeScale` / `DeltaTime` / `UnscaledDeltaTime` / `ElapsedTime` / `FrameCount`).
- **State tags**: `Disabled` / `Static` / `Prefab` tag components with query conventions (`None<Disabled>`, etc.).
- **Random component**: `GlobalRandom` singleton, a deterministic random source based on `Unity.Mathematics.Random`.
- All components are unmanaged structs compatible with Burst-compiled job systems and are registered automatically by the Ember source generator.
- NUnit test project: 11 pure math/determinism tests plus 5 World integration tests requiring Unity native containers (auto-skipped on CLI, executed in the Unity Test Runner).
