# Design a distributed key value caching system, like Memcached or Redis.

## Features:
This is the first part of any system design interview, coming up with the features which the system should support. As an interviewee, you should try to list down all the features you can think of which our system should support. Try to spend around 2 minutes for this section in the interview. You can use the notes section alongside to remember what you wrote.


<details>
  <summary><h4>Q: What is the amount of data that we need to cache? </h4></summary>
  
A: Let's assume we are looking to cache on the scale of Google or Twitter. The total size of the cache would be a few TBs.
</details>

<details>
  <summary><h4>Q: What should be the eviction strategy? </h4></summary>
  
A: It is possible that we might get entries when we would not have space to accommodate new entries. In such cases, we would need to remove one or more entries to make space for the new entry.
</br></br>
     <b>Which entry should we evict?</b> There are multiple strategies possible. Following are a few standard ones :
</br></br>
<b>FIFO ( First in, first out ) :</b>The entry that was first added to the queue would be evicted/removed first. In other words, Elements are evicted in the same order as they come in. This is the simplest to implement and performs well when either, access pattern is completely random ( All elements are equally probably to be accessed ) OR if the use of an element makes it less likely to be used in the future.
</br></br>
<b>LRU ( Least Recently Used ) :</b> Default. As the name suggests, the element evicted is the ones which has the oldest last access time. The last accessed timestamp is updated when an element is put into the cache or an element is retrieved from the cache with a get call. This is by far the most popular eviction strategy.
</br></br>
<b>LFU ( Least Frequently Used ) :</b> Again, as the name suggests, every entry has a frequency associated with it. At the time of eviction, the entry with lowest frequency is evicted. This is most effective in cases when most of access is limited to a very small portion of the data ( pareto distribution ).
</br></br>
As you can see, every eviction strategy has its own set of cases when they are most effective. The choice of which eviction strategy to choose is largely dependent on the expected access pattern. We will go with the default here which is LRU.
</br>
</details>

<details>
  <summary><h4>Q. What should be the access pattern for the given cache? </h4></summary>
  
A: There are majorly three kinds of caching systems :
</br></br>
<b>
Write through cache :</b> This is a caching system where writes go through the cache and write is confirmed as success only if writes to DB and the cache BOTH succeed. This is really useful for applications which write and re-read the information quickly. However, write latency will be higher in this case as there are writes to 2 separate systems.
</br></br>
<b>
Write around cache :</b> This is a caching system where write directly goes to the DB. The cache system reads the information from DB incase of a miss. While this ensures lower write load to the cache and faster writes, this can lead to higher read latency incase of applications which write and re-read the information quickly.
</br></br>
<b>
Write back cache :</b> This is a caching system where the write is directly done to the caching layer and the write is confirmed as soon as the write to the cache completes. The cache then asynchronously syncs this write to the DB. This would lead to a really quick write latency and high write throughput. But, as is the case with any non-persistent / in-memory write, we stand the risk of losing the data incase the caching layer dies. We can improve our odds by introducing having more than one replica acknowledging the write ( so that we don???t lose data if just one of the replica dies ).
</br>
</details>

## Estimations:

This is usually the second part of a design interview, coming up with the estimated numbers of how scalable our system should be. Important parameters to remember for this section is the number of queries per second and the data which the system will be required to handle.
Try to spend around 5 minutes for this section in the interview.

**Total cache size : Let's say 30TB as discussed earlier.**


<details>
  <summary><h4>Q: What is the kind of QPS we expect for the system? </h4></summary>
  
A: This estimation is important to understand the number of machines we will need to answer the queries. For example, if our estimations state that a single machine is going to handle 1M QPS, we run into a high risk of high latency / the machine dying because of queries not being answered fast enough and hence ending up in the backlog queue.
Again, let's assume the scale of Twitter / Google. We can expect around 10M QPS if not more.
</details>


<details>
  <summary><h4>Q: What is the number of machines required to cache? </h4></summary>
  
A: A cache has to be inherently of low latency. Which means all cache data has to reside in main memory.
A production level caching machine would be 72G or 144G of RAM. Assuming beefier cache machines, we have 72G of main memory for 1 machine. Min. number of machine required = 30 TB / 72G which is close to 420 machines.
Do know that this is the absolute minimum. Its possible we might need more machines because the QPS per machine is higher than we want it to be.
</details>

## Design Goals:

**Latency** - Is this problem very latency sensitive (Or in other words, Are requests with high latency and a failing request, equally bad?). For example, search typeahead suggestions are useless if they take more than a second.

**Consistency** - Does this problem require tight consistency? Or is it okay if things are eventually consistent?

**Availability** - Does this problem require 100% availability?
There could be more goals depending on the problem. It's possible that all parameters might be important, and some of them might conflict. In that case, you???d need to prioritize one over the other.

<details>
  <summary><h4>Q: Is Latency a very important metric for us? </h4></summary>
  
A: Yes. The whole point of caching is low latency.
</details>

<details>
  <summary><h4>Q: Consistency vs Availability? </h4></summary>
  
