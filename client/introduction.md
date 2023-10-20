---
description: A small introduction to the client.
---

# Introduction

The [client ](https://github.com/Parallel-Origin/PO-Client)is the visual representation. It displays what the player sees and registers his interactions to forward them to the server or receive them from the server.

The server calculates and directs. And the client only displays. At the same time, the client has very few rights. All important calculations or decisions are necessarily made on the server, because a client is easy to manipulate.

The client is completely written in C# and Unity. [LiteNetLib ](https://github.com/RevenantX/LiteNetLib)is used for network communication with the server. And for displaying and managing the entities [Unitys ECS ](https://unity.com/de/ecs)is used. So the client internalizes a hybrid of Unity's normal GameObject flow and the ECS architecture.

More about this in the next chapters.&#x20;
