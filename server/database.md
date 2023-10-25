---
description: The most basic about the integrated database.
---

# Database

## Introduction

Let's start with the database. Everything is stored there to save the state of the world, the players and the environment. To make this easier we use [EF-Core](https://github.com/dotnet/efcore), a framework that abstracts the database logic for us.

The most important classes here are the [Context ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Persistence/Context.cs)and the [Model](https://github.com/Parallel-Origin/PO-Server/blob/master/ParallelOriginGameServer/Server/Persistence/Model.cs). The Context connects to the (in memory) database, provides Methods to interact with the Database and the Model contains the database model. Each class in the model describes a table in the database, these tables represent the state of the world. These classes and tables are also called "entities".

As a rule, everything is stored that should survive a server start. This includes every account, every player, every item, every tree, every stone, every mob and all their attributes that are important. Important attributes differ from entity to entity, for example the position, HP what is left or how many items you have of which type. Attributes that are not unique per entity or do not change are stored in the server code and not saved.

## Usages and structure

It is probably important to know WHERE the database is used. Pretty much everywhere. Towards the [start](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L200C9-L200C57), database entities are loaded, including accounts and converted into game entities. During the game, on the other hand, the players and the environment are regularly saved and or loaded. This happens at a regular interval, every few minutes. This can be set in the code. The [`DatabaseGroup` ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L33)is responsible for the (interval) saving. In other places something is also loaded or saved on certain interactions, [for example when creating or entering chunks](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Extensions/PersistenceEnvironmentExtensions.cs) or during [login](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Extensions/PersistenceAuthentificationExtensions.cs#L26).

The [`DatabaseGroup`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L33) is a group of different ECS systems that run in sequence to execute in-game loop logic. In this case, database logic and operations. It consists of the following ordered systems:

* The [`ModelSystem`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L115) iterates over all game entities without a stored database model and creates one for each. This ensures that all game entities are stored in the database. So you can simply create game entities and this system automatically takes care of transferring them to the database.
* The [`DeleteAndDetachModelSystem`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L469C34-L469C34) iterates over all game entities marked with `Destroy`. This destroys them in the game and should therefore also remove them from the database. Some of them also need the `Delete` component to signal that they should be removed from the database. This depends on the context. The system makes sure that these marked entities are removed from the database appropriately.
* The [`ModelUpdateSystem`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L290) iterates over all game entities and their attached database model. In the process, the state of the entity is transferred to the database model. This ensures that the database model is up to date.
* The [`PersistenceSystem`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Systems/PersistenceSystems.cs#L57) stores the [`Context`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Persistence/Context.cs)in the interval and thus all database operations that took place up to this point.

## Configuration

By default, the [Context ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Persistence/Context.cs)uses an in-memory database. This means there is no need to worry about an external one and the server runs out of the box. In the [`GameDBContext`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Persistence/Context.cs#L22) constructor you can set if this should be used or an external one. If an external one is to be used, a [connection string ](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Persistence/Context.cs#L22)must be set in the context and the table structure on the database must be created using EF-Core.\
More info [here](https://github.com/Parallel-Origin/PO-Server/tree/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c#preparing).

## Examples

To learn you can take a look at the [main](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L200C9-L200C57), where for example at the beginning the accounts are loaded.\
Since EF-Core is used, working with the Database is very easy (Example from [main](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L200C9-L200C57)):

{% code title="Program.cs" %}
```csharp
// Loads account from the database and returns them as a list
var accounts = _gameDbContext.Accounts.ToList();
...
```
{% endcode %}

So to work on the database you always need the [`_gameDBContext`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L75) from the [`Program.cs`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Program.cs#L75). Through it you can perform database operations in the normal EF core manner. These operations should mostly be outsourced to own (extension) methods to improve the read flow. Here is another more complex [example](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Extensions/PersistenceAuthentificationExtensions.cs#L65):

{% code title="PersistenceAuthentificationExtensions.cs" %}
```csharp
public static Account Register(this GameDbContext context, string username, string password, string email, Gender gender, Vector2d position = default) {
        
    // Check if account or mail already exists to prevent another registration...
    var accountExists = context.AccountExists(username);
    accountExists.Wait();
    var emailExists = context.EmailExists(email);
    emailExists.Wait();
    
    // Create identity and position
    var identity = new Identity { Tag = "player", Type = "1:1" };
    var transform = new Transform { X = (float)position.X, Y = (float)position.Y, RotX = 0, RotY = 0, RotZ = 0 };
    
    // Create character
    var character = new Character
    {
        Identity = identity,
        Transform = transform,
        Chunk = null,
        Inventory = new HashSet<InventoryItem>(32),
        Health = 100.0f
    };
    
    // Create account and save
    var account = new Account
    {
        Username = username,
        Password = password,
        Email = email,
        Character = character,
        Gender = gender,
        Type = Type.NORMAL,
        Registered = DateTime.UtcNow,
        LastSeen = DateTime.UtcNow,
        AcceptedGdpr = DateTime.UtcNow
    };
    
    // Add some starting items
    var startWoodItem = new InventoryItem
    {
        Identity = new Identity { Id = (long)RandomExtensions.GetUniqueULong(), Type = Types.Gold, Tag = Tags.Item },
        Character = character,
        Amount = 10,
        Level = 0
    };
    character.Inventory.Add(startWoodItem);
    
    // Add to context
    context.Accounts.Add(account);
    context.Identities.Add(identity);
    context.Characters.Add(character);
    context.InventoryItems.Add(startWoodItem);
    
    context.SaveChanges();  // Saves newly added entity automatically
    return account;
}
```
{% endcode %}

The code above should hopefully be self-evident. The extension method [`Register`](https://github.com/Parallel-Origin/PO-Server/blob/78acfe53e7234d61f1cdd0f0a300ac16e9997c2c/ParallelOriginGameServer/Server/Extensions/PersistenceAuthentificationExtensions.cs#L65) is used to create a new account and player. First, it checks asynchronously if the player already exists; if not, it continues and creates and persists the database model of the player. Thus it can be easily called to register new accounts, like this:

```csharp
var registeredAccount = _gameDBContext.Register("TestAccount", "TestPassword", "TestEmail@gmail.com", Gender.Male, new Position{ X = 9.1f, Y = 54.1f}); 
```

Pretty easy, right? \