A: Unavailability in a caching system means that the caching machine goes down, which in turn means that we have a cache miss which leads to a high latency.
 As said before, we are caching for a Twitter / Google like system. When fetching a timeline for a user, I would be okay if I miss on a few tweets which were very recently posted as long as I eventually see them in reasonable time. 
  Unavailability could lead to latency spikes and increased load on DB. Choosing from consistency and availability, we should prioritize for availability. 
</details>

## Skeleton of the design:

The next step in most cases is to come up with the barebone design of your system, both in terms of API and the overall workflow of a read and write request. Workflow of read/write request here refers to specifying the important components and how they interact. Try to spend around 5 minutes for this section in the interview.
Important : Try to gather feedback from the interviewer here to indicate if you are headed in the right direction.

## Deep Dive:
Lets dig deeper into every component one by one. Discussion for this section will take majority of the interview time(20-30 minutes).


<details>
  <summary><h4>Q: How would a LRU cache work on a single machine which is single threaded?</h4><br></br><h4>Q: What if we never had to remove entries from the LRU cache because we had enough space, what would you use to support and get and set? </h4></summary>
  
A: A simple map / hashmap would suffice.
</details>

<details>
  <summary><h4>Q: How should we modify our approach if we also have to evict keys at some stage?</h4></summary>
  
A: We need a data structure which at any given instance can give me the least recently used objects in order. Let's see if we can maintain a linked list to do it. We try to keep the list ordered by the order in which they are used.
So whenever, a get operation happens, we would need to move that object from a certain position in the list to the front of the list. Which means a delete followed by insert at the beginning. Insert at the beginning of the list is trivial. How do we achieve erase of the object from a random position in least time possible? How about we maintain another map which stores the value to the corresponding linked list node.
Ok, now when we know the node, we would need to know its previous and next node in the list to enable the deletion of the node from the list. We can get the next in the list from next pointer ? What about the previous node ? To encounter that, we make the list doubly linked list.

<br></br>
A: Since we only have one thread to work with, we cannot do things in parallel. So we will take a simple approach and implement a LRU cache using a linked list and a map. The Map stores the value to the corresponding linked list node and is useful to move the recently accessed node to the front of the list.
</details>

<details>
  <summary><h4>Q: How would a LRU cache work on a single machine which is multi threaded ?</h4><br></br>
  <h4>Q: How would you break down cache write and read into multiple instructions?</h4></summary>
  
A:</br>
<b>Read path :</b> Read a value corresponding to a key. This requires :</br></br>
    &nbsp;&nbsp;Operation 1 : A read from the HashMap and then,</br>
    &nbsp;&nbsp;Operation 2 : An update in the doubly LinkedList</br></br>

<b>Write path : </b>Insert a new key-value entry to the LRU cache. This requires :</br></br>
&nbsp;If the cache is full, then</br>
   &nbsp;&nbsp; Operation 3: Figure out the least recently used item from the linkedList</br>
   &nbsp;&nbsp; Operation 4: Remove it from the hashMap</br>
   &nbsp;&nbsp; Operation 5: Remove the entry from the linkedList.</br>
   &nbsp;&nbsp; Operation 6: Insert the new item in the hashMap</br>
   &nbsp;&nbsp; Operation 7: Insert the new item in the linkedList.</br>
</details>
<details>
  <summary><h4>Q: How would you prioritize above operations to keep latency to a minimum for our system?</h4></summary>
  
A: As is the case with most concurrent systems, writes compete with reads and other writes. That requires some form of locking when a write is in progress. We can choose to have writes as granular as possible to help with performance.
Read path is going to be highly frequent. As latency is our design goal, Operation 1 needs to be really fast and should require minimum locks. Operation 2 can happen asynchronously. Similarly, all of the write path can happen asynchronously and the client???s latency need not be affected by anything other than Operation 1. Let's dig deeper into Operation 1. What are the things that Hashmap is dealing with?
Hashmap deals with Operation 1, 4 and 6 with Operation 4 and 6 being write operations. One simple, but not so efficient way of handling read/write would be to acquire a higher level Read lock for Operation 1 and Write lock for Operation 4 and 6.
However, Operation 1 as stressed earlier is the most frequent ( by a huge margin ) operation and its performance is critical to how our caching system works.
</details>

<details>
  <summary><h4>Q: How would you implement HashMap? </h4></summary>
  
A: The HashMap itself could be implemented in multiple ways. One common way could be hashing with linked list (colliding values linked together in a linkedList) :
 Let's say our hashmap size is N and we wish to add (k,v) to it 
  Let H = size N array of pointers with every element initialized to NULL 
  For a given key k, generate g = hash(k) % N newEntry = LinkedList Node with value = v newEntry.next = H[g] 
 H[g] = newEntry    

More details at https://en.wikipedia.org/wiki/Hash_table
Given this implementation, we can see that instead of having a lock on a hashmap level, we can have it for every single row. This way, a read for row i and a write for row j would not affect each other if i != j. Note that we would try to keep N as high as possible here to increase granularity.
</details>

