# Minecraft server optimization guide

Note for users that are on vanilla, Fabric or Spigot (or anything below Paper) - go to your server.properties and change `sync-chunk-writes` to `false`. This option is forcibly set to false on Paper and its forks, but on other server implementations you need to switch this to false manually. This allows the server to save chunks off the main thread, lessening the load on the main tick loop.

Guide for version 1.21.5. Some things may still apply to 1.15 - 1.21.4.

Based on [this guide](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) and other sources (all of them are linked throughout the guide when relevant).

Use the table of contents located above (next to `README.md`) to easily navigate throughout this guide.

# Intro
There will never be a guide that will give you perfect results. Each server has their own needs and limits on how much you can or are willing to sacrifice. Tinkering around with the options to fine tune them to your servers needs is what it's all about. This guide only aims to help you understand what options have impact on performance and what exactly they change. If you think you found inaccurate information within this guide, you're free to open an issue or set up a pull request to correct it.

# Preparations

## Server JAR
Your choice of server software can make a huge difference in performance and API possibilities. There are currently multiple viable popular server JARs, but there are also a few that you should stay away from for various reasons.

Recommended top picks:
* [Paper](https://github.com/PaperMC/Paper) - The most popular server software that aims to improve performance while fixing gameplay and mechanics inconsistencies.
* [Purpur](https://github.com/PurpurMC/Purpur) - Paper fork focused on features and the freedom of customization.

You should stay away from:
* Any paid server JAR that claims async anything - 99.99% chance of being a scam.
* Bukkit/CraftBukkit/Spigot - Extremely outdated in terms of performance compared to other server software you have access to.
* Any plugin/software that enables/disables/reloads plugins on runtime. See [this section](#plugins-enablingdisabling-other-plugins) to understand why.
* Many forks further downstream from Paper or Purpur will encounter instability and other issues. If you're seeking more performance gains, optimize your server or invest in a personal private fork.

## Map pregen
Map pregeneration, thanks to various optimizations to chunk generation added over the years is now only useful on servers with terrible, single threaded, or limited CPUs. Though, pregeneration is commonly used to generate chunks for world-map plugins such as Pl3xMap or Dynmap.

If you still want to pregen the world, you can use a plugin such as [Chunky](https://github.com/pop4959/Chunky) to do it. Make sure to set up a world border so your players don't generate new chunks! Note that pregenning can sometimes take hours depending on the radius you set in the pregen plugin. Keep in mind that with Paper and above your tps will not be affected by chunk loading, but the speed of loading chunks can significantly slow down when your server's cpu is overloaded.

It's key to remember that the overworld, nether and the end have separate world borders that need to be set up for each world. The nether dimension is 8x smaller than the overworld (if not modified with a datapack), so if you set the size wrong your players might end up outside of the world border!

**Make sure to set up a vanilla world border (`/worldborder set [diameter]`), as it limits certain functionalities such as lookup range for treasure maps that can cause lag spikes.**

# Configurations

## Networking

### [server.properties]

#### network-compression-threshold

`Good starting value: 256`

This allows you to set the cap for the size of a packet before the server attempts to compress it. Setting it higher can save some CPU resources at the cost of bandwidth, and setting it to -1 disables it. Setting this higher may also hurt clients with slower network connections. If your server is in a network with a proxy or on the same machine (with less than 2 ms ping), disabling this (-1) will be beneficial, since internal network speeds can usually handle the additional uncompressed traffic.

### [purpur.yml]

#### use-alternate-keepalive

`Good starting value: true`

You can enable Purpur's alternate keepalive system so players with bad connection don't get timed out as often. Has known incompatibility with TCPShield.

> Enabling this sends a keepalive packet once per second to a player, and only kicks for timeout if none of them were responded to in 30 seconds. Responding to any of them in any order will keep the player connected. AKA, it won't kick your players because 1 packet gets dropped somewhere along the lines  
~ https://purpurmc.org/docs/Configuration/#use-alternate-keepalive

---

## Chunks

### [server.properties]

#### simulation-distance

`Good starting value: 4`

Simulation distance is distance in chunks around the player that the server will tick. Essentially the distance from the player that things will happen. This includes furnaces smelting, crops and saplings growing, etc. This is an option you want to purposefully set low, somewhere around `3` or `4`, because of the existence of `view-distance`. This allows to load more chunks without ticking them. This effectively allows players to see further without the same performance impact.

#### view-distance

`Good starting value: 7`

This is the distance in chunks that will be sent to players, similar to no-tick-view-distance from paper.

The total view distance will be equal to the greatest value between `simulation-distance` and `view-distance`. For example, if the simulation distance is set to 4, and the view distance is 12, the total distance sent to the client will be 12 chunks.

### [spigot.yml]

#### view-distance

`Good starting value: default`

This value overwrites server.properties one if not set to `default`. You should keep it default to have both simulation and view distance in one place for easier management.

### [paper-world configuration]

#### delay-chunk-unloads-by

`Good starting value: 10s`

This option allows you to configure how long chunks will stay loaded after a player leaves. This helps to not constantly load and unload the same chunks when a player moves back and forth. Too high values can result in way too many chunks being loaded at once. In areas that are frequently teleported to and loaded, consider keeping the area permanently loaded. This will be lighter for your server than constantly loading and unloading chunks.

#### prevent-moving-into-unloaded-chunks

`Good starting value: true`

When enabled, prevents players from moving into unloaded chunks and causing sync loads that bog down the main thread causing lag. The probability of a player stumbling into an unloaded chunk is higher the lower your view-distance is.

#### entity-per-chunk-save-limit

```
Good starting values:

    area_effect_cloud: 8
    arrow: 16
    breeze_wind_charge: 8
    dragon_fireball: 3
    egg: 8
    ender_pearl: 8
    experience_bottle: 3
    experience_orb: 16
    eye_of_ender: 8
    fireball: 8
    firework_rocket: 8
    llama_spit: 3
    splash_potion: 8
    lingering_potion: 8
    shulker_bullet: 8
    small_fireball: 8
    snowball: 8
    spectral_arrow: 16
    trident: 16
    wind_charge: 8
    wither_skull: 4
```

With the help of this entry you can set limits to how many entities of specified type can be saved. You should provide a limit for each projectile at least to avoid issues with massive amounts of projectiles being saved and your server crashing on loading that. You can put any entity id here, see the minecraft wiki to find IDs of entities. Please adjust the limit to your liking. Suggested value for all projectiles is around `10`. You can also add other entities by their type names to that list. This config option is not designed to prevent players from making large mob farms.


## Mobs

### [bukkit.yml]

#### spawn-limits

```
Good starting values:

    monsters: 20
    animals: 5
    water-animals: 2
    water-ambient: 2
    water-underground-creature: 3
    axolotls: 3
    ambient: 1
```

The math of limiting mobs is `[playercount] * [limit]`, where "playercount" is current amount of players on the server. Logically, the smaller the numbers are, the less mobs you're gonna see. `per-player-mob-spawn` applies an additional limit to this, ensuring mobs are equally distributed between players. Reducing this is a double-edged sword; yes, your server has less work to do, but in some gamemodes natural-spawning mobs are a big part of a gameplay. You can go as low as 20 or less if you adjust `mob-spawn-range` properly. Setting `mob-spawn-range` lower will make it feel as if there are more mobs around each player. If you are using Paper, you can set mob limits per world in [paper-world configuration].

#### ticks-per

```
Good starting values:

    monster-spawns: 10
    animal-spawns: 400
    water-spawns: 400
    water-ambient-spawns: 400
    water-underground-creature-spawns: 400
    axolotl-spawns: 400
    ambient-spawns: 400
```

This option sets how often (in ticks) the server attempts to spawn certain living entities. Water/ambient mobs do not need to spawn each tick as they don't usually get killed that quickly. As for monsters: Slightly increasing the time between spawns should not impact spawn rates even in mob farms. In most cases all of the values under this option should be higher than `1`. Setting this higher also allows your server to better cope with areas where mob spawning is disabled.

### [spigot.yml]

#### mob-spawn-range

`Good starting value: 3`

Allows you to reduce the range (in chunks) of where mobs will spawn around the player. Depending on your server's gamemode and its playercount you might want to reduce this value along with [bukkit.yml]'s `spawn-limits`. Setting this lower will make it feel as if there are more mobs around you. This should be lower than or equal to your simulation distance, and never larger than your hard despawn range / 16.

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

#### nerf-spawner-mobs

`Good starting value: true`

You can make mobs spawned by a monster spawner have no AI. Nerfed mobs will do nothing. You can make them jump while in water by changing `spawner-nerfed-mobs-should-jump` to `true` in [paper-world configuration].

### [paper-world configuration]

#### despawn-ranges

```
Good starting values:

      ambient:
        hard: 72
        soft: 30
      axolotls:
        hard: 72
        soft: 30
      creature:
        hard: 72
        soft: 30
      misc:
        hard: 72
        soft: 30
      monster:
        hard: 72
        soft: 30
      underground_water_creature:
        hard: 72
        soft: 30
      water_ambient:
        hard: 72
        soft: 30
      water_creature:
        hard: 72
        soft: 30
```

Lets you adjust entity despawn ranges (in blocks). Lower those values to clear the mobs that are far away from the player faster. You should keep soft range around `30` and adjust hard range to a bit more than your actual simulation-distance, so mobs don't immediately despawn when the player goes just beyond the point of a chunk being loaded (this works well because of `delay-chunk-unloads-by` in [paper-world configuration]). When a mob is out of the hard range, it will be instantly despawned. When between the soft and hard range, it will have a random chance of despawning. Your hard range should be larger than your soft range. You should adjust this according to your view distance using `(simulation-distance * 16) + 8`. This partially accounts for chunks that haven't been unloaded yet after player visited them.

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

#### armor-stands.tick

`Good starting value: false`

In most cases you can safely set this to `false`. If you're using armor stands or any plugins that modify their behavior and you experience issues, re-enable it. This will prevent armor stands from being pushed by water or being affected by gravity.

#### armor-stands.do-collision-entity-lookups

`Good starting value: false`

Here you can disable armor stand collisions. This will help if you have a lot of armor stands and don't need them colliding with anything.

#### tick-rates

```
Good starting values:

  behavior:
    villager:
      validatenearbypoi: 60
      acquirepoi: 120
  sensor:
    villager:
      secondarypoisensor: 80
      nearestbedsensor: 80
      villagerbabiessensor: 40
      playersensor: 40
      nearestlivingentitysensor: 40
```

This decides how often specified behaviors and sensors are being fired in ticks. `acquirepoi` for villagers seems to be the heaviest behavior, so it's been greately increased. Decrease it in case of issues with villagers finding their way around.

### [purpur.yml]

#### zombie.aggressive-towards-villager-when-lagging

`Good starting value: false`

Enabling this will cause zombies to stop targeting villagers if the server is below the tps threshold set with `lagging-threshold` in [purpur.yml].

#### entities-can-use-portals

`Good starting value: false`

This option can disable portal usage of all entities besides the player. This prevents entities from loading chunks by changing worlds which is handled on the main thread. This has the side effect of entities not being able to go through portals.

#### villager.lobotomize.enabled

`Good starting value: true`

> This should only be enabled if villagers are causing lag! Otherwise, the pathfinding checks may decrease performance.

Lobotomized villagers are stripped from their AI and only restock their offers every so often. Enabling this will lobotomize villagers that are unable to pathfind to their destination. Freeing them should unlobotomize them.

#### villager.search-radius

```
Good starting values:

          acquire-poi: 16
          nearest-bed-sensor: 16
```

Radius within which villagers will search for job site blocks and beds. This significantly boosts performance with large amount of villagers, but will prevent them from detecting job site blocks or beds that are further away than set value.

---

## Misc

### [spigot.yml]

#### merge-radius

```
Good starting values:

      item: 3.5
      exp: 4.0
```

This decides the distance between the items and exp orbs to be merged, reducing the amount of items ticking on the ground. Setting this too high will lead to the illusion of items or exp orbs disappearing as they merge together. Setting this too high will break some farms, as well as allow items to teleport through blocks. There are no checks done to prevent items from merging through walls (unless Paper's `fix-items-merging-through-walls` setting is activated). Exp is only merged on creation.

#### hopper-transfer

`Good starting value: 8`

Time in ticks that hoppers will wait to move an item. Increasing this will help improve performance if there are a lot of hoppers on your server, but will break hopper-based clocks and possibly item sorting systems if set too high.

#### hopper-check

`Good starting value: 8`

Time in ticks between hoppers checking for an item above them or in the inventory above them. Increasing this will help performance if there are a lot of hoppers on your server, but will break hopper-based clocks and item sorting systems relying on water streams.

### [paper-world configuration]

#### alt-item-despawn-rate

```
Good starting values:

      enabled: true
      items:
        cobblestone: 300
        netherrack: 300
        sand: 300
        red_sand: 300
        gravel: 300
        dirt: 300
        short_grass: 300
        pumpkin: 300
        melon_slice: 300
        kelp: 300
        bamboo: 300
        sugar_cane: 300
        twisting_vines: 300
        weeping_vines: 300
        oak_leaves: 300
        spruce_leaves: 300
        birch_leaves: 300
        jungle_leaves: 300
        acacia_leaves: 300
        dark_oak_leaves: 300
        mangrove_leaves: 300
        cherry_leaves: 300
        cactus: 300
        diorite: 300
        granite: 300
        andesite: 300
        scaffolding: 600
```

This list lets you set alternative time (in ticks) to despawn certain types of dropped items faster or slower than default. This option can be used instead of item clearing plugins along with `merge-radius` to improve performance.

#### redstone-implementation

`Good starting value: ALTERNATE_CURRENT`

Replaces the redstone system with faster and alternative versions that reduce redundant block updates, lowering the amount of logic your server has to calculate. Using a non-vanilla implementation may introduce minor inconsistencies with very technical redstone, but the performance gains far outweigh the possible niche issues. A non-vanilla implementation option may additionally fix other redstone inconsistencies caused by CraftBukkit.

The `ALTERNATE_CURRENT` implementation is based off of the [Alternate Current](https://modrinth.com/mod/alternate-current) mod. More information on this algorithm can be found on their resource page.

#### hopper.disable-move-event

`Good starting value: false`

`InventoryMoveItemEvent` doesn't fire unless there is a plugin actively listening to that event. This means that you only should set this to true if you have such plugin(s) and don't care about them not being able to act on this event. **Do not set to true if you want to use plugins that listen to this event, e.g. protection plugins!**

#### hopper.ignore-occluding-blocks

`Good starting value: true`

Determines if hoppers will ignore containers inside full blocks, for example hopper minecart inside sand or gravel block. Keeping this enabled will break some contraptions depending on that behavior.

#### tick-rates.mob-spawner

`Good starting value: 2`

This option lets you configure how often spawners should be ticked. Higher values mean less lag if you have a lot of spawners, although if set too high (relative to your spawners delay) mob spawn rates will decrease.

#### optimize-explosions

`Good starting value: true`

Setting this to `true` replaces the vanilla explosion algorithm with a faster one, at a cost of slight inaccuracy when calculating explosion damage. This is usually not noticeable.

#### treasure-maps.enabled

`Good starting value: false`

Generating treasure maps is extremely expensive and can hang a server if the structure it's trying to locate is in an ungenerated chunk. It's only safe to enable this if you pregenerated your world and set a vanilla world border.

#### treasure-maps.find-already-discovered

```
Good starting values:
      loot-tables: true
      villager-trade: true
```

Default value of this option forces the newly generated maps to look for unexplored structure, which are usually in not yet generated chunks. Setting this to true makes it so maps can lead to the structures that were discovered earlier. If you don't change this to `true` you may experience the server hanging or crashing when generating new treasure maps. `villager-trade` is for maps traded by villagers and loot-tables refers to anything that generates loot dynamically like treasure chests, dungeon chests, etc.

#### tick-rates.grass-spread

`Good starting value: 4`

Time in ticks between the server trying to spread grass or mycelium. This will make it so large areas of dirt will take a little longer to turn to grass or mycelium. Setting this to around `4` should work nicely if you want to decrease it without the decreased spread rate being noticeable.

#### tick-rates.container-update

`Good starting value: 1`

Time in ticks between container updates. Increasing this might help if container updates cause issues for you (it rarely happens), but makes it easier for players to experience desync when interacting with inventories (ghost items).

#### non-player-arrow-despawn-rate

`Good starting value: 20`

Time in ticks after which arrows shot by mobs should disappear after hitting something. Players can't pick these up anyway, so you may as well set this to something like `20` (1 second).

#### creative-arrow-despawn-rate

`Good starting value: 20`

Time in ticks after which arrows shot by players in creative mode should disappear after hitting something. Players can't pick these up anyway, so you may as well set this to something like `20` (1 second).


### [purpur.yml]

#### dolphin.disable-treasure-searching

`Good starting value: true`

Prevents dolphins from performing structure search similar to treasure maps

#### teleport-if-outside-border

`Good starting value: true`

Allows you to teleport the player to the world spawn if they happen to be outside of the world border. Helpful since the vanilla world border is bypassable and the damage it does to the player can be mitigated.

---

## Helpers

### [paper-world configuration]

#### anti-xray.enabled

`Good starting value: true`

Enable this to hide ores from x-rayers. For detailed configuration of this feature check out [Configuring Anti-Xray](https://docs.papermc.io/paper/anti-xray). Enabling this will actually decrease performance, however it is much more efficient than any anti-xray plugin. In most cases the performance impact will be negligible.

#### nether-ceiling-void-damage-height

`Good starting value: 127`

If this option is greater that `0`, players above the set y level will be damaged as if they were in the void. This will prevent players from using the nether roof. Vanilla nether is 128 blocks tall, so you should probably set it to `127`. If you modify the height of the nether in any way you should set this to `[your_nether_height] - 1`.

---

# Java startup flags
[Vanilla Minecraft and Minecraft server software in version 1.20.5+ requires Java 21 or higher](https://docs.papermc.io/java-install-update). Oracle has changed their licensing, and there is no longer a compelling reason to get your java from them. Recommended vendors are [Adoptium](https://adoptium.net/) and [Amazon Corretto](https://aws.amazon.com/corretto/). Alternative JVM implementations such as OpenJ9 or GraalVM can work, however they are not supported by Paper and have been known to cause issues, therefore they are not currently recommended.

Your garbage collector can be configured to reduce lag spikes caused by big garbage collector tasks. You can find startup flags optimized for Minecraft servers [here](https://docs.papermc.io/paper/aikars-flags) [`SOG`]. Keep in mind that this recommendation will not work on alternative JVM implementations.
It's recommended to use the [flags.sh](https://flags.sh) startup flags generator to get the correct startup flags for your server


# "Too good to be true" plugins

## Plugins removing ground items
Absolutely unnecessary since they can be replaced with [merge-radius](#merge-radius) and [alt-item-despawn-rate](#alt-item-despawn-rate) and frankly, they're less configurable than basic server configs. They tend to use more resources scanning and removing items than not removing the items at all.

## Mob stacker plugins
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all due to the server constantly trying to spawn more mobs. The only "acceptable" use case is for spawners on servers with a large amount of spawners.

## Plugins enabling/disabling other plugins
Anything that enables or disables plugins on runtime is extremely dangerous. Loading a plugin like that can cause fatal errors with tracking data and disabling a plugin can lead to errors due to removing dependency. The `/reload` command suffers from exact same issues and you can read more about them in [me4502's blog post](https://madelinemiller.dev/blog/problem-with-reload/)

# What's lagging? - measuring performance

## mspt
Paper offers a `/mspt` command that will tell you how much time the server took to calculate recent ticks. If the first and second value you see are lower than 50, then congratulations! Your server is not lagging! If the third value is over 50 then it means there was at least 1 tick that took longer. That's completely normal and happens from time to time, so don't panic.
  
## Spark
[Spark](https://spark.lucko.me/) is a plugin that allows you to profile your server's CPU and memory usage. You can read on how to use it [on its wiki](https://spark.lucko.me/docs/). There's also a guide on how to find the cause of lag spikes [here](https://spark.lucko.me/docs/guides/Finding-lag-spikes).

---

# Minecraft exploits and how to fix them
To see how to fix exploits that can cause lag spikes or crashes on a Minecraft server, refer to [here](https://github.com/YouHaveTrouble/minecraft-exploits-and-how-to-fix-them).

[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[server.properties]: https://docs.papermc.io/paper/reference/server-properties
[bukkit.yml]: https://docs.papermc.io/paper/reference/bukkit-configuration
[spigot.yml]: https://docs.papermc.io/paper/reference/spigot-configuration
[paper-global configuration]: https://docs.papermc.io/paper/reference/global-configuration
[paper-world configuration]: https://docs.papermc.io/paper/reference/world-configuration
[purpur.yml]: https://purpurmc.org/docs/Configuration/
