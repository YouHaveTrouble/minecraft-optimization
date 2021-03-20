# Concepts for dummies

This page will help you understand basic concepts behind hosting a minecraft server.

## RAM allocation and Garbage Collection (GC)

### Your startup Xms and Xmx flags should always have the same value.

Consider the following analogy:

> Imagine you're going to school, and you get yourself a single page for notes. Week comes to an end and you see your page is full, but there are bound to be more notes to be made next week, so you go and buy a 16 page notebook. Few weeks go around any you notice your notebook is full, but there is still a lot of time until the end of school, so you go buy 32 page notebook. Repeat this until you get to 9 thousand page notebook. School is finally over. 
  Wouldn't it be way more cheaper and time and resource efficient to buy bigger notebook in the first place?

If your Xms is lower than Xmx your CPU has to take resources to increase the amount of allocated memory each time it hits current allocation limit. By setting equal values for those you allocate all the memory at the start, saving your cpu's time on runtime.

### You shouldn't allocate too much RAM

Allocating too much memory can hurt your server the same way as allocating too little.

Taking the previous analogy from this section, consider this:

> The school year is over and the next one is about to begin shortly. You just need to go through your notes. Turns out a lot of the pages are just silly sketches or stuff that you won't use in the next year. If only you filtered your notes more often during the previous school year while there were less of them!

Smaller amounts of memory are faster to clean up by garbage collection. The more data garbage collection has to filter through, the more time the collection will take, possibly stalling your server.

### Java will always use more memory than you specified

> Imagine you have a storage box, that is filled to the brim. There is absolutely no free space inside. That storage box is your allocated memory. If you wanted to clean it up, you would need to take some things outside of it to check if they are of any use. In order to do that you need the space outside of the storage box.

Java needs extra memory to check what data can be removed. That's why there is usually an overhead to the RAM you allocated via your Xms and Xmx flags. If there isn't enough memory for that, you will get an Out of Memory error.

## Async workload and server oversleep

### Oversleep

Consider the following analogy (by [BillyGalbreath](https://github.com/BillyGalbreath)):

> Imagine you have a job where you work 12 hours a day. The other 12 hours of the day someone else comes in to do that job. the two of you alternate days and nights.
  If things go smoothly, you end your days within your allotted time, or earlier. On bad days you have to stay late and work overtime until the job is finished. Same applies for the night time worker.
  Oversleep is what happens when you wake up and go to work at your regularly scheduled time, but you have to sit and wait for the night time employee to finish their overtime, because they didnt finish their work.
  It sounds silly, I know. Like, why can't you both just work at the same time to finish the job quicker, or why can't you just take over. That's what would happen in real life, but in computing it's a little trickier to do that. So they dont. One has to wait for the other to finish before they can start.

Doing things asynchronously isn't always better, and it's not a magical solution to every problem. In some cases doing things async can even hurt performance. Consider another analogy, also based on [BillyGalbreath's](https://github.com/BillyGalbreath) analogy:

### Async programming

> What is the fastest way of going to your neighbours house? A car is pretty fast, so why not try that. It's definitely faster than you walking, but when you calculate in having to get in the car, buckle up, start the engine, reverse out into the road, drive to the next driveway, pull in, kill the engine, unbuckle and get out, it turns out that just walking would have been faster.

Async operations usually require overhead to take place, like the steps required to enter and exit the car before being able to drive. Sometimes it's just not worth it to use.
