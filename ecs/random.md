

**Seeding using UnityEngine.Random**

```cs
var rng = new Unity.Mathematics.Random((uint)UnityEngine.Random.Range(1, int.MaxValue));
```

Use case: if you're setting up the game, and need to randomly place some objects (entities). The entities are instantiated inside OnUpdate of an ISystem. 
Issues: 
  - cannot seed rng like this in a worker thread because UnityEngine.Random has static state
  - passing this rng to a parallel job will result in getting the same random numbers in each thread

Notes: UnityEngine.Random will produce different results on every run of the application, as well as each time you enter play mode. This happens because UnityEngine.Random is seeded on process start. 


**Seeding using System.DateTime.Now.Ticks**

AVOID doing this. This code cannot be burst compiled
```cs
var rng = Unity.Mathematics.Random.CreateFromIndex((uint)System.DateTime.Now.Ticks);
```


**Passing as argument**

Random is a struct and keeps an interal state.  Therefore passing as value will not update the internal state and result in repeated numbers being generated
```cs
var rng = new Unity.Mathematics.Random(123);
for (int i = 0; i < 10; i++) {
    PrintSameRandomNumber(rng);
}

for (int i = 0; i < 10; i++) {
    PrintDifferentRandomNumbers(ref rng);
}

private void PrintSameRandomNumber(Random rng) => UnityEngine.Debug.Log(rng.NextInt());
private void PrintDifferentRandomNumbers(ref Random rng) => UnityEngine.Debug.Log(rng.NextInt());
```
