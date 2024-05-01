Let's say you need to track a particular binary state of an Entity.  For example, assume the Entity represents an enemy, and the state is In/Out of view.  You want to track this state because for example, for enemies "in view" you want to run the movement and AI systems. Where you don't want those systems run for enemies that are "out of view".

Before the going into analysys, my conclusion is that using Enableable components is almost always preferred, unless you need to track the state of 1M+ entities and only small fraction usually has a certain state (e.g. InView)


You could achive this two ways:

### 1.  Add / remove a tag component
```csharp
public struct InView : IComponentData { };

// when Enemy moves into the view
AddComponent<InView>(entity);

// when Enemy moves out of view
RemoveComponent<InView>(entity);
```

Then in your Systems or Jobs you could filter Enemies by InView tag:

```csharp
// in ISystem
SystemAPI.Query<Enemy>.WithAll<InView>()

// in Job
[WithAll(typeof(Inview))]
```

Performance:  
* Adding / removing InView tag component is expensive (think 1-3ms for 1000 add/remove component) because they're structural changes
* Querying becomes very fast for two reasons:
  *  All entities with InView tag will be in the same Archetype (assuming all other components of enemies are the same)
  *  The system will not have to go through enemies that are out of view. So imagine if you have 10k enemies out of view and only 100 enemies in view, the system/job will only iterate over 100 enemies, it will not touch or check the remaining 9900 entities that do not have the tag component.
 
### 2. Enable / disable IEnableableComponent 

```csharp
public struct InView : IEnableableComponent, IComponentData { }
// when Enemy moves into the view
SetComponentEnabled<InView>(entity, true);

// when Enemy moves out of view
SetComponentEnabled<InView>(entity, false);
```

Then in your Systems or Jobs you could filter Enemies by **the value of** InView component.  Although the code is the same `WithAll<InView>`, behind the scenes they work differently.  When InView is an enableable component, the engine has to loop through all entities that have the component and check if the component is enabled or not.  In contrast, when InView is a tag component, all entities with that tag component are already in the same Archetype.  So the query simply iterates over entities in that Archetype

```csharp
// in ISystem
SystemAPI.Query<Enemy>.WithAll<InView>()
```


Performance:  
* Enabling / disabling InView tag component is super fast (think 0.01-0.02ms for 1000 enable/disable operation).
* Querying becomes slower the system will have to iterate through all enemies and check their InView state.  So, if you have 10k enemies, but only 100 are in view at any given time, the system will still have to check all 10k enemies in every frame. Although in practice, this will be only noticeable if you have millions of entities. For reference, querying 1M entities with enableable components took just over 1ms.
