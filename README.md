
<h1>Minecraft server optimization guide</h1>

<p>Guide for version 1.16.4</p>
<p>Based on <a href="https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/">this guide</a> and other sources (all of them are linked throughout the guide when relevant).</p>

<h2>Intro</h2>
<p>
To begin I'd like to mention that this is not some sort of "magical cure". 
There are no guides that will tell you exactly what to do for your server. 
This is meant to set you in the right direction of finding your server's "sweet spot".</p>

<h2>Server jar</h2>
<p>
Choice of server jar can make huge difference in performance and api possibilities. There are currently multiple viable popular server jars,
but there are also a few that you should stay away from for various reasons.
</p>
My recommendation is:
<ul>
<li><a href="https://github.com/PaperMC/Paper">Paper</a> - Most popular server software on latest minecraft version</li>
<li><a href="https://github.com/Spottedleaf/Tuinity">Tuinity</a> - Paper fork improving performance with little to no consequences</li>
<li><a href="https://github.com/pl3xgaming/Purpur">Purpur</a> - Tuinity fork that gives you way more configurability and extra features</li>
</ul>
You shoud stay away from:
<ul>
<li>Yatopia - "The combined power of Paper forks for maximum instability and unmaintainablity!" - Messy, tossed salad of people that haven't even really understood the patch system and destroys functionality of some of these forks. - <a href="https://github.com/KennyTV/list-of-shame">KennyTV's list of shame</a>.</li>
<li>Any paid server jar that claims async anything - 99.99% of being a scam.</li>
<li>Bukkit/Craftbukkit/Spigot - Extremely outdated and not up to par with paper+ performance.</li>
</ul>

<h2>Map pregen</h2>
<p>
Map pregeneration is one of the most important steps to have lag-free server. 
In modern versions chunk generation is extremely slow and even servers on best 
hardware can grind into a halt. You can use plugin such as <a href="https://github.com/pop4959/Chunky">chunky</a>
to pregenerate the world. Remember to also set up a world border so your players don't generate new chunks while the server is open to the public!
Pregeneration of the map can take hours (it depends on a radius you set in the pregen plugin).
</p>
<p>
It's key to remember that overworld, nether and the end have separate world borders and you have to set it up for each world.
Remember that nether dimension is usually 8x smaller than overworld, because if you set worldborder wrong your players might end up
outside of world border!
</p>

<h2>Configurations</h2>

<h3>server.properties</h3>

<h4>network-compression-threshold</h4>
<b>default:</b> 256<br>
<b>optimized:</b> Standalone(512) BungeeCord(-1)<br>
<b>explanation:</b><br>
This option caps the size of a packet before the server attempts to compress it. Setting it higher can save some 
resources at the cost of bandwidth, setting it to -1 disables it. If your server is in a network with the proxy on 
localhost or the same datacenter (<2 ms ping), disabling this (-1) will be beneficial.

<hr>
<h3>bukkit.yml</h3>

<h4>spawn-limits</h4>
<b>default:</b> monsters:70, animals:10, water-animals:15, ambient:15<br>
<b>optimized:</b> monsters:45, animals:8, water-animals:3, ambient:1<br>
<b>explanation:</b><br>
Lower values mean less mobs. Less mobs is less lag in general, but you want to balance it with player quality of life,
finding mobs in the world is big part of gameplay. More detailed explanation of how limits are calculated can be found in <a href="https://www.spigotmc.org/attachments/bukkit-spawn-limits-pdf.444347/">this pdf</a>.

<h4>chunk-gc.period-in-ticks</h4>
<b>default:</b> 600<br>
<b>optimized:</b> 400<br>
<b>explanation:</b><br>
This decides how often vacant chunks are unloaded. Ticking fewer chunks means less TPS consumption.

<h4>ticks-per.monster-spawns</h4>
<b>default:</b> 1<br>
<b>optimized:</b> 4<br>
<b>explanation:</b><br>
This sets how often (in ticks) the server attempts to spawn a monster. Slighty increasing the time between spawns should not impact spawn rates.

<hr>
<h3>spigot.yml</h3>

<h4>max-tick-time</h4>
<b>default:</b> tile:50, entity:50<br>
<b>optimized:</b> tile:1000, entity:1000<br>
<b>explanation:</b><br>
Setting those to optimized values disables this feature. 
You can read why it should be disabled <a href="https://aikar.co/2015/10/08/spigot-tick-limiter-dont-use-max-tick-time/">here</a>.

<h4>view-distance</h4>
<b>default:</b> default<br>
<b>optimized:</b> 3<br>
<b>explanation:</b><br>
Actual view distance should be set low due to the fact that less chunks will be ticked and paper's no-tick-view-distance
lets you send more chunks to player that are actually ticked.

