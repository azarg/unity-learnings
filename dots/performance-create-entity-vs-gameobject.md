## Testing performance of creating / destroying 500 GameObjects vs Entities

### Simplest GameObject:

```csharp
private void Update() {
    for (int i = 0; i < 500; i++) {
        var gameObject = new GameObject();
        Destroy(gameObject);
    }
}
```
<table>
  <tr><th></th><th>CPU</th><th>GC Alloc</th></tr>
  <tr><td>simplest GameObject</td><td>5 ms</td><td>20 KB</td></tr>
</table>

### Simplest Entity:
```csharp
public void OnUpdate(ref SystemState state) {
    for(int i = 0; i < 500; i++) {
        var entity = state.EntityManager.CreateEntity();
        state.EntityManager.DestroyEntity(entity);
    }
}
```

<table>
  <tr><th></th><th>CPU</th><th>GC Alloc</th><th>Archetype memory</th></tr>
  <tr><td>simplest Entity</td><td>4 ms</td><td>0 KB</td><td>64 KB</td></tr>
</table>

Note that entities require more than memory game objects. This is because ECS allocates memory in Chunks of 16KB. By design, each Chunk is limited to at most 128 entities. This means a minimum memory allocation for 500 entities is (16 * RoundUp(500 / 128)) = 64KB!

### Entity with LocalTransform:

Comparing simplest Entity with simplest GameObject is not apples to apples, because creating the simplest GameObject also creates a Transform component.  Whereas the simplest Entity does create a LocalTransform (entities equivalent of Transform) component.  Therefore, next test creates entities with LocalTransform for more fair comparison with GameObjects

```csharp
public void OnUpdate(ref SystemState state) {
    for(int i = 0; i < 500; i++) {
        var entity = state.EntityManager.CreateEntity();
        state.EntityManager.AddComponentData(entity, new LocalTransform { });
        state.EntityManager.DestroyEntity(entity);
    }
}
```
<table>
  <tr><th></th><th>CPU</th><th>GC Alloc</th><th>Archetype memory</th></tr>
  <tr><td>Entity with transform</td><td>8 ms</td><td>0 KB</td><td>64 KB</td></tr>
</table>

### Creating entities using EntityCommandBuffer:
```csharp
public void OnUpdate(ref SystemState state) {
    var ecb = new EntityCommandBuffer(Allocator.TempJob);
    for (int i = 0; i < 500; i++) {
        var entity = ecb.CreateEntity();
        ecb.AddComponent(entity, new LocalTransform { });
        ecb.DestroyEntity(entity);
    }
    ecb.Playback(state.EntityManager);
    ecb.Dispose();
}
```
<table>
  <tr><th></th><th>CPU</th><th>GC Alloc</th><th>Archetype memory</th></tr>
  <tr><td>Entity with transform</td><td><2 ms</td><td>0 KB</td><td>64 KB</td></tr>
</table>
    
![image](https://github.com/azarg/unity-learnings/assets/6077141/4a04135c-3ef2-4079-ade4-c9fb3e950f96)

### Conclusion:
Creating/destroying Entities is NOT significantly faster than creating/destroying GameObjects, unlike what I presumed before the tests.

In fact, creating/destroying entities with a LocalTransform component is **slower** than creating/destroying GameObjects (which get Transforms created automatically). The performance comparison is 5ms for GameObjects vs 8ms for Entities (300 each), i.e. Entities are >50% slower. 

We get better performance if we batch Entity creation/destruction using EntityCommandBuffer. This time, performance of creating/destroying Entities goes down to 2ms. However, in pactice it may be hard to consolidate all entity creation/destruction into a single command buffer.  

