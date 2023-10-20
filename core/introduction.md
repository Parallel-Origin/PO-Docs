---
description: A small introduction to the shared codebase between client and server.
---

# Introduction

The [Core ](https://github.com/Parallel-Origin/PO-Core)is the shared codebase between Client and Server. Why all this? Well, there is a lot of code that is shared and would otherwise have to be duplicated. So you can be a bit clever here and save some work.

Client and server both used an ECS architecture. In this architecture the game entities have exactly the same components they consist of. Also the networking code is shared and uses the same commands. This gives you full control at a glance.

More about this in the next chapters.&#x20;