<h4>mob-spawn-range</h4>
<b>default:</b> 8<br>
<b>optimized:</b> 2<br>
<b>explanation:</b><br>
This usually should be 1 less than view-distance. You can experiment with other values when changing bukkit mob caps.

<h4>entity-activation-range</h4>
<b>default:</b> animals:32, monsters:32, raiders: 48, misc:16<br>
<b>optimized:</b> animals:16, monsters:24, raiders: 48, misc:8<br>
<b>explanation:</b><br>
Entities past this range will be ticked less often. Avoid setting this too low or you might break mob behavior (mob aggro, raids, etc).

<h4>tick-inactive-villagers</h4>
<b>default:</b> true<br>
<b>optimized:</b> false<br>
<b>explanation:</b><br>
Enabling this prevents the server from ticking villagers outside the activation range. Villager tasks in 1.14+ are very heavy.

<h4>merge-radius</h4>
<b>default:</b> item:2.5, exp:3.0<br>
<b>optimized:</b> item:3.0, exp:6.0<br>
<b>explanation:</b><br>
This will decide the distance between the items to be merged, reducing the amount of items ticking on the ground.
Merging will lead to the illusion of items disappearing as they merge together. A minor annoyance.

<h4>nerf-spawner-mobs</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
When enabled, mobs from spawners will not have AI (will not swim/attack/move). This is big TPS savings on servers with mob 
farms, but also messes with their behavior.

<hr>
<h3>paper.yml</h3>
Most of the settings in this file can be configured per-world. See 
<a href="https://www.spigotmc.org/attachments/per-world-guide-pdf.444348/">this pdf</a> for details.

<h4>max-auto-save-chunks-per-tick</h4>
<b>default:</b> 24<br>
<b>optimized:</b> 8<br>
<b>explanation:</b><br>
Slows down incremental world saving spreading the task over time even more for better average performance. You might want 
to set this higher with more than 20-30 players, because if incremental save can't finish in time bukkit will automatically
save leftover chunks at once and begin the process again.

<h4>per-player-mob-spawns</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
By default mob limits are counted for the entire server which means mobs might end up being distributed unevenly between 
online players. This option enables per-player mob limits, meaning all players can get approximately the same number of 
mobs around them regardless of number of online players. Enabling this option also allows you to lower `spawn-limits` in 
`bukkit.yml` since those are optimized for per-server mob limits.

<h4>optimize-explosions</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Faster explosion alghoritm with no impact on gameplay.

<h4>max-entity-collisions</h4>
<b>default:</b> 8<br>
<b>optimized:</b> 2<br>
<b>explanation:</b><br>
Less collisions calculation per entity.

<h4>grass-spread-tick-rate</h4>
<b>default:</b> 1<br>
<b>optimized:</b> 4<br>
<b>explanation:</b><br>
Time in ticks before server tries to spread grass/mycelium. No gameplay impact in most cases.

<h4>despawn-ranges</h4>
<b>default:</b> soft: 32, hard: 128<br>
<b>optimized:</b> soft: 28, hard: 48<br>
<b>explanation:</b><br>
Lower ranges clear background mobs and allow more to be spawned in areas with player traffic. This further reduces the 
gameplay impact of reduced spawning (bukkit.yml). Values adjusted for view-distance: 3.

<h4>hopper.disable-move-event</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
This will significantly reduce hopper lag by preventing InventoryMoveItemEvent being called for EVERY slot in a container.
<b>Do not enable if you use plugins that listen to this event!</b>

<h4>non-player-arrow-despawn-rate</h4>
<b>default:</b> -1<br>
<b>optimized:</b> 20<br>
<b>explanation:</b><br>
Makes arrows shot by mobs disappear after 1 second after hitting. 

<h4>creative-arrow-despawn-rate</h4>
<b>default:</b> -1<br>
<b>optimized:</b> 20<br>
<b>explanation:</b><br>
Makes arrows shot by players in creative disappear after 1 second after hitting. 

<h4>prevent-moving-into-unloaded-chunks</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Prevents players from entering an unloaded chunk (due to lag), which causes more lag. The true setting will set them back 
to a safe location instead.

<h4>use-faster-eigencraft-redstone</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Alternative, faster redstone system. Reduces redundant redstone updates by nearly 95%.

<h4>alt-item-despawn-rate.enabled</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
This option lets you despawn selected items faster than default despawn rate. You can add things like cobblestone, netherrack 
etc. to the list and make them despawn after ~20 seconds (400 ticks).

<h4>enable-treasure-maps</h4>
<b>default:</b> true<br>
<b>optimized:</b> false<br>
<b>explanation:</b><br>
Generating treasure maps is extremely expensive and can hang a server if the structure it's trying to locate is really 
far away.

