# Common pitfalls and best practices

This article aims to explain common pitfalls that server owners face.

## Always backup
There are two types of people - those who make backups, and those who will start making backups. It's just a matter of time when you experience data loss. Always make copies to avoid losing your worlds or plugin data. You can apply this to any computer related workflow, not just minecraft.

## Don't use outdated software
By running outdated software versions you risk players abusing unpatched exploits, including item duplication (infinite items). It also adds an inconvenience factor since your players have to specifically downgrade their client version to match your server. This can be circumvented by using a protocol hack, but it's not ideal.

## Don't run Bukkit/Spigot anymore
Bukkit and Spigot are basically in maintenance mode. They update anytime there's a new version and if a critical exploit is found, but don't add any performance updates. This means any performance issues you may experience on those softwares will never be improved over time. To avoid that, upgrade to [Paper](https://papermc.io/downloads) or [Purpur](https://purpurmc.org/downloads). Bukkit/Spigot plugins will work just as well (maybe even better) with the server software listed. If they don't, then it's safe to assume that the plugin dev is either doing things that they shouldn't or did a negligent job creating their plugin. They also add optimization patches like a chunk loading system that can take advantage of multiple cpu threads or a setting that allows the server to tick less chunks than it actually sends to the player. See the [main optimization guide](https://github.com/YouHaveTrouble/minecraft-optimization) for more details.

## Avoid shared hosting if possible
Shared hosts are usually the cheapest option, and that's for a valid reason. They offer you 2 types of resources - guaranteed and shared. Guaranteed resources are usually laughably low and may not be enough to run a server for a few players. Shared resources on the other hand are usually enough to run a server with decent performance. There is a catch, though; shared resources, like the name implies, are shared between your server and other servers on the same physical machine. Your server can only benefit from having them when no other server uses them. The situation where your server fully utilises shared resources is pretty much impossible to happen, as most shared hosts oversell their resources. Like airplane tickets, the hosting site sells more resources than they have available in hopes that not all of them will be used. This often leads to situations where all servers are bogged down because there aren't enough resources to spare.

## Avoid datapacks that use command functions
Datapacks that run commands are extremely laggy. It may not be much with a few players on, but that doesn't scale well with the playercount and will lag your server pretty quickly as you gain players. Datapacks that modify biomes, loot tables, etc are fine. You're better off looking for a plugin alternative.

## Choosing hardware
Don't just go off of how much RAM you need. You should instead focus on what kind of CPU you should use, since the CPU is the most important part of the server. You want something that [ranks good on single core performance](https://www.cpubenchmark.net/singleThread.html), as a server mainly runs on one thread. Multiple threads are utilised for quite some time now in systems like async chunk loading on paper, however.

You should absolutely avoid Hard Drives (HDDs). Their speeds are simply way too slow to justify running a server on them since minecraft is heavy on I/O operations (especially with high view distances and higher player counts). A Solid State drive (SSD) is a far better choice because of it's much faster I/O.
