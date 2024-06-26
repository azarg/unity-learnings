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
Important to note that although you have to specify an initial capacity for the TransformAccessArray, it will automatically grow (by doubling its size) when it fills up.

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

## Questions and issues:
### What happens when the game object is disabled (common practice in object pooling)
TransformAccessArray will still hold the reference to the transform, and surprisingly, the IJobParallelForTransform will continue updating the transforms!!!  This is unlike updating transform through Update() method, in which case the Update() will not be called for disabled game objects.  

It may seem like this is an issue if you're using object pooling strategy where game objects are disabled when returned to the pool.  The issue would be that the job would still run and the game object transforms would still get updated which may seem wasteful and bad for performance.  However, updating transforms using IJobParallelForTransform is really fast (1ms for 10k transform updates). And the calculations run in a burst compiled job, hence also very fast.

But still, this may become an issue with too many objects and complex calculations.

### What happens when the game object is destroyed
Short answer: try to avoid destroying game objects that are added to TransformAccessArray.  The reason is that when the game object is destroyed, its corresponding element in the TransformAccessArray becomes null.  If you subsequently add new elements to the TransformAccessArray, the array will automatically grow instead of filling the null elements.  If you continuously add to TransformAccessArray and then destroy these game objects, the array will keep growing and have lots of null elements which is... not good.  Although this will be a noticeable problem once the array size is in millions.

One workaround would be to dispose and then rebuild the TransformAccessArray from time to time. For example if you have waves of enemies:
1. Create a new TransformAccessArray
2. Spawn enemies and add them to the array
3. Eventually all enemies dies (destroyed) and then you dispose of the array
4. go to 1 for the next wave
