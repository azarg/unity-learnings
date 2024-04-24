The simplest approach is to update the game object's transform in MonoBehavior's `Update()` method

```cs
private void Update() {
    transform.position = entityManager.GetComponentData<LocalTransform>(entity).Position;
}
```

**However** this is slow because 
1. calls Update() on every game object (assuming we have a lot of them)
2. accesses the game object's transform
3. GetComponentData becomes a random memory access (because it needs to look up every entity in non sequential order by entity id)

**Alternative** approach (about 30% faster) is to keep a reference to the game object's transform in the entity (as a managed component) and then update all game object transforms in the ISystem:

1. Declare a managed component (note that this has to be a `class` not `struct`):
```cs
public class TransformReference : IComponentData { 
    public Transform Value;
}
```
2. When you create the Entity, add a reference to the game object's transform as a component
```cs
entityManager.AddComponentData(entity, new TransformReference {
    Value = this.transform,
});
```
3. Finally in the ISystem where you run the job to update entity LocalTransforms, update all game object transforms in a single loop
```cs
foreach (var (transform, localTransform) in SystemAPI.Query<TransformReference, RefRO<LocalTransform>>()) {
    transform.Value.position = localTransform.ValueRO.Position;
}
```

Full listing. Assumes there is a single game object in the scene that has `GameObjectAuthoring` MonoBehavior attached to it.  End result is that the game object moves to the right along the x axis.

```cs
public class GameObjectAuthoring : MonoBehaviour
{
    Entity entity;
    EntityManager entityManager;

    private void Start() {
        entityManager = World.DefaultGameObjectInjectionWorld.EntityManager;
        entity = entityManager.CreateEntity();
        entityManager.AddComponentData(entity, new LocalTransform {
            Position = this.transform.position,
            Rotation = quaternion.identity,
            Scale = 1f,
        });
        entityManager.AddComponentData(entity, new TransformReference {
            Value = this.transform,
        });
    }

    //private void Update() {
    //    transform.position = entityManager.GetComponentData<LocalTransform>(entity).Position;
    //}
}

public class TransformReference : IComponentData { 
    public Transform Value;
}

public partial struct TransformUpdateSystem : ISystem
{
    public void OnUpdate(ref SystemState state) {
        new TransformUpdateJob {
            DeltaTime = Time.deltaTime,
        }.Schedule();

        state.Dependency.Complete();

        foreach (var (transform, localTransform) in SystemAPI.Query<TransformReference, RefRO<LocalTransform>>()) {
            transform.Value.position = localTransform.ValueRO.Position;
        }
    }
}

[BurstCompile]
public partial struct TransformUpdateJob : IJobEntity
{
    [ReadOnly] public float DeltaTime;
    public void Execute(ref LocalTransform transform) {
        transform.Position += new float3(1,0,0) * DeltaTime;
    }
}
```