<h4>viewdistances.no-tick-view-distance</h4>
<b>default:</b> -1<br>
<b>optimized:</b> 8<br>
<b>explanation:</b><br>
This allows players to see further without ticking as many chunks as regular view-distance would. Although it's not really
heavy on the server keep in mind that sending more chunks will affect bandwidth.

<h4>projectile-load-save-per-chunk-limit</h4>
<b>default:</b> -1<br>
<b>optimized:</b> 8<br>
<b>explanation:</b><br>
Limits the amount of projectiles that can be saved in a chunk. This prevents issues that arise with lower view-distance
like players throwing massive amounts of snowballs into unloaded chunk that has a potential to crash your server on 
loading of that chunk.

<h4>anti-xray.enabled</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Hides ores from x-rayers. For detailed configuration of this feature check out 
<a href="https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e">Stonar96's recommended settings</a>.

<hr>
<h3>purpur.yml</h3>
Only applicable for purpur.

<h4>use-alternate-keepalive</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Alternate system for keepalive packets so players with bad connection don't get timed out as often.

<h4>dont-send-useless-entity-packets</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Prevent the server from sending empty position change packets (by default server sends move packet for each entity
even if the entity hasn't moved)

<h4>gameplay-mechanics.player.teleport-if-outside-border</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Teleport the player to the world spawn if they happen to be outside of the world border. This will help, because vanilla
world border is bypassable and the damage it does to the player can be mitigated.

<h4>gameplay-mechanics.player.entities-can-use-portals</h4>
<b>default:</b> true<br>
<b>optimized:</b> false<br>
<b>explanation:</b><br>
Disables portal usage of all entities besides player. This potentially fixes a dupe* and prevents entities changing 
worlds loading chunks on main thread.<br>
*more sources needed.

<h4>mobs.dolphin.disable-treasure-searching</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Prevents dolphins from performing structure search similiar to the one that treasure maps do.

<h4>mobs.zombie.aggressive-towards-villager-when-lagging</h4>
<b>default:</b> false<br>
<b>optimized:</b> true<br>
<b>explanation:</b><br>
Zombies stop targetting villagers when tps is under lag treshold. This saves the server precious time of calculating 
paths for zombies that are not targetting players.

<h2>Java startup flags</h2>
<a href="https://papermc.io/forums/t/java-11-mc-1-17-and-paper/5615">Paper and its forks in upcoming version 1.17 will require Java 11 (LTS) or higher</a>. Good resolution 2021 to finally update your version of Java!(or at least inform your host so they can handle the migration).
JVM can be configured to reduce lag spikes caused by big garbage collector tasks. You can find
startup flags optimized for minecraft servers <a href="https://mcflags.emc.gs/">here</a>.

<h2>"Performance" plugins</h2>

<h3>Plugins removing ground items</h3>
<p>
Absolutely unnecessary, can be replaced with spigot configuration 
(see <a href="https://github.com/YouHaveTrouble/minecraft-optimization#merge-radius">merge radius</a> and <a href="https://github.com/YouHaveTrouble/minecraft-optimization#alt-item-despawn-rateenabled">alt-item-despawn-rate</a>)
and frankly, they're less configurable than basic server configs. The fact that they usually use more resources to scan 
and remove items than that items if they would be left alone doesn't help either.
</p>

<h3>Mob stacker plugins</h3>
<p>
It's really hard to justify using one. Stacking naturally spawned entities causes more lag than not stacking them at all
due to server constantly trying to spawn more mobs. Only "acceptable" use case is spawner mobs on servers with large amount of spawners.
</p>

<h2>What's lagging? - measuring performance</h2>
<h3>mspt</h3>
<p>
Paper offers a `/mspt` command that will tell you how much time server took to calculate recent ticks. If the first and 
second value you see are lower than 50, then congratulations! Your server is not lagging! If third value is over 50 it 
means there was at least 1 tick that took longer, that's completely normal and happens from time to time, don't panic.
</p>
<h3>timings</h3>
<p>
Great way to see what might be going on when your server is lagging are timings. Timings is a tool that lets you see 
exactly what tasks are taking the longest. It's the most basic troubleshooting tool and if you ask for help regarding 
lag you will most likely be asked for your timings.
</p>
<p>
To get timings of your server you just need to execute `/timings paste` command and click the link you're provided with.
You can share this link with other people to let them help you. It's also easy to misread if you don't know what you're 
doing. There is a detailed <a href="https://www.youtube.com/watch?v=T4J0A9l7bfQ">video tutorial by Aikar</a> on how to 
read them.
</p>

<h3>spark</h3>
<p>
 <a href="https://github.com/lucko/spark">Spark</a> is a plugin that allows you to profile your servers CPU and memory usage.
 You can read on how to use it <a href="https://github.com/lucko/spark/wiki/Commands">on its wiki</a>.
</p>

