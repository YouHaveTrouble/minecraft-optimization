
# Minecraft server optimization guide

Guide for version 1.16.5

WARNING: This version of the guide has experimental layout that is not yet complete!

Based on [this guide](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

## Intro
There will never be a guide that you can follow and it will give you perfect results. Each server has their own needs and limits on how much you can or you are willing to sacrifice. Tinkering around with the options to fine tune them to your servers needs it's what it's all about. This guide only aims to help you understand what options have impact on performance and what exactly they change. If you think you found inaccurate information within the guide you're free to open github issue telling me about it or open a pull request.

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
* Any plugin/software that enables/disables/reloads plugins on runtime. See [this section](#plugins-enablingdisabling-other-plugins) to read why.

## Map pregen
Map pregeneration is one of the most important steps in improving a low-budget server. This helps out servers that are hosted on a shared cpu/single core node the most, since they can't fully utilize async chunk loading. You can use a plugin such as [chunky](https://github.com/pop4959/Chunky) to pregenerate the world. Make sure to set up a world border so your players don't generate new chunks! Note that pregenning can sometimes take hours depending on the radius you set in the pregen plugin.

It's key to remember that the overworld, nether and the end have separate world borders that need to be set up for each world. The nether dimension is 8x smaller than the overworld (if not modified with a datapack), so if you set the size wrong your players might end up outside of the world border!

**Make sure to set up a vanilla world border (`/worldborder set [radius]`), as it limits certain functionalities such as lookup range for treasure maps that can cause lag spikes.**

# Configurations

## Networking

### Network-compression-threshold
There is an option in `server.properties` file, called `network-compression-threshold`. This allows you to set the cap for the size of a packet before the server attempts to compress it. Setting it higher can save some resources at the cost of bandwidth, and setting it to -1 disables it. If your server is in a network with the proxy or on the same machine (with less than 2 ms ping), disabling this (-1) will be beneficial, as usually internal network speeds can handle the additional uncompressed traffic.

---

## Chunks

### view-distance
View-distance is distance in chunks around the player that server will tick. Essentially the distance from player that things will happen. This includes mobs being active, crops and saplings growing, etc. You should set this value in `spigot.yml`, as it overwrites the one from `server.properties` and can be set per-world. This option you want to purposefully set low, around `3` or `4`, because of the existance of `no-tick-view-distance`.

### no-tick-view-distance
This option is added in `paper.yml` and allows you to set the maximum distance in chunks that players will see. This enables you to have lower `view-distance` and still let players see further. It's important to know that while the chunks beyond actual `view-distance` won't tick, they will still load from your storage, so don't go overboard. `10` is basically maximum of what you should set this to. As of now chunks are sent to the client regardless of their view distance setting, so going on higher values for this option can cause issues for players with slower connection.

### chunk-gc
`spigot.yml` lest you configure how often unused chunks should be unloaded. `peroid-in-ticks` is time in ticks between the unloads. The default value for this is `600`, however going as low as `400` is recommended, so you reduce the amount of ticking chunks.

---

## Mobs

### per-player-mob-spawns
This option decides if mob spawns should account for how many mobs are around target player already. You can bypass a lot of issues regarding mob spawns being inconsistent due to players creating farms that take up entire mobcap. if you change `per-player-mob-spawns` to `true` in `paper.yml`. This will also make the job easier to properly set entity limits, as it makes the math easier.

### spawn-limits
Located in `bukkit.yml`. When `per-player-mob-spawns` is enabled the math limiting mobs is just `playercount*limit`, where "playercount" is current amount of players on the server. Logically, smaller the numbers are, less mobs you're gonna see. Reducing this is a double-edged sword, as yes, your server has less work to do, but in some gamemodes natural mobs are big part of a gameplay. You can go as low as 20 or less if you adjust `mob-sapwn-range` properly.

### mob-spawn-range
`spigot.yml` offers an option to reduce the range (in chunks) of where mobs will spawn around the player. Depending on your server's gamemode and its playercount you might want to reduce this value along with `bukkit.yml`'s `spawn-limits`.

### entity-tracking-range
Option available in `spigot.yml`. This is distance in blocks from which entities will be visible. Reducing those ranges only saves bandwidth, as entities are still ticked above this range. They just won't be sent to players. If set too low this can cause mobs seem to appear out of nowhere near a player.

---

## Java startup flags
[Paper and its forks in upcoming version 1.17 will require Java 11 (LTS) or higher](https://papermc.io/forums/t/java-11-mc-1-17-and-paper/5615). Good 2021 resolution to finally update your version of Java! (or at least inform your host so they can handle the migration).  

JVM can be configured to reduce lag spikes caused by big garbage collector tasks. You can find startup flags optimized for minecraft servers [here](https://mcflags.emc.gs/) [SOG].

## "Too good to be true" plugins

### Plugins removing ground items
Absolutely unnecessary since they can be replaced with [merge radius](#merge-radius) and [alt-item-despawn-rate](#alt-item-despawn-rateenabled) and frankly, they're less configurable than basic server configs. They tend to use more resources scanning and removing items than not removing the items at all.

### Mob stacker plugins
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all due to the server constantly trying to spawn more mobs. The only "acceptable" use case is for spawners on servers with a large amount of spawners.

### Plugins enabling/disabling other plugins
Anything that enables or disables plugins on runtime is extremely dangerous. Loading a plugin like that can cause fatal errors with tracking data and disabling a plugin can lead to errors due to removing dependency. The `/reload` command suffers from exact same issues and you can read more about them in [this me4502's blog post](https://matthewmiller.dev/blog/problem-with-reload/)

## What's lagging? - measuring performance

### mspt
Paper offers a `/mspt` command that will tell you how much time server took to calculate recent ticks. If the first and second value you see are lower than 50, then congratulations! Your server is not lagging! If the third value is over 50 then it means there was at least 1 tick that took longer. That's completely normal and happens from time to time, so don't panic.


### timings
Great way to see what might be going on when your server is lagging are timings. Timings is a tool that lets you see exactly what tasks are taking the longest. It's the most basic troubleshooting tool and if you ask for help regarding lag you will most likely be asked for your timings.

To get timings of your server you just need to execute `/timings paste` command and click the link you're provided with. You can share this link with other people to let them help you. It's also easy to misread if you don't know what you're doing. There is a detailed [video tutorial by Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) on how to read them.

### spark
[Spark](https://github.com/lucko/spark) is a plugin that allows you to profile your servers CPU and memory usage. You can read on how to use it [on its wiki](https://github.com/lucko/spark/wiki/Commands). There's also a guide on how to find the cause of lag spikes [here](https://github.com/lucko/spark/wiki/Finding-the-cause-of-lag-spikes).


[SOG]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/