In short, `TransformAccessArray` is an insanely fast way to update your game object transforms.

## Simplest use case:

###### 1. Add `Transform`s to a `TransformAccessArray`

```cs
public class Spawner : MonoBehaviour
{
    public static Spawner Instance;
    public GameObject prefab;
    public TransformAccessArray accessArray;
    private void Awake() {
        Instance = this;

        // allocate the TransformAccessArray with initial capacity
        accessArray = new TransformAccessArray(capacity:10);
        Spawn();
    }
    void Spawn() {
        GameObject instance = Instantiate(prefab);

        // add game object transform to the TransformAccessArray
        accessArray.Add(instance.transform);
    }
}
```
Important to note that although you have to specify an initial capacity for the TransformAccessArray, it will automatically grow (by doubling its size) when time it fills up.

###### 2. Update transforms in a job called from a system

```cs
public partial struct TransformUpdateSystem : ISystem
{
    public void OnUpdate(ref SystemState state) {
        new TransformUpdateJob { }.Schedule(Spawner.Instance.accessArray);
    }
}
[BurstCompile]
public struct TransformUpdateJob : IJobParallelForTransform
{
    public void Execute(int index, TransformAccess transform) {
        transform.position += new Vector3(0.01f, 0.01f, 0f);
    }
}
```
###### 3. Enjoy the benefits
1. Accessing `Transform`s from a `TransformAccess` is orders of magnitude faster than accessing them from GameObject
2. There is no `Update()` on each game object.  This save quite a bit when you have many game objects
3. The job can be burst compiled.  Any complex calculations will be fast
4. The job will run parallel for transforms that are on different hierarchies.  Meaning if all your transforms (in the TransformAccessArray) are under the same parent game object, then the job will run in a single thread, but if the transforms are under 8 different parent game objects (which are not nested with each other), then it will run on 8 different threads in parallel.  


