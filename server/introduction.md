---
description: A small introduction to the server.
---

# Introduction

The [server ](https://github.com/Parallel-Origin/PO-Server)is the heartpiece of Parallel-Origin. I hope you already know what a server is and how it roughly works. Here again a short explanation.

The actual game takes place on the server. It manages all players, mobs, and environments and takes care of all interactions between the entities. Therefore it is the heart, the brain of this project.

The server is completely written in C# and .Net 7. It has a database interface using [EF-Core](https://github.com/dotnet/efcore) that writes to either a [PostgreSQL ](https://www.postgresql.org/)or an in-memory database. [LiteNetLib ](https://github.com/RevenantX/LiteNetLib)is used for network communication with the client. The complete server uses an ECS architecture, which stands for Entity Component System. The ECS that is used is [Arch](https://github.com/genaray/Arch). If you don't know what this is, please educate yourself about it. We cannot explain everything in detail.

More about this in the next chapters.&#x20;
