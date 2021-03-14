# Concepts for dummies

This page will help you understand basic concepts behind hosting a minecraft server.

## RAM allocation and Garbage Collection (GC)

#### Your startup Xms and Xmx flags should always have the same value.

Consider the following analogy:

> Imagine you're going to school, and you get yourself a single page. week comes to an end and you see your page is full, but there are bound to be more notes to be made next week, so you go and buy a 16 page notebook. Few weeks go around any you notice your notebook is full, but there is still a lot of time until the end of school, so you go buy 32 page notebook. Repeat this until you get to 9 thousand page notebook. School is finally over. 
  Wouldn't it be way more cheaper and time and resource efficient to buy bigger notebook in the first place?

If you Xms is lower than Xmx your CPU has to take resources to increase the amount of allocated memory each time it hits current allocation limit. By setting equal values for those you allocate all the memory at the start, saving your cpu's time on runtime.

#### You shouldn't allocate too much RAM

Allocating too much memory can hurt your server the same way as allocating too little.

Taking the previous analogy from this section, consider this:

> The school year is over and the next one is about to begin shortly. You just need to go through your notes. Turns out a lot of the pages are just silly sketches or stuff that you won't use in the next year. If only you filtered your notes more often during the previous school year while there were less of them!

Smaller amounts of memory are faster to clean up by garbage collection. The more data garbage collection has to filter through, more time the collection will take, possibly stalling your server.