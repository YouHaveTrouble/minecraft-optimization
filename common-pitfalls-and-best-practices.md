# Common pitfalls and best practices

This article aims to explain common pitfalls that server owners face.

## Always backup
There are two types of people - ones making backups, and ones that will start making backups. It's just a matter of time when you experience data loss, always make copies to avoid losing your worlds or plugin data. You can apply this to any computer related workflow, not only minecraft.

## Don't use outdated software
By running outdated software versions you risk players abusing unpatched exploits including item duplication (infinite items). Besides that your players have to specifically downgrade their client version to match your server unless you use a protocol hack, which is not ideal aswell. 

## Don't run bukkit/spigot anymore
Bukkit and spigot are basically in maintenence mode. They update version and critical exploits, but don't add any performance updates. This means you will most likely experience performance issues on those. To avoid that upgrade to [paper](https://papermc.io/downloads), [tuinity](https://ci.codemc.io/job/Spottedleaf/job/Tuinity) or [purpur](https://purpur.pl3x.net/downloads) which support 99.99% of the spigot plugins (the general rule is if a plugin works on spigot but doesn't work on paper plugin dev did a bad job or messes with things that they shouldn't mess with). In addition those forks also add optimization patches, like chunk loading system that can take advantage of multiple cpu threads or a setting that allows the server to tick less chunks that it actually sends to the player. See the [main optimization guide](https://github.com/YouHaveTrouble/minecraft-optimization) for details.

## Avoid shared hosting if possible
Shared hosts are usually the cheapest option, and that's for a reason. They offer you 2 types of resources - guaranteed and shared. Guaranteed resources are usually laughably low and may be even not enough to run a server for a few players. Shared resources is different story, that's usually enough to run a server with decent performance. There is a catch though. Shared resources, like name implies are shared between your server and other servers on the same physical machine. Your server can only benefit from having them when no other server uses them. The situation where your server fully utilises shared resources is pretty much impossible to happen, as most shared hosts oversell the resources. Like airplane tickets, hosting site sells more resources than they have available in hopes that not all of them will be used. This often leads to situations where all servers are bogged down because there aren't enough resources to spare.