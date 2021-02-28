
# Minecraft server optimization guide

Guide for version 1.16.5

WARNING: This version of the guide has an experimental layout that is not yet complete!

Based on [this guide](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

## Intro
There will never be a guide that will give you perfect results. Each server has their own needs and limits on how much you can or are willing to sacrifice. Tinkering around with the options to fine tune them to your servers needs is what it's all about. This guide only aims to help you understand what options have impact on performance and what exactly they change. If you think you found inaccurate information within this guide, you're free to open an issue or set up a pull request.

# Preparations

## Server jar
Your choice of server software can make a huge difference in performance and api possibilities. There are currently multiple viable popular server jars, but there are also a few that you should stay away from for various reasons.

Recommended top picks:
* [Paper](https://github.com/PaperMC/Paper) - This is the more popular server software that aims to improve performance while also fix gameplay and mechanics inconsistencies
* [Tuinity](https://github.com/Spottedleaf/Tuinity) - A fork of paper that aims at improving server performance farther than paper
* [Purpur](https://github.com/pl3xgaming/Purpur) - A fork of tuinity that aims at giving the server owner more freedom in feature configurability

You should stay away from:
* Yatopia - "The combined power of Paper forks for maximum instability and unmaintainablity!" - [KennyTV's list of shame](https://github.com/KennyTV/list-of-shame). Nothing more to be said.
* Any paid server jar that claims async anything - 99.99% chance of being a scam.
* Bukkit/Craftbukkit/Spigot - Extremely outdated in terms of performance compared to other server software you have access to.
* Any plugin/software that enables/disables/reloads plugins on runtime. See [this section](#plugins-enablingdisabling-other-plugins) to understand why.

## Map pregen
Map pregeneration is one of the most important steps in improving a low-budget server. This helps out servers that are hosted on a shared cpu/single core node the most, since they can't fully utilize async chunk loading. You can use a plugin such as [chunky](https://github.com/pop4959/Chunky) to pregenerate the world. Make sure to set up a world border so your players don't generate new chunks! Note that pregenning can sometimes take hours depending on the radius you set in the pregen plugin.

It's key to remember that the overworld, nether and the end have separate world borders that need to be set up for each world. The nether dimension is 8x smaller than the overworld (if not modified with a datapack), so if you set the size wrong your players might end up outside of the world border!

**Make sure to set up a vanilla world border (`/worldborder set [radius]`), as it limits certain functionalities such as lookup range for treasure maps that can cause lag spikes.**

# Configurations

## Networking

### [`server.properties`]

#### network-compression-threshold
This allows you to set the cap for the size of a packet before the server attempts to compress it. Setting it higher can save some resources at the cost of bandwidth, and setting it to -1 disables it. If your server is in a network with a proxy or on the same machine (with less than 2 ms ping), disabling this (-1) will be beneficial, since internal network speeds can usually handle the additional uncompressed traffic.

### [`purpur.yml`]

#### use-alternate-keepalive
you can enable purpur's alternate keepalive system so players with bad connection don't get timed out as often. Has known incompatibility with TCPShield.  

```
Enabling this sends a keepalive packet once per second to a player, and only kicks for timeout if none of them were responded to in 30 seconds.
Responding to any of them in any order will keep the player connected. AKA, it won't kick your players because 1 packet gets dropped somewhere along the lines
```
https://pl3xgaming.github.io/PurpurDocs/Configuration/#use-alternate-keepalive

---

## Chunks

### [`spigot.yml`]

#### view-distance
View-distance is distance in chunks around the player that the server will tick. Essentially the distance from the player that things will happen. This includes mobs being active, crops and saplings growing, etc. You should set this value in [`spigot.yml`], as it overwrites the one from [`server.properties`] and can be set per-world. This is an option you want to purposefully set low, somewhere around `3` or `4`, because of the existance of `no-tick-view-distance`.

### [`paper.yml`]

#### no-tick-view-distance
This option allows you to set the maximum distance in chunks that the players will see. This enables you to have lower `view-distance` and still let players see further. It's important to know that while the chunks beyond actual `view-distance` won't tick, they will still load from your storage, so don't go overboard. `10` is basically maximum of what you should set this to. As of now chunks are sent to the client regardless of their view distance setting, so going on higher values for this option can cause issues for players with slower connections.

#### delay-chunk-unloads-by
Lets you configure how long chunks will stay loaded after a player leaves. This helps to not constantly load and unload the same chunks when a player moves back and forth. Too high values can result in way too many chunks being loaded at once.

#### max-auto-save-chunks-per-tick
Lets you slow down incremental world saving by spreading the task over time even more for better average performance. You might want to set this higher than `8` with more than 20-30 players. If incremental save can't finish in time then bukkit will automatically save leftover chunks at once and begin the process again.

#### prevent-moving-into-unloaded-chunks
When enabled, prevents players from moving into unloaded chunks and causing sync loads that bog down the main thread causing lag. The probablility of a player stumbling into an unloaded chunk is higher the lower your view-distance is.

#### entity-per-chunk-save-limit
With the help of this entry you can set limits to how many entities of specified type can be saved. You should provide a limit for each projectile at least to avoid issues with massive amounts of projectiles being saved and your server crashing on loading that. There is an list of all projectiles provided below. Please adjust the limit to your liking. Suggested value for all projectiles is around `10`. You can also add other entities by their type names to that list.
```
entity-per-chunk-save-limit:
    arrow: -1
    dragonfireball: -1
    egg: -1
    ender_pearl: -1
    fireball: -1
    firework: -1
    largefireball: -1
    lingeringpotion: -1
    llamaspit: -1
    shulkerbullet: -1
    sizedfireball: -1
    snowball: -1
    spectralarrow: -1
    splashpotion: -1
    thrownexpbottle: -1
    trident: -1
    witherskull: -1
```

#### armor-stands-tick
In most cases you can safely set this to `false`. If you're using armor stands or any plugins that modify their behavior and you experience issues, re-enable it.

#### armor-stands-do-collision-entity-lookups
Here you can disable armor stand collisions. This will help if you have a lot of armor stands and don't need them colliding with anything.

---

## Mobs

### [`bukkit.yml`]

#### spawn-limits
When `per-player-mob-spawns` is enabled, the math of limiting mobs is just `[playercount] * [limit]`, where "playercount" is current amount of players on the server. Logically, the smaller the numbers are, the less mobs you're gonna see. Reducing this is a double-edged sword; yes, your server has less work to do, but in some gamemodes natural-spawning mobs are a big part of a gameplay. You can go as low as 20 or less if you adjust `mob-spawn-range` properly. If you are using tuinity, you can set mob limits per world in [`tuinity.yml`].

#### ticks-per
This option sets how often (in ticks) the server attempts to spawn certain living entities. Water/ambient mobs do not need to spawn each tick as they don't usually get killed that quickly. As for monsters: Slightly increasing the time between spawns should not impact spawn rates even in mob farms. In most cases all of the values under this option should be higher than `1`.

### [`spigot.yml`]

#### mob-spawn-range
Allows you to reduce the range (in chunks) of where mobs will spawn around the player. Depending on your server's gamemode and its playercount you might want to reduce this value along with [`bukkit.yml`]'s `spawn-limits`.

#### entity-activation-range
You can set what distance from the player an entity should be for it to tick (do stuff). Reducing those values helps performance, but may result in irresponsive mobs until the player gets really close to them.

#### entity-tracking-range
This is distance in blocks from which entities will be visible. Reducing those ranges only saves bandwidth, as entities are still ticked above this range. They just won't be sent to players. If set too low this can cause mobs to seem to appear out of nowhere near a player.

#### tick-inactive-villagers
This allows you to decide if villagers should be ticked outside of the activation range. This will make villagers proceed as normal and ignore the activation range. Disabling this will help performance, but might be confusing for players in certain situations.

#### nerf-spawner-mobs
You can make mobs spawned by a monster spawner have no AI. Nerfed mobs will do nothing. You can make them jump while in water by changing `spawner-nerfed-mobs-should-jump` to `true` in [`paper.yml`].

### [`paper.yml`]

#### despawn-ranges
Lets you adjust entity despawn ranges (in blocks). Lower those values to clear the mobs that are far away from the player faster. You should keep soft range around `30` and adjust hard range to a bit more than your actual view-distance, so mobs don't immediately despawn when the player goes just beyond the point of a chunk being loaded (this works well because of `delay-chunk-unloads-by` in [`paper.yml`]).

#### per-player-mob-spawns
This option decides if mob spawns should account for how many mobs are around target player already. You can bypass a lot of issues regarding mob spawns being inconsistent due to players creating farms that take up the entire mobcap. This will also make the job easier to properly set entity limits, as it makes the math easier.

#### max-entity-collisions
Overwrites option with the same name in [`spigot.yml`]. It lets you decide how many collisions one entity can process at once. Value of `0` will cause inablity to push other entities, including players. Value of `2` should be enough in most cases.

#### update-pathfinding-on-block-update
Disabling this will result in less pathfinding being done, increasing performance, but mobs can appear more laggy; They will just passively update their path every 5 ticks (0.25 sec).

#### fix-climbing-bypassing-cramming-rule
Enabling this will fix entities not being affected by cramming while climbing. This will prevent absurd amounts of mobs being stacked in small spaces even if they're climbing (spiders).

### [`purpur.yml`]

#### dont-send-useless-entity-packets
Enabling this option will save you bandwidth by preventing the server from sending empty position change packets (by default the server sends this packet for each entity even if the entity hasn't moved). May cause some issues with plugins that use client-side entities.

#### aggressive-towards-villager-when-lagging
Enabling this will cause zombies to stop targeting villagers if the server is below the tps treshold set with `lagging-threshold` in [`purpur.yml`].

#### entities-can-use-portals
This option can disable portal usage of all entities besides the player. This prevents entities from loading chunks by changing worlds which is handled on the main thread.

#### villager.brain-ticks
This option allows you to set how often (in ticks) villager brains (work and poi) will tick. Going higher than `3` is confirmed to make villagers inconsistant/buggy.

#### villager.lobotomize
Lobotomized villagers are stripped from their AI and only restock their offers every so often. Enabling this will lobotomize villagers that are unable to pathfind to their destination. Freeing them should unlobotomize them.

---

## Misc

### [`spigot.yml`]

#### merge-radius
This decides the distance between the items and exp orbs to be merged, reducing the amount of items ticking on the ground. Setting this too high will lead to the illusion of items or exp orbs disappearing as they merge together.

### [`paper.yml`]

#### alt-item-despawn-rate
This list lets you set alternative time (in ticks) to despawn certain types of dropped items faster or slower than default. This option can be used instead of item clearing plugins along with `merge-radius` to improve performance.

#### use-faster-eigencraft-redstone
When enabled, the redstone system is replaced by a faster and alternative version that reduces redundant block updates, lowering the amount of work your server has to do.

#### disable-move-event
`InventoryMoveItemEvent` doesn't fire unless there is a plugin actively listening to that event. This means that you only should set this to true if you have such plugin(s) and don't care about them not being able to act on this event. **Do not set to true if you want to use plugins that listen to this event, e.g. protection plugins!**

#### optimize-explosions
Setting this to `true` replaces the vanilla explosion algorithm with a faster one, at a cost of slight inaccuracy when calculating explosion damage. This is usually not noticeable.

#### enable-treasure-maps
Generating treasure maps is extremely expensive and can hang a server if the structure it's trying to locate is outside of your pregenerated world. It's only safe to enable this if you pregenerated your world and set a vanilla world border.

#### treasure-maps-return-already-discovered
Default value of this option forces the newly generated maps to look for unexplored structure, which are usually outside of your pregenerated terrain. Setting this to true makes it so maps can lead to the structures that were discovered earlier. If you don't change this to `true` you may experience server hangs or crashes when generating new treasure maps.

#### grass-spread-tick-rate
Time in ticks the server is trying to spread grass or mycelium. This will make it so large areas of dirt will take a little longer to turn to grass or mycelium. Setting this to around `4` should work nicely if you want to decrease it without it being noticeable.

#### non-player-arrow-despawn-rate
Time in ticks arrows shot by mobs should disappear after hitting something. Players can't pick these up anyway, so you may aswell set this to something like `20`.

#### creative-arrow-despawn-rate
Time in ticks arrows shot by players in creative mode should disappear after hitting something. Players can't pick these up anyway, so you may aswell set this to something like `20`.

### [`purpur.yml`]

#### disable-treasure-searching
Prevents dolphins from performing structure search similiar to treasure maps

#### teleport-if-outside-border
Allows you to teleport the player to the world spawn if they happen to be outside of the world border. Helpful since the vanilla world border is bypassable and the damage it does to the player can be mitigated.

---

## Helpers

### [`paper.yml`]

#### anti-xray
Enable this to hide ores from x-rayers. For detailed configuration of this feature check out [Stonar96's recommended settings](https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e).

#### remove-corrupt-tile-entities
Change this to `true` if you're getting your console spammed with errors regarding tile entities. This will remove any tile entities that cause the error instead of ignoring it.

#### nether-ceiling-void-damage-height
If this option is greater that `0`, players above the set y level will be damaged as if they were in the void. This will prevent players from using the nether roof. Vanilla nether is 128 blocks tall, so you should probably set it to `127`. If you modify the height of the nether in any way you should set this to `[your_nether_height] - 1`.

---

## Java startup flags
[Paper and its forks in upcoming version 1.17 will require Java 11 (LTS) or higher](https://papermc.io/forums/t/java-11-mc-1-17-and-paper/5615). Good 2021 resolution to finally update your version of Java! (or at least inform your host so they can handle the migration).  

JVM can be configured to reduce lag spikes caused by big garbage collector tasks. You can find startup flags optimized for minecraft servers [here](https://mcflags.emc.gs/) [`SOG`].

## "Too good to be true" plugins

#### Plugins removing ground items
Absolutely unnecessary since they can be replaced with [merge radius](#merge-radius) and [alt-item-despawn-rate](#alt-item-despawn-rateenabled) and frankly, they're less configurable than basic server configs. They tend to use more resources scanning and removing items than not removing the items at all.

#### Mob stacker plugins
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all due to the server constantly trying to spawn more mobs. The only "acceptable" use case is for spawners on servers with a large amount of spawners.

#### Plugins enabling/disabling other plugins
Anything that enables or disables plugins on runtime is extremely dangerous. Loading a plugin like that can cause fatal errors with tracking data and disabling a plugin can lead to errors due to removing dependency. The `/reload` command suffers from exact same issues and you can read more about them in [me4502's blog post](https://matthewmiller.dev/blog/problem-with-reload/)

## What's lagging? - measuring performance

#### mspt
Paper offers a `/mspt` command that will tell you how much time the server took to calculate recent ticks. If the first and second value you see are lower than 50, then congratulations! Your server is not lagging! If the third value is over 50 then it means there was at least 1 tick that took longer. That's completely normal and happens from time to time, so don't panic.

#### timings
Great way to see what might be going on when your server is lagging are timings. Timings is a tool that lets you see exactly what tasks are taking the longest. It's the most basic troubleshooting tool and if you ask for help regarding lag you will most likely be asked for your timings.

To get timings of your server you just need to execute the `/timings paste` command and click the link you're provided with. You can share this link with other people to let them help you. It's also easy to misread if you don't know what you're doing. There is a detailed [video tutorial by Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) on how to read them.

#### spark
[Spark](https://github.com/lucko/spark) is a plugin that allows you to profile your servers CPU and memory usage. You can read on how to use it [on its wiki](https://github.com/lucko/spark/wiki/Commands). There's also a guide on how to find the cause of lag spikes [here](https://github.com/lucko/spark/wiki/Finding-the-cause-of-lag-spikes).


[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[`server.properties`]: https://minecraft.gamepedia.com/Server.properties
[`bukkit.yml`]: https://bukkit.gamepedia.com/Bukkit.yml
[`spigot.yml`]: https://www.spigotmc.org/wiki/spigot-configuration/
[`paper.yml`]:  https://paper.readthedocs.io/en/latest/server/configuration.html
[`purpur.yml`]: https://purpur.pl3x.net/docs
[`tuinity.yml`]: https://github.com/Spottedleaf/Tuinity/wiki/Config