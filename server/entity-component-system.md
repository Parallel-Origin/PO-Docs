---
description: Everything important about the ECS.
---

# Entity Component System

## Introduction

The [ECS (Entity Component System)](https://www.simplilearn.com/entity-component-system-introductory-guide-article) is part of the basic server architecture. It manages game entities, allows them to be created from compositions of different components and allows us to easily iterate over them to execute logic on them. It is the way we express and deal with game-entities. And the best... it is incredibly efficient and fast! For this, the server uses [Arch](https://github.com/genaray/Arch), an ECS written by the same team and one of the fastest (if not the fastest without bragging).&#x20;

{% hint style="info" %}
For more information on how exactly an ECS works, check out the [Arch ](https://github.com/genaray/Arch)[repo](https://github.com/genaray/Arch) and [wiki](https://github.com/genaray/Arch/wiki).
{% endhint %}

## Usages

So where is it used and how? Well... everywhere. The whole world and the main part of the server is expressed as ECS. Already in [`Program.cs`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L127) it starts, there the world is created in which all game entities will live.&#x20;

{% code title="Program.cs" %}
```csharp
_world = World.Create();
```
{% endcode %}

And shortly after that a [group of systems is created](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L127), which contain the world logic and process game entities. This group is processed every few frames, it is [part of game loop](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L295)... it IS the game loop.

```csharp
var systems = new Arch.System.Group(
 
        // Reactive Systems
        new ReactiveGroup(_world),
        new StartCommandBufferSystem(_world),
        new InitialisationSystem(_world),

        // Commands, triggering logic between entities mostly 
        new CommandGroup(_world),
        
        // UI & Interacting
        new InteractionGroup(_world),

        // Environment group ( Chunks, terrain generation... )
        new EnvironmentGroup(_world, _entityPrototyper, _biomeGeoTiff),

        // Behaviour like ai and animations
        new BehaviourGroup(_world),

        // Movement & Combat
        new MovementGroup(_world),
        new CombatGroup(_world),

        // Phyics  & Activity group
        new PhysicsGroup(_world),
        new ActivityGroup(_world, _entityPrototyper),

        // Networking group & Network Commands
        new NetworkingGroup(_world, _entityPrototyper, Network),

        // Database group
        new DatabaseGroup(_world, _gameDbContext),
        new IntervallGroup(10.0f, new DebugSystem(_world, Network)),

        // End of the frame
        new ClearTrackedAnimationsSystem(_world),
        new DestroySystem(_world)
);
systems.Initialize();
```

Each of these groups is called and processed in order. Each group in turn contains several systems or further groups. Each group has a task, e.g. there is the [`MovementGroup` ](https://github.com/Parallel-Origin/PO-Server/blob/master/ParallelOriginGameServer/Server/Systems/MovementSystem.cs#L51)in there, which makes sure that for entities of each frame the movement is calculated and set correctly or the [`DatabaseGroup` ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L33)which takes care of the interval saving and other entity database operations. However, groups and systems do not necessarily have to target entities.&#x20;

{% hint style="info" %}
Each group can contain multiple systems and each system can contain multiple queries that execute logic on entities.
{% endhint %}

So we have the world and the groups/systems with queries that execute logic on entities. What is still missing? Exactly, the entities themselves. They are created in many different places. For example, at the start, after the accounts are loaded, they are [converted to player entities in the world](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L281).

{% code title="Program.cs" %}
```csharp
// Spawn in players
for (var index = 0; index < accounts.Count; index++)
{
    // Get account & character
    var acc = accounts[index]; 
    acc.ToEcs(); // Converts it to a game-entity by a extension-method               
}
```
{% endcode %}

Or [here ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/EnvironmentSystems.cs#L442C12-L447C74)when creating new areas and chunks in the world.

{% code title="EnvironmentSystems.cs" %}
```csharp
var newEntity = _prototyperHierarchy.Clone(choosedEntity.Type);   // Creates an entity
ref var entityTransform = ref newEntity.Get<NetworkTransform>();
ref var entityRotation = ref newEntity.Get<NetworkRotation>();

entityTransform.Pos = noiseGeocordinates.Geocoordinates;
entityRotation.Value = RandomExtensions.QuaternionStanding();
```
{% endcode %}

{% hint style="info" %}
The creation of entities is often abstracted, in the example above once using an extension method and below using a prototyper.
{% endhint %}

## Example

Let's take a quick look at the [`MovementSystem` ](https://github.com/Parallel-Origin/PO-Server/blob/master/ParallelOriginGameServer/Server/Systems/MovementSystem.cs#L51)within the [`MovementGroup` ](https://github.com/Parallel-Origin/PO-Server/blob/master/ParallelOriginGameServer/Server/Systems/MovementSystem.cs#L51)to see what such a system and a query looks like.

```csharp
[Query]
[None(typeof(Prefab), typeof(Dead))]
[MethodImpl(MethodImplOptions.AggressiveInlining)]
private void Move([Data] in float elapsedTime, in Entity entity, ref NetworkTransform transform, ref Movement movement)
{
   // Use the passed entity to modify its data to make it move
}
```

Don't worry, I left out some code in the method. Because the most important thing is the method signature. This method acts as a query. It will be executed for all game entities that match the filters of the method. The best thing to do is to browse here how exactly this works.&#x20;

So the method above will be executed for all entities that:

* Without a prefab component
* Without a dead component
* With a NetworkTransform component
* With a Movement component&#x20;

In this case, the method is called, passing the entity being iterated over with its requested components. And the game-time.

And if we have the entity, its transform and its movement. We can easily move the entity a little bit each time it is called. Pretty simple, isn't it?
