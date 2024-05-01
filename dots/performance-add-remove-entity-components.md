## Testing performance of adding/removing 1000 components to Entities

### 1. Simple Component with a single float field

![image](https://github.com/azarg/unity-learnings/assets/6077141/c6aa0775-ec79-4c97-a297-eae6088e5e37)


<details>
<summary>View code</summary>

```csharp
namespace Assets.Scripts.Systems
{
    public struct TestComponent : IComponentData { }
    public struct FloatComponent : IComponentData { public float Value; }

    public partial struct TestSystem : ISystem
    {
        static readonly ProfilerMarker __marker = new ProfilerMarker("MyMarker.AddRemoveComponent");

        public void OnCreate(ref SystemState state) {
            for (int i = 0; i < 1000; i++) {
                var entity = state.EntityManager.CreateEntity();
                state.EntityManager.AddComponent<TestComponent>(entity);
            }
        }

        public void OnUpdate(ref SystemState state) {
            __marker.Begin();
            var ecb = new EntityCommandBuffer(Allocator.Temp);
            foreach(var (_, entity) in SystemAPI.Query<TestComponent>().WithEntityAccess()) {
                ecb.AddComponent(entity, new FloatComponent { Value = 0f });
                ecb.RemoveComponent<FloatComponent>(entity);
            }
            ecb.Playback(state.EntityManager);
            ecb.Dispose();
            __marker.End();
        }
    }
}
```
</details>

### 2. Adding BurstCompile

Code change:

```csharp
[BurstCompile]
public void OnUpdate(ref SystemState state){ }
```

![image](https://github.com/azarg/unity-learnings/assets/6077141/f518c898-d914-44e2-b6f8-7e83a29f503c)

### 3. Disable safety checks

Code change:

```csharp
[BurstCompile(DisableSafetyChecks = true)]
public void OnUpdate(ref SystemState state){ }
```

![image](https://github.com/azarg/unity-learnings/assets/6077141/c51987a6-439b-49ec-9917-eb1fedaa1a44)

### 4. Add/remove tag component

Tag components are IComponentData structs that have no fields.

Code change (still using [BurstCompile(DisableSafetyChecks = true)]) :

```csharp
public struct TagComponent : IComponentData { }
...
    ecb.AddComponent<TagComponent>(entity);
    ecb.RemoveComponent<TagComponent>(entity);
```

![image](https://github.com/azarg/unity-learnings/assets/6077141/13a630ae-9fde-4050-989d-f2896acc94d7)

### 5. Add/remove tag component without BurstCompile

Same as previous, but [BurstCompile] removed.  Compare results to test 1.

![image](https://github.com/azarg/unity-learnings/assets/6077141/22c4ecb3-8ae3-4ad3-8a5e-f6f5efa0f4ba)

### 6. Add/remove empty managed component (cannot be Bursted)

Code change:

```csharp
public class ManagedComponent : IComponentData { }
...
    ecb.AddComponent<ManagedComponent>(entity);
    ecb.RemoveComponent<ManagedComponent>(entity);
```

![image](https://github.com/azarg/unity-learnings/assets/6077141/245c71f4-3157-4e84-911c-4d4ef7edf199)
