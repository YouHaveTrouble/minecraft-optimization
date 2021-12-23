# Minecraft server optimization guide

Note for users that are on vanilla, Fabric or Spigot (or anything below Paper) - go to your server.properties and change `sync-chunk-writes` to `false`. This option is force disabled on Paper and its forks, but on server implementations before that you need to switch this off manually. This allows the server to save chunks off the main thread, lessening the load on the main tick loop.

Guide for version 1.18. Some things may still apply to 1.15 - 1.17.

**PLEASE KEEP IN MIND THAT SOME THINGS ARE NOT YET UPDATED ON PAPER AND BEYOND**

Based on [this guide](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

Use the table of contents located above (next to `README.md`) to easily navigate throughout this guide.

# Intro
There will never be a guide that will give you perfect results. Each server has their own needs and limits on how much you can or are willing to sacrifice. Tinkering around with the options to fine tune them to your servers needs is what it's all about. This guide only aims to help you understand what options have impact on performance and what exactly they change. If you think you found inaccurate information within this guide, you're free to open an issue or set up a pull request.

# Preparations

## Server JAR
Your choice of server software can make a huge difference in performance and API possibilities. There are currently multiple viable popular server JARs, but there are also a few that you should stay away from for various reasons.

Recommended top picks:
* [Paper](https://github.com/PaperMC/Paper) - The most popular server software that aims to improve performance while fixing gameplay and mechanics inconsistencies.
* [Pufferfish](https://github.com/pufferfish-gg/Pufferfish) - Paper fork that aims to further improve server performance.

You should stay away from:
* Yatopia - "The combined power of Paper forks for maximum instability and unmaintainablity!" - [KennyTV's list of shame](https://github.com/KennyTV/list-of-shame). Nothing more to be said. (Moreover, the project has been discontinued.)
* Sugarcane - Yatopia 2.0.
* Mohist - "Mohist is programmed to be malicious, game-breaking, and very unstable" - [Reasons why you shouldn't use it](https://essentialsx.net/do-not-use-mohist.html)
* Any paid server JAR that claims async anything - 99.99% chance of being a scam.
* Bukkit/CraftBukkit/Spigot - Extremely outdated in terms of performance compared to other server software you have access to.
* Any plugin/software that enables/disables/reloads plugins on runtime. See [this section](#plugins-enablingdisabling-other-plugins) to understand why.
* Many forks further downstream from Pufferfish or Purpur will encounter instability and other issues. If you're seeking more performance gains, optimize your server or invest in a personal private fork.

## Map pregen
Map pregeneration is one of the most important steps in improving a low-budget server. This helps out servers that are hosted on a shared CPU/single core node the most, since they can't fully utilize async chunk loading. You can use a plugin such as [Chunky](https://github.com/pop4959/Chunky) to pregenerate the world. Make sure to set up a world border so your players don't generate new chunks! Note that pregenning can sometimes take hours depending on the radius you set in the pregen plugin. Keep in mind that with Paper and above your tps will not be affected by chunk loading, but the speed of loading chunks can significantly slow down when your server's cpu is overloaded.

It's key to remember that the overworld, nether and the end have separate world borders that need to be set up for each world. The nether dimension is 8x smaller than the overworld (if not modified with a datapack), so if you set the size wrong your players might end up outside of the world border!

**Make sure to set up a vanilla world border (`/worldborder set [radius]`), as it limits certain functionalities such as lookup range for treasure maps that can cause lag spikes.**

# Configurations

## Networking

### [server.properties]

#### network-compression-threshold

`Good starting value: 256`

This allows you to set the cap for the size of a packet before the server attempts to compress it. Setting it higher can save some CPU resources at the cost of bandwidth, and setting it to -1 disables it. Setting this higher may also hurt clients with slower network connections. If your server is in a network with a proxy or on the same machine (with less than 2 ms ping), disabling this (-1) will be beneficial, since internal network speeds can usually handle the additional uncompressed traffic.

#### simulation-distance

`Good starting value: 4`

Currently Paper has not updated no-tick-view-distance patch, but Mojang has given us very similiar option. This is the distance in chunks that will actually be ticked, aka. "things will happen in". This is here temporarily until Paper updates no-tick-view-distance patch.

#### view-distance

`Good starting value: 7`

Currently Paper has not updated no-tick-view-distance patch, but Mojang has given us very similiar option. This is the distance in chunks that will be sent to players, similiar to no-tick-view-distance from paper. 1.18 client now respects server side view-distance, which causes ugly fog to appear it this is set low. This is here temporarily until Paper updates no-tick-view-distance patch.

The total view distance will be equal to the greatest value between `simulation-distance` and `view-distance`. For example, if the simulation distance is set to 4, and the view distance is 12, the total distance sent to the client will be 12 chunks.

---

## Chunks

### [spigot.yml]

#### view-distance

`Good starting value: 4`

View-distance is distance in chunks around the player that the server will tick. Essentially the distance from the player that things will happen. This includes furnaces smelting, crops and saplings growing, etc. You should set this value in [spigot.yml], as it overwrites the one from [`server.properties`] and can be set per-world. This is an option you want to purposefully set low, somewhere around `3` or `4`, because of the existence of `no-tick-view-distance`. No-tick allows players to load more chunks without ticking them. This effectively allows players to see further without the same performance impacts.

### [paper.yml]

#### no-tick-view-distance

`Good starting value: 7`

This option allows you to set the maximum distance in chunks that the players will see. This enables you to have lower `view-distance` and still let players see further. It's important to know that while the chunks beyond actual `view-distance` won't tick, they will still load from your storage, so don't go overboard. `10` is basically maximum of what you should set this to. View distance will be matched to client client's render distance setting if it's lower than this value.

#### delay-chunk-unloads-by

`Good starting value: 10`

This option allows you to configure how long chunks will stay loaded after a player leaves. This helps to not constantly load and unload the same chunks when a player moves back and forth. Too high values can result in way too many chunks being loaded at once. In areas that are frequently teleported to and loaded, consider keeping the area permanently loaded. This will be lighter for your server than constantly loading and unloading chunks.

#### max-auto-save-chunks-per-tick

`Good starting value: 8`

Lets you slow down incremental world saving by spreading the task over time even more for better average performance. You might want to set this higher than `8` with more than 20-30 players. If incremental save can't finish in time then bukkit will automatically save leftover chunks at once and begin the process again.

#### prevent-moving-into-unloaded-chunks

`Good starting value: true`

When enabled, prevents players from moving into unloaded chunks and causing sync loads that bog down the main thread causing lag. The probability of a player stumbling into an unloaded chunk is higher the lower your no-tick-view-distance is.

#### entity-per-chunk-save-limit

```
Good starting values:

      experience_orb: 16
      arrow: 16
      dragon_fireball: 3
      egg: 8
      ender_pearl: 8
      eye_of_ender: 8
      fireball: 8
      small_fireball: 8
      firework_rocket: 8
      potion: 8
      llama_spit: 3
      shulker_bullet: 8
      snowball: 8
      spectral_arrow: 16
      experience_bottle: 3
      trident: 16
      wither_skull: 4
      area_effect_cloud: 8
```

With the help of this entry you can set limits to how many entities of specified type can be saved. You should provide a limit for each projectile at least to avoid issues with massive amounts of projectiles being saved and your server crashing on loading that. You can put any entity id here, see the minecraft wiki to find IDs of entities. Please adjust the limit to your liking. Suggested value for all projectiles is around `10`. You can also add other entities by their type names to that list. This config option is not designed to prevent players from making large mob farms.

---

## Mobs

### [bukkit.yml]

#### spawn-limits

```
Good starting values:

    monsters: 8
    animals: 5
    water-animals: 2
    water-ambient: 2
    water-underground-creature: 3
    ambient: 1
```

The math of limiting mobs is `[playercount] * [limit]`, where "playercount" is current amount of players on the server. Logically, the smaller the numbers are, the less mobs you're gonna see. `per-player-mob-spawn` applies an additional limit to this, ensuring mobs are equally distributed between players. Reducing this is a double-edged sword; yes, your server has less work to do, but in some gamemodes natural-spawning mobs are a big part of a gameplay. You can go as low as 20 or less if you adjust `mob-spawn-range` properly. Setting `mob-spawn-range` lower will make it feel as if there are more mobs around each player. If you are using Paper, you can set mob limits per world in [paper.yml].

#### ticks-per

```
Good starting values:

    monster-spawns: 10
    animal-spawns: 400
    water-spawns: 40
    water-ambient-spawns: 20
    water-underground-creature-spawns: 40
    ambient-spawns: 80
```

This option sets how often (in ticks) the server attempts to spawn certain living entities. Water/ambient mobs do not need to spawn each tick as they don't usually get killed that quickly. As for monsters: Slightly increasing the time between spawns should not impact spawn rates even in mob farms. In most cases all of the values under this option should be higher than `1`. Setting this higher also allows your server to better cope with areas where mob spawning is disabled.

### [spigot.yml]

#### mob-spawn-range

`Good starting value: 2`

Allows you to reduce the range (in chunks) of where mobs will spawn around the player. Depending on your server's gamemode and its playercount you might want to reduce this value along with [bukkit.yml]'s `spawn-limits`. Setting this lower will make it feel as if there are more mobs around you. This should be lower than or equal to your view distance, and never larger than your hard despawn range / 16.

#### entity-activation-range

```
Good starting values:

      animals: 16
      monsters: 24
      raiders: 48
      misc: 8
      water: 8
      villagers: 16
      flying-monsters: 48
```

You can set what distance from the player an entity should be for it to tick (do stuff). Reducing those values helps performance, but may result in irresponsive mobs until the player gets really close to them. Lowering this too far can break certain mob farms; iron farms being the most common victim.

#### entity-tracking-range

```
Good starting values:

      players: 48
      animals: 48
      monsters: 48
      misc: 32
      other: 64
```

This is distance in blocks from which entities will be visible. They just won't be sent to players. If set too low this can cause mobs to seem to appear out of nowhere near a player. In the majority of cases this should be higher than your `entity-activation-range`.

#### tick-inactive-villagers

`Good starting value: false`

This allows you to control whether villagers should be ticked outside of the activation range. This will make villagers proceed as normal and ignore the activation range. Disabling this will help performance, but might be confusing for players in certain situations. This may cause issues with iron farms and trade restocking.


### [paper.yml]

#### despawn-ranges

```
Good starting values:

      monster:
        soft: 30
        hard: 56
      creature:
        soft: 30
        hard: 56
      ambient:
        soft: 30
        hard: 56
      axolotls:
        soft: 30
        hard: 56
      underground_water_creature:
        soft: 30
        hard: 56
      water_creature:
        soft: 30
        hard: 56
      water_ambient:
        soft: 30
        hard: 56
      misc:
        soft: 30
        hard: 56
```

Lets you adjust entity despawn ranges (in blocks). Lower those values to clear the mobs that are far away from the player faster. You should keep soft range around `30` and adjust hard range to a bit more than your actual view-distance, so mobs don't immediately despawn when the player goes just beyond the point of a chunk being loaded (this works well because of `delay-chunk-unloads-by` in [paper.yml]). When a mob is out of the hard range, it will be instantly despawned. When between the soft and hard range, it will have a random chance of despawning. Your hard range should be larger than your soft range. You should adjust this according to your view distance using `(view-distance * 16) + 8`. This partially accounts for chunks that haven't been unloaded yet after player visited them.

#### per-player-mob-spawns

`Good starting value: true`

This option decides if mob spawns should account for how many mobs are around target player already. You can bypass a lot of issues regarding mob spawns being inconsistent due to players creating farms that take up the entire mobcap. This will enable a more singleplayer-like spawning experience, allowing you to set lower `spawn-limits`. Enabling this does come with a very slight performance impact, however it's impact is overshadowed by the improvements in `spawn-limits` it allows.

#### max-entity-collisions

`Good starting value: 2`

Overwrites option with the same name in [spigot.yml]. It lets you decide how many collisions one entity can process at once. Value of `0` will cause inability to push other entities, including players. Value of `2` should be enough in most cases. It's worth noting that this will render maxEntityCramming gamerule useless if its value is over the value of this config option.

#### update-pathfinding-on-block-update

`Good starting value: false`

Disabling this will result in less pathfinding being done, increasing performance. In some cases this will cause mobs to appear more laggy; They will just passively update their path every 5 ticks (0.25 sec).

#### fix-climbing-bypassing-cramming-rule

`Good starting value: true`

Enabling this will fix entities not being affected by cramming while climbing. This will prevent absurd amounts of mobs being stacked in small spaces even if they're climbing (spiders).

#### armor-stands-tick

`Good starting value: false`

In most cases you can safely set this to `false`. If you're using armor stands or any plugins that modify their behavior and you experience issues, re-enable it. This will prevent armor stands from being pushed by water or being affected by gravity.

#### armor-stands-do-collision-entity-lookups

`Good starting value: false`

Here you can disable armor stand collisions. This will help if you have a lot of armor stands and don't need them colliding with anything.

#### tick-rates

```
Good starting values:
      sensor:
        villager:
          secondarypoisensor: 40
      behavior:
        villager:
          validatenearbypoi: -1
          acquirepoi: 20
```

This decides how often specified behaviors and sensors are being fired in ticks. It is not recommended to use with DAB.


### [pufferfish.yml]

#### max-loads-per-projectile

`Good starting value: 8`

Specifies the maximum amount of chunks a projectile can load in its lifetime. Decreasing will reduce chunk loads caused by entity projectiles, but could cause issues with tridents, enderpearls, etc.

#### max-tick-freq

`Good starting value: 20`

This option defines the slowest amount entities farthest from players will be ticked. Increasing this value may improve the performance of entities far from view but may break farms or greatly nerf mob behavior.

#### activation-dist-mod

`Good starting value: 7`

Controls the gradient in which mobs are ticked. DAB works on a gradient instead of a hard cutoff like EAR. Instead of fully ticking close entities and barely ticking far entities, DAB will reduce the amount an entity is ticked based on the result of this calculation. Decreasing this will activate DAB closer to players, improving DAB's performance gains, but will affect how entities interact with their surroundings and may break mob farms.

---

## Misc

### [spigot.yml]

#### merge-radius

```
Good starting values:

      item: 3.5
      exp: 4.0
```

This decides the distance between the items and exp orbs to be merged, reducing the amount of items ticking on the ground. Setting this too high will lead to the illusion of items or exp orbs disappearing as they merge together. Setting this too high will break some farms, as well as allow items to teleport through blocks. There are no checks done to prevent items from merging through walls. Exp is only merged on creation.

#### hopper-transfer

`Good starting value: 8`

Time in ticks that hoppers will wait to move an item. Increasing this will help improve performance if there are a lot of hoppers on your server, but will break hopper-based clocks and possibly item sorting systems if set too high.

#### hopper-check

`Good starting value: 8`

Time in ticks between hoppers checking for an item above them or in the inventory above them. Increasing this will help performance if there are a lot of hoppers on your server, but will break hopper-based clocks and item sorting systems relying on water streams.

### [paper.yml]

#### alt-item-despawn-rate

```
Good starting values:

      enabled: true
      items:
          COBBLESTONE: 300
          NETHERRACK: 300
          SAND: 300
          RED_SAND: 300
          GRAVEL: 300
          DIRT: 300
          GRASS: 300
          PUMPKIN: 300
          MELON_SLICE: 300
          KELP: 300
          BAMBOO: 300
          SUGAR_CANE: 300
          TWISTING_VINES: 300
          WEEPING_VINES: 300
          OAK_LEAVES: 300
          SPRUCE_LEAVES: 300
          BIRCH_LEAVES: 300
          JUNGLE_LEAVES: 300
          ACACIA_LEAVES: 300
          DARK_OAK_LEAVES: 300
          CACTUS: 300
          DIORITE: 300
          GRANITE: 300
          ANDESITE: 300
          SCAFFOLDING: 600
```

This list lets you set alternative time (in ticks) to despawn certain types of dropped items faster or slower than default. This option can be used instead of item clearing plugins along with `merge-radius` to improve performance.

#### use-faster-eigencraft-redstone

`Good starting value: true`

When enabled, the redstone system is replaced by a faster and alternative version that reduces redundant block updates, lowering the amount of work your server has to do. Enabling this can significantly improve performance without introducing gameplay inconsistencies. Enabling this will even fix some redstone inconsistencies from craftbukkit.

#### disable-move-event

`Good starting value: false`

`InventoryMoveItemEvent` doesn't fire unless there is a plugin actively listening to that event. This means that you only should set this to true if you have such plugin(s) and don't care about them not being able to act on this event. **Do not set to true if you want to use plugins that listen to this event, e.g. protection plugins!**

#### mob-spawner-tick-rate

`Good starting value: 2`

This option lets you configure how often spawners should be ticked. Higher values mean less lag if you have a lot of spawners, although if set too high (relative to your spawners delay) mob spawn rates will decrease.

#### optimize-explosions

`Good starting value: true`

Setting this to `true` replaces the vanilla explosion algorithm with a faster one, at a cost of slight inaccuracy when calculating explosion damage. This is usually not noticeable.

#### treasure-maps-return-already-discovered

`Good starting value: true`

Default value of this option forces the newly generated maps to look for unexplored structure, which are usually outside of your pregenerated terrain. Setting this to true makes it so maps can lead to the structures that were discovered earlier. If you don't change this to `true` you may experience the server hanging or crashing when generating new treasure maps.

#### grass-spread-tick-rate

`Good starting value: 4`

Time in ticks between the server trying to spread grass or mycelium. This will make it so large areas of dirt will take a little longer to turn to grass or mycelium. Setting this to around `4` should work nicely if you want to decrease it without the decreased spread rate being noticeable.

#### container-update-tick-rate

`Good starting value: 2`

Time in ticks between container updates. Increasing this might help if container updates cause issues for you (it rarely happens), but makes it easier for players to experience desync when interacting with inventories (ghost items).

#### non-player-arrow-despawn-rate

`Good starting value: 20`

Time in ticks after which arrows shot by mobs should disappear after hitting something. Players can't pick these up anyway, so you may as well set this to something like `20` (1 second).

#### creative-arrow-despawn-rate

`Good starting value: 20`

Time in ticks after which arrows shot by players in creative mode should disappear after hitting something. Players can't pick these up anyway, so you may as well set this to something like `20` (1 second).

---

## Helpers

### [paper.yml]

#### anti-xray

`Good starting value: true`

Enable this to hide ores from x-rayers. For detailed configuration of this feature check out [Stonar96's recommended settings](https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e). Enabling this will actually decrease performance, however it is much more efficient than any anti-xray plugin. In most cases the performance impact will be negligible.

#### remove-corrupt-tile-entities

`Good starting value: true`

Change this to `true` if you're getting your console spammed with errors regarding tile entities. This will remove any tile entities that cause the error instead of ignoring it. If you get frequent warnings about tile entities, investigate why they are breaking. This is not a solution to the root issue.

#### nether-ceiling-void-damage-height

`Good starting value: 127`

If this option is greater that `0`, players above the set y level will be damaged as if they were in the void. This will prevent players from using the nether roof. Vanilla nether is 128 blocks tall, so you should probably set it to `127`. If you modify the height of the nether in any way you should set this to `[your_nether_height] - 1`.

---

# Java startup flags
[Vanilla Minecraft and Minecraft server software in version 1.18 requires Java 17 or higher](https://paper.readthedocs.io/en/latest/java-update/index.html). Oracle has changed their licensing, and there is no longer a compelling reason to get your java from them. Recommended vendors are [Amazon Corretto](https://aws.amazon.com/corretto/) and [Adoptium](https://adoptium.net/). Alternative JVM implementations such as OpenJ9 or GraalVM can work, however they are not supported by paper and have been known to cause issues, therefore they are not currently recommended.

Your garbage collector can be configured to reduce lag spikes caused by big garbage collector tasks. You can find startup flags optimized for Minecraft servers [here](https://mcflags.emc.gs/) [`SOG`]. Keep in mind that this recommendation will not work on alternative jvm implementations.

# "Too good to be true" plugins

## Plugins removing ground items
Absolutely unnecessary since they can be replaced with [merge radius](#merge-radius) and [alt-item-despawn-rate](#alt-item-despawn-rate) and frankly, they're less configurable than basic server configs. They tend to use more resources scanning and removing items than not removing the items at all.

## Mob stacker plugins
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all due to the server constantly trying to spawn more mobs. The only "acceptable" use case is for spawners on servers with a large amount of spawners.

## Plugins enabling/disabling other plugins
Anything that enables or disables plugins on runtime is extremely dangerous. Loading a plugin like that can cause fatal errors with tracking data and disabling a plugin can lead to errors due to removing dependency. The `/reload` command suffers from exact same issues and you can read more about them in [me4502's blog post](https://madelinemiller.dev/blog/problem-with-reload/)

# What's lagging? - measuring performance

## mspt
Paper offers a `/mspt` command that will tell you how much time the server took to calculate recent ticks. If the first and second value you see are lower than 50, then congratulations! Your server is not lagging! If the third value is over 50 then it means there was at least 1 tick that took longer. That's completely normal and happens from time to time, so don't panic.

## timings
Great way to see what might be going on when your server is lagging are timings. Timings is a tool that lets you see exactly what tasks are taking the longest. It's the most basic troubleshooting tool and if you ask for help regarding lag you will most likely be asked for your timings.

To get timings of your server you just need to execute the `/timings paste` command and click the link you're provided with. You can share this link with other people to let them help you. It's also easy to misread if you don't know what you're doing. There is a detailed [video tutorial by Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) on how to read them.
  
## spark
[Spark](https://github.com/lucko/spark) is a plugin that allows you to profile your servers CPU and memory usage. You can read on how to use it [on its wiki](https://spark.lucko.me/docs/). There's also a guide on how to find the cause of lag spikes [here](https://spark.lucko.me/docs/guides/Finding-lag-spikes).


[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[server.properties]: https://minecraft.fandom.com/Server.properties
[bukkit.yml]: https://bukkit.gamepedia.com/Bukkit.yml
[spigot.yml]: https://www.spigotmc.org/wiki/spigot-configuration/
[paper.yml]:  https://paper.readthedocs.io/en/latest/server/configuration.html
[pufferfish.yml]: https://github.com/pufferfish-gg/Pufferfish
