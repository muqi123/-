[Tao](https://engineering.fb.com/2013/06/25/core-data/tao-the-power-of-the-graph/)  

Problem to solve:
1. Most recent data read the most
2. Caching could miss or be inconsistent
3. code for engineer to write is complex 

Three type it write to keep it simple:
1. bool id1, id2 are in relationship
2. Rnage, e.g. what are the items id1 like/liked by
3. Count, e.g. number of items id1 like

Trade-off:
No complex traversals or pattern matching on the graph an optimal.
Division of labor between application servers, data store servers, and the network connecting them.

Cache consistency:
Two tier of caching system, Client -> Follower cache -> Leader cache( responsible for maintain follower consistency within the cluster) -> Database.
Update the cache on success write operation(eventually consistency).  
When user updated the post -> follower cache -> leader cache -> database. After wrting success -> leader cache -> follower cache -> user. Then leader cache also send a refill cache instruction to other caches.( benifit -> idemopotent and save the bytes in case the change is not in that cache)  

DataCenter consistency:
replicate datacenter get the update -> forward to master datacenter leader cache -> write to master database -> master send replicate instructions to replicate datacenter with refill cache instructions -> update the cache in replicate cache

What does leader do:
1. if follower miss the cache, leader response to refill it from database
2. if it hot, usualy leader already has that missing piese, it is fast to send back to follower

Trade-off: TAO losing consistency is a lesser evil than losing availability. Easy to recover if system failed. TAO clients may override the default policy at the expense of higher processing cost and potential loss of availability.

Solve multi-device problem:
Some user stick to a certain cache, user update the feed, after writing successfully in database and updated the cache, we tell people we have success update the cache.