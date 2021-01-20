
# Minecraft server optimization guide

Guide for version 1.16.5
Based on [this guide](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

## Intro
To begin I'd like to mention that this is not some sort of "magical cure". There are no guides that will tell you exactly what to do for your server. This is meant to set you in the right direction of finding your server's "sweet spot".

## Server jar
Choice of server jar can make huge difference in performance and api possibilities. There are currently multiple viable popular server jars,
but there are also a few that you should stay away from for various reasons.

My recommendation is:
* [Paper](https://github.com/PaperMC/Paper) - Most popular server software on latest minecraft version
* [Tuinity](https://github.com/Spottedleaf/Tuinity) - per fork improving performance with little to no consequence
* [Purpur](https://github.com/pl3xgaming/Purpur) - Tuinity fork that gives you way more configurability and extra features

You shoud stay away from:
* Yatopia - "The combined power of Paper forks for maximum instability and unmaintainablity!" - Messy, tossed salad of people that haven't even really understood the patch system and destroys functionality of some of these forks. - [KennyTV's list of shame](https://github.com/KennyTV/list-of-shame).
* Any paid server jar that claims async anything - 99.99% of being a scam.
* Bukkit/Craftbukkit/Spigot - Extremely outdated and not up to par with paper+ performance.

## Map pregen
Map pregeneration is one of the most important steps to have lag-free server. In modern versions chunk generation is extremely slow and even servers on best hardware can grind into a halt. You can use plugin such as [chunky](https://github.com/pop4959/Chunky) to pregenerate the world. Remember to also set up a world border so your players don't generate new chunks while the server is open to the public! Pregeneration of the map can take hours (it depends on a radius you set in the pregen plugin).

It's key to remember that overworld, nether and the end have separate world borders and you have to set it up for each world. Remember that nether dimension is usually 8x smaller than overworld, because if you set worldborder wrong your players might end up outside of world border!

**Make sure to set up vanilla world border (`/worldborder set [radius]`), as it limits certain functionalities such as 
lookup range for treasure maps that can cause lag spikes.**

## Configurations

### server.properties

#### network-compression-threshold
**default:** 256
**optimized:** Standalone(512) BungeeCord(-1)
**explanation:**
This option caps the size of a packet before the server attempts to compress it. Setting it higher can save some resources at the cost of bandwidth, setting it to -1 disables it. If your server is in a network with the proxy on localhost or the same datacenter (<2 ms ping), disabling this (-1) will be beneficial.

---

### bukkit.yml

#### spawn-limits
**default:** monsters:70, animals:10, water-animals:15, water-ambient:20, ambient:15
**optimized:** monsters:12, animals:5, water-animals:2, water-ambient:2, ambient:0
**explanation:**
Lower values mean less mobs. Less mobs is less lag in general, but you want to balance it with player quality of life, finding mobs in the world is big part of gameplay. With [per-player-mob-spawns](https://github.com/YouHaveTrouble/minecraft-optimization#per-player-mob-spawns) those numbers represent basically 1:1 limit of mobs in the given category per player, so mob cap math is `playercount*limit`.

#### chunk-gc.period-in-ticks
**default:** 600
**optimized:** 400
**explanation:**
This decides how often vacant chunks are unloaded. Ticking fewer chunks means less TPS consumption.

#### ticks-per
**default:** monster-spawn: 1, animal-spawns: 400, water-spawns: 1, ambient-spawns: 1, water-ambient-spawns: 1 
**optimized:** monster-spawn: 10, animal-spawns: 400, water-spawns: 40, ambient-spawns: 40, water-ambient-spawns: 40
**explanation:**
This sets how often (in ticks) the server attempts to spawn certain living entities. Water/ambient mobs do not need to spawn each tick as they don't usually get killed fast. As for monsters: Slightly increasing the time between spawns should not impact spawn rates even in mob farms.

---

### spigot.yml

#### max-tick-time
**default:** tile:50, entity:50
**optimized:** tile:1000, entity:1000
**explanation:**
Setting those to optimized values disables this feature. You can read why it should be disabled [here](https://aikar.co/2015/10/08/spigot-tick-limiter-dont-use-max-tick-time/).

#### view-distance
**default:** default
**optimized:** 3
**explanation:**
Actual view distance should be set low due to the fact that less chunks will be ticked and paper's no-tick-view-distance lets you send more chunks to player that are actually ticked.

#### mob-spawn-range
**default:** 8
**optimized:** 2
**explanation:**
This usually should be 1 less than view-distance. You can experiment with other values when changing bukkit mob caps.

#### entity-activation-range
**default:** animals:32, monsters:32, raiders: 48, misc:16
**optimized:** animals:16, monsters:24, raiders: 48, misc:8
**explanation:**
Entities past this range will be ticked less often. Avoid setting this too low or you might break mob behavior (mob aggro, raids, etc).

#### tick-inactive-villagers
**default:** true
**optimized:** false
**explanation:**
Enabling this prevents the server from ticking villagers outside the activation range. Villager tasks in 1.14+ are very heavy.

#### merge-radius
**default:** item:2.5, exp:3.0
**optimized:** item:3.0, exp:6.0
**explanation:**
This will decide the distance between the items to be merged, reducing the amount of items ticking on the ground.
Merging will lead to the illusion of items disappearing as they merge together. A minor annoyance.

#### nerf-spawner-mobs
**default:** false
**optimized:** true
**explanation:**
When enabled, mobs from spawners will not have AI (will not swim/attack/move). This is big TPS savings on servers with mob farms, but also messes with their behavior.

---

### paper.yml
Most of the settings in this file can be configured per-world. See [this pdf](https://www.spigotmc.org/attachments/per-world-guide-pdf.444348/) for details.

#### max-auto-save-chunks-per-tick
**default:** 24
**optimized:** 8
**explanation:**
Slows down incremental world saving spreading the task over time even more for better average performance. You might want to set this higher with more than 20-30 players, because if incremental save can't finish in time bukkit will automatically save leftover chunks at once and begin the process again.

#### per-player-mob-spawns
**default:** false
**optimized:** true
**explanation:**
By default mob limits are counted for the entire server which means mobs might end up being distributed unevenly between online players. This option enables per-player mob limits, meaning all players can get approximately the same number of mobs around them regardless of number of online players. Enabling this option also allows you to lower `spawn-limits` in `bukkit.yml` since those are optimized for per-server mob limits.

#### optimize-explosions
**default:** false
**optimized:** true
**explanation:**
Faster explosion alghoritm with no impact on gameplay.

#### max-entity-collisions
**default:** 8
**optimized:** 2
**explanation:**
Less collisions calculation per entity.

#### grass-spread-tick-rate
**default:** 1
**optimized:** 4
**explanation:**
Time in ticks before server tries to spread grass/mycelium. No gameplay impact in most cases.

#### despawn-ranges
**default:** soft: 32, hard: 128
**optimized:** soft: 28, hard: 48
**explanation:**
Lower ranges clear background mobs and allow more to be spawned in areas with player traffic. This further reduces the gameplay impact of reduced spawning (bukkit.yml). Values adjusted for view-distance: 3.

#### hopper.disable-move-event
**default:** false
**optimized:** true
**explanation:**
This will significantly reduce hopper lag by preventing InventoryMoveItemEvent being called for EVERY slot in a container.
**Do not enable if you use plugins that listen to this event!**

#### non-player-arrow-despawn-rate
**default:** -1
**optimized:** 20
**explanation:**
Makes arrows shot by mobs disappear after 1 second after hitting. 

#### creative-arrow-despawn-rate
**default:** -1
**optimized:** 20
**explanation:**
Makes arrows shot by players in creative disappear after 1 second after hitting. 

#### prevent-moving-into-unloaded-chunks
**default:** false
**optimized:** true
**explanation:**
Prevents players from entering an unloaded chunk (due to lag), which causes more lag. The true setting will set them back to a safe location instead.

#### use-faster-eigencraft-redstone
**default:** false
**optimized:** true
**explanation:**
Alternative, faster redstone system. Reduces redundant redstone updates by nearly 95%.

#### alt-item-despawn-rate.enabled
**default:** false
**optimized:** true
**explanation:**
This option lets you despawn selected items faster than default despawn rate. You can add things like cobblestone, netherrack etc. to the list and make them despawn after ~20 seconds (400 ticks).

#### enable-treasure-maps
**default:** true
**optimized:** false
**explanation:**
Generating treasure maps is extremely expensive and can hang a server if the structure it's trying to locate is outside of your pregenerated world. It's only safe to enable this if you pregenerated your world and set vanilla world border.

#### treasure-maps-return-already-discovered
**default:** false
**optimized:** true
**explanation:**
Default value forces the newly generated maps to look for unexplored structure, which are usually outside of your pregenerated terrain. Setting this to true makes it so maps can lead to the structures that were discovered earlier.

#### viewdistances.no-tick-view-distance
**default:** -1
**optimized:** 8
**explanation:**
This allows players to see further without ticking as many chunks as regular view-distance would. Although it's not really heavy on the server keep in mind that sending more chunks will affect bandwidth.

#### entity-per-chunk-save-limit
**default:** -1
**optimized:**
```
entity-per-chunk-save-limit:
    arrow: 8
    dragonfireball: 8
    egg: 8
    ender_pearl: 8
    fireball: 8
    firework: 8
    largefireball: 8
    lingeringpotion: 8
    llamaspit: 8
    shulkerbullet: 8
    sizedfireball: 8
    snowball: 8
    spectralarrow: 8
    splashpotion: 8
    thrownexpbottle: 8
    trident: 8
    witherskull: 8
```
**explanation:**
Limits the amount of projectiles that can be saved in a chunk. This prevents issues that arise with lower view-distance like players throwing massive amounts of snowballs into unloaded chunk that has a potential to crash your server on loading of that chunk.

#### anti-xray.enabled
**default:** false
**optimized:** true
**explanation:**
Hides ores from x-rayers. For detailed configuration of this feature check out [Stonar96's recommended settings](https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e).

---

### purpur.yml
Only applicable for purpur.

#### use-alternate-keepalive
**default:** false
**optimized:** true
**explanation:**
Alternate system for keepalive packets so players with bad connection don't get timed out as often.

#### dont-send-useless-entity-packets
**default:** false
**optimized:** true
**explanation:**
Prevent the server from sending empty position change packets (by default server sends move packet for each entity even if the entity hasn't moved)

#### gameplay-mechanics.player.teleport-if-outside-border
**default:** false
**optimized:** true
**explanation:**
Teleport the player to the world spawn if they happen to be outside of the world border. This will help, because vanilla world border is bypassable and the damage it does to the player can be mitigated.

#### gameplay-mechanics.player.entities-can-use-portals
**default:** true
**optimized:** false
**explanation:**
Disables portal usage of all entities besides player. This potentially fixes a dupe* and prevents entities changing worlds loading chunks on main thread.

**\* more sources needed.**

#### mobs.dolphin.disable-treasure-searching
**default:** false
**optimized:** true
**explanation:**
Prevents dolphins from performing structure search similiar to the one that treasure maps do. Same rules for setting this to false apply.

#### mobs.zombie.aggressive-towards-villager-when-lagging
**default:** false
**optimized:** true
**explanation:**
Zombies stop targetting villagers when tps is under lag treshold. This saves the server precious time of calculating 
paths for zombies that are not targetting players.

## Java startup flags
[Paper and its forks in upcoming version 1.17 will require Java 11 (LTS) or higher](https://papermc.io/forums/t/java-11-mc-1-17-and-paper/5615). Good resolution 2021 to finally update your version of Java!(or at least inform your host so they can handle the migration). JVM can be configured to reduce lag spikes caused by big garbage collector tasks. You can find startup flags optimized for minecraft servers [here](https://mcflags.emc.gs/).

## "Performance" plugins

### Plugins removing ground items
Absolutely unnecessary, can be replaced with spigot configuration (see [merge radius](https://github.com/YouHaveTrouble/minecraft-optimization#merge-radius) and [alt-item-despawn-rate](https://github.com/YouHaveTrouble/minecraft-optimization#alt-item-despawn-rateenabled)) and frankly, they're less configurable than basic server configs. The fact that they usually use more resources to scan and remove items than that items if they would be left alone doesn't help either.

### Mob stacker plugins
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all due to server constantly trying to spawn more mobs. Only "acceptable" use case is spawner mobs on servers with large amount of spawners.

## What's lagging? - measuring performance

### mspt
Paper offers a `/mspt` command that will tell you how much time server took to calculate recent ticks. If the first and 
second value you see are lower than 50, then congratulations! Your server is not lagging! If third value is over 50 it 
means there was at least 1 tick that took longer, that's completely normal and happens from time to time, don't panic.


### timings
Great way to see what might be going on when your server is lagging are timings. Timings is a tool that lets you see exactly what tasks are taking the longest. It's the most basic troubleshooting tool and if you ask for help regarding lag you will most likely be asked for your timings.

To get timings of your server you just need to execute `/timings paste` command and click the link you're provided with. You can share this link with other people to let them help you. It's also easy to misread if you don't know what you're doing. There is a detailed [video tutorial by Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) on how to read them.

### spark
[Spark](https://github.com/lucko/spark) is a plugin that allows you to profile your servers CPU and memory usage. You can read on how to use it [on its wiki](https://github.com/lucko/spark/wiki/Commands).