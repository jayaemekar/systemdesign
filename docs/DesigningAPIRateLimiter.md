# Designing an API Rate Limiter

## Problem Statement
Let's create an API Rate Limiter that throttles users based on the amount of requests they're sending.

- Difficulty Level: Medium

### What is a Rate Limiter?
Consider a service that receives a large number of requests but can only handle a certain number of requests per second. To deal with this issue, we'd need some sort of throttling or rate limiting mechanism that would only allow a specific amount of requests per second so that our service could react to them all. At its most basic level, a rate limiter restricts the amount of events a certain object (person, device, IP, etc.) can do in a given time range. For instance:

1. A user can send only one message per second.
2. A user is allowed only three failed credit card transactions per day.
3. A single IP can only create twenty accounts per day.
In general, a rate limiter caps how many requests a sender can issue in a specific time window. It then blocks requests once the cap is reached.

### Why do we need API rate limiting?
Rate limiting protects services from abusive behaviors such as denial-of-service (DOS) assaults, brute-force password attempts, and brute-force credit card transactions, among others. These attacks usually consist of a bombardment of HTTP/S requests that appear to be coming from real people but are actually created by machines (or bots). As a result, these attacks are often harder to detect and can more easily bring down a service, application, or an API.

Rate restriction is also used to prevent revenue loss, infrastructure costs, spam, and online abuse. The following are some circumstances where rate restriction can improve the reliability of a service (or API):

**Misbehaving clients/scripts:**

- Some entities can overload a service by sending a huge number of requests, either purposefully or unintentionally. 
- Another instance is when a user makes a large number of low-priority requests and we want to ensure that they do not interfere with high-priority traffic. 
- Users that send a large number of requests for analytics data, for example, should not be permitted to obstruct critical transactions for other users.

**Security:**

Limiting the number of second-factor attempts (in 2-factor auth) that users are permitted to make, such as the number of times they are permitted to try with a bad password.

**To prevent abusive behavior and poor design practices:** Without API restrictions, client application developers might resort to sloppy development strategies such as repeatedly asking the same information.

**To keep costs and resource use in check:**
Services are typically built for standard input behavior, such as a user making a single post in under a minute. An API may easily push thousands of times per second. The rate limiter gives you more control over service APIs.

**Revenue:**
Certain services may wish to limit operations based on the tier of service provided to its customers, resulting in a revenue model based on rate limitation. For any of the APIs that a service provides, there may be default restrictions. To go above and beyond, the user must purchase higher limits.

To avoid traffic spikiness, make sure the service is available to everyone else.

## Pratice Problem

***Let's get started on the system design solution.***

**If you run into any problems, please see the solution below.**

<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta name="description" content="X-Frame-Bypass: Web Component extending IFrame to bypass X-Frame-Options: deny/sameorigin">
</head>
<body>
    <a href="https://jayaemekar.github.io/praticedesign/" target="_blank">Pratice on full Screen</a>
    <br><br>
	<iframe is="x-frame-bypass" src="https://ej2.syncfusion.com/showcase/angular/diagrambuilder/" width="725" height="500"></iframe>

    <br><br>
    <h2>Hints to solve the problem</h2>

    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#requirements-and-goals-of-the-system" target="_blank">1. Consider functional and non-functional requirements. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#what-are-different-types-of-algorithms-used-for-rate-limiting" target="_blank">2. What are different types of algorithms used for Rate Limiting? </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#high-level-design-for-rate-limiter" target="_blank">3. Consider high level design. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#sliding-window-algorithm" target="_blank">4. How do you create a Sliding Window algorithm and counters? </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#basic-system-design-and-algorithm" target="_blank">5. Think about Basic System Design and Algorithm?</a>
    <br>
    <br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningAPIRateLimiter/#data-sharding-and-caching" target="_blank">6.  Consider Cache and Load Balancing </a>
    <br>
<br><br>
</body>
</html>


## <h1>Solution<h1>

### Requirements and Goals of the System
Our Rate Limiter should meet the following requirements:

**Functional Requirements:**

- Limit the number of requests an entity can make to an API in a given time period, such as 15 per second.
- Because the APIs are accessed via a cluster, the rate restriction must be considered across all servers. 
- When the stated threshold is exceeded inside a single server or across many servers, the user should receive an error message.

**Non-Functional Requirements:**

- The system should have a high level of availability. Because it protects our service from external threats, the rate limiter should always be enabled.
- Our rate limitation should not cause significant delays in the user's experience.

**How to do Rate Limiting?**
- Consumers can access APIs at different rates and speeds, which is known as rate limiting. 
- Throttling is the process of restricting customer access to APIs for a set period of time. 
- Throttling can be set at both the application and API level. The server returns HTTP status "429 - Too many requests" when a throttling limit is exceeded.

**What are different types of throttling?**

The three most common types of throttling utilized by various services are as follows:

**Hard Throttling:** The maximum number of API queries is limited by the throttling limit.

**Soft Throttling:** We can set the API request limit to a specified percentage in this type. If we have a rate limit of 100 messages per minute and a 10% exceed-limit, our rate limiter will allow us to send up to 110 messages per minute.

**Elastic or Dynamic Throttling:** If the system has some resources available, elastic throttling allows the number of requests to exceed the threshold. For example, if a user is only authorized to send 100 messages per minute, we can allow them to send more if there are free resources available in the system.

### What are different types of algorithms used for Rate Limiting?
Following are the two types of algorithms used for Rate Limiting:

**Fixed Window Algorithm:** 

The time window is considered in this algorithm from the beginning to the end of the time unit. For example, regardless of the time window in which the API call was made, a period would be regarded 0-60 seconds for a minute. There are two messages between 0-1 second and three messages between 1-2 seconds in the diagram below. This algorithm will only throttle'm5' if the rate limitation is set to two messages per second.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter.JPG">
  </a>
  </kbd>
</p>

**Rolling Window Algorithm:** 

The time window is calculated using the fraction of the time the request is made plus the time window length in this procedure. If two messages are sent at the 300th and 400th milliseconds of a second, for example, we'll count them as two messages from the 300th millisecond of that second until the 300th millisecond of the next second. We'll throttle'm3' and'm4' in the figure above, keeping two messages per second.

### High level design for Rate Limiter

The Rate Limiter will be in charge of determining which requests are fulfilled by the API servers and which are rejected. When a new request comes in, the Web Server first asks the Rate Limiter whether it should be served or throttled. The request will be delivered to the API servers if it is not throttled.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter1.JPG">
  </a>
  </kbd>
</p>

### Basic System Design and Algorithm

Let’s take the example where we want to limit the number of requests per user. In this instance, we'd record a count of how many requests each unique user has made, as well as a timestamp for when we started counting the requests. We may store it in a hashtable with the 'UserID' as the key and a structure comprising an integer for the 'Count' and an integer for the Epoch time as the value:

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter2.JPG">
  </a>
  </kbd>
</p>

Assume that our rate limitation allows three requests per minute per user, thus everytime a new request arrives, our rate limiter will do the following:

1. If the 'UserID' isn't in the hash-table, add it, set the 'Count' to 1, and change the 'StartTime' to the current time (normalized to a minute) before allowing the request.

2. If CurrentTime – StartTime >= 1 minute, find the record for the 'UserID' and set the 'StartTime' to the current time, 'Count' to 1, and grant the request.

3. If the difference between CurrentTime and StartTime is 1 minute and

    If ‘Count < 3’, increment the Count and allow the request.
    If ‘Count >= 3’, reject the request.


<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter3.JPG">
  </a>
  </kbd>
</p>

**What are some of the problems with our algorithm?**

1. Because we're resetting the 'StartTime' at the end of every minute, this is a **Fixed Window algorithm**, which implies it can possibly enable twice the number of requests every minute. Consider what happens if Kristie submits three requests in the last second of a minute, then sends three more in the first second of the next minute, for a total of six requests in two seconds. A sliding window approach, which we'll describe later, would be the solution to this problem.


<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter4.JPG">
  </a>
  </kbd>
</p>

**2. Atomicity:** The "read-then-write" practice can cause a race issue in a distributed setting. Consider the case when Kristie's current 'Count' is "2" and she makes two more requests. If two distinct processes served each of these requests and read the Count concurrently before either of them modified it, each process would believe Kristie could have one more request and had not reached the rate limit.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter5.JPG">
  </a>
  </kbd>
</p>

If we're storing our key-value in Redis, one way to overcome the atomicity issue is to employ a Redis lock for the duration of the read-update operation. However, this would slow down concurrent requests from the same user and add another layer of complexity. We could use Memcached, but it would have similar issues.

We can have a custom implementation for 'locking' each record if we use a basic hash-table to handle our atomicity problems.

**How much memory would we need to store all of the user data?** 

Let's start with a simple solution in which all of the data is stored in a hash table.

Assume that 'UserID' is 8 bytes long. Let's also suppose that a two-byte 'Count,' which can count up to 65k, is plenty for our needs. Although the epoch time requires four bytes, we can record only the minute and second parts, which can be stored in two bytes. As a result, storing a user's data requires a total of 12 bytes:

                        8 + 2 + 2 = 12 bytes
Assume that each record in our hash table has a 20-byte overhead. If we need to track one million people at any given time, we'll require 32MB of total memory:

                        (12 + 20) bytes * 1 million => 32MB
We'll need 36MB of memory if we think we'll need a 4-byte number to lock each user's record to address our atomicity problems.

This can easily fit on a single server, but we don't want to route all of our traffic through that computer. Furthermore, assuming a rate restriction of 10 requests per second, our rate limiter would have 10 million QPS! 

This is just too much for a single server to handle. In a distributed system, we can probably expect to employ a Redis or Memcached-like solution.All of the data will be stored on remote Redis servers, which all Rate Limiter servers will read (and update) before serving or throttling any requests.

### Sliding Window algorithm
We can keep a sliding window open if we keep track of each user's requests. In our hash-'value' table's column, we may store the timestamp of each request in a Redis Sorted Set.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter6.JPG">
  </a>
  </kbd>
</p>

Let's say our rate limiter allows three requests per minute per user. When a new request arrives, the Rate Limiter will do the following:

1. Remove all timestamps older than "CurrentTime - 1 minute" from the Sorted Set.

2. Count how many elements there are in the sorted set. If this number exceeds our "3" throttling limit, reject the request.

3. Accept the request and add the current time to the sorted set.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter7.JPG">
  </a>
  </kbd>
</p>

**How much memory would we need to store all of the user data for sliding window?** 

Assume that 'UserID' is 8 bytes long. Each epoch will take up 4 bytes. Let's pretend we need to limit requests to 500 per hour. Assume a hash-table overhead of 20 bytes and a Sorted Set overhead of 20 bytes. To store one user's data, we'd require a maximum of 12KB:

            8 + (4 + 20 (sorted set overhead)) * 500 + 20 (hash-table overhead) = 12KB
We're reserving 20 bytes each element in this case. We may assume that in a sorted set, we'll need at least two pointers to keep the items in order — one to the previous element and one to the next. Each pointer on a 64-bit computer will take up 8 bytes. As a result, pointers will require 16 bytes. An extra word (4 bytes) was added to store other overhead.

If we need to track one million users at any time, total memory we would need would be 12GB:

            12KB * 1 million ~= 12GB
Sliding Window Algorithm takes a lot of memory compared to the Fixed Window; this would be a scalability issue. What if we can combine the above two algorithms to optimize our memory usage?

### Sliding Window with Counters

- What if we kept track of request counts for each user throughout numerous set time periods, such as 1/60th the size of our rate limit's time window? 
- When we receive a new request, for example, we can retain a count for each minute and calculate the sum of all counters in the previous hour to establish the throttling limit. 
- This would decrease our RAM use. Let's say we set a rate limit of 500 requests per hour, with an extra 10 requests per minute limit.
- Kristie has surpassed the rate limit when the aggregate of the counters with timestamps in the last hour exceeds the request threshold (500). 
- Furthermore, she is limited to sending ten requests per minute. Because none of the real users would send frequent requests, this would be a sensible and practical consideration. Even if they succeed, retries will be successful because their restrictions are reset every minute.
- Our counters may be stored in a Redis Hash, which provides highly efficient storage for less than 100 keys. Each request sets the hash to expire an hour later by incrementing a counter in the hash. 
- Each 'time' will be converted to a minute.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/apiratemeter8.JPG">
  </a>
  </kbd>
</p>

**How much memory we would need to store all the user data for sliding window with counters?**

Assume that 'UserID' is 8 bytes long. Each epoch will require 4 bytes, whereas the Counter will require 2 bytes. Let's pretend we need to limit requests to 500 per hour. Assume a hash-table overhead of 20 bytes and a Redis hash overhead of 20 bytes. We'll need 60 entries for each user because we'll be keeping a minute-by-minute count. To store one user's data, we'd require a total of 1.6KB:

            8 + (4 + 2 + 20 (Redis hash overhead)) * 60 + 20 (hash-table overhead) = 1.6KB
If we need to track one million users at any time, total memory we would need would be 1.6GB:

            1.6KB * 1 million ~= 1.6GB
So, our ‘Sliding Window with Counters’ algorithm uses 86% less memory than the simple sliding window algorithm.

### Data Sharding and Caching
- To spread the user's data, we can shard based on the 'UserID.' 
- Consistent Hashing should be used for fault tolerance and replication. We can opt to shard each user per API if we want different throttle limitations for distinct APIs. 
- Consider URL Shortener: each user or IP can have a distinct rate limiter for the createURL() and deleteURL() APIs.
- If our APIs are partitioned, a separate (slightly smaller) rate restriction for each API shard would be a reasonable idea. Take our URL Shortener, for example, where we wish to limit each user to no more than 100 short URLs each hour. 
- We can rate limit each partition to allow a user to produce no more than three short URLs per minute in addition to 100 short URLs per hour if we utilize Hash-Based Partitioning for our createURL() API.
- Caching recent active users can provide significant benefits to our system. Before reaching backend servers, application servers can rapidly check if the cache contains the necessary record. 
- By updating all counts and timestamps in cache only, our rate limiter may greatly benefit from the Write-back cache. 
- The persistent storage can be written to at predetermined intervals. We can ensure that the rate limiter adds the least amount of latency to the user's requests this way. 
- When the user has reached their maximum limit and the rate limiter is merely reading data without any updates, the reads can always hit the cache first, which will be tremendously handy.

For our system, the Least Recently Used (LRU) policy may be an appropriate cache eviction policy.

**Should we rate limit by IP or by user?**

Let’s discuss the pros and cons of using each one of these schemes:

**IP:** 

- We limit requests per-IP in this approach; while it's not ideal for distinguishing between 'good' and 'bad' actors, it's still better than having no rate limitation as all. 
- When several users share a single public IP, such as in an internet cafe or smartphone users utilizing the same gateway, IP based throttling becomes a major issue. 
- Throttling can be caused by a single bad user. 
- Another issue could develop with caching IP-based limits: because a hacker has access to a large number of IPv6 addresses from just one computer, it's easy to cause a server to run out of memory tracking IPv6 addresses!

**User:** 

- After user authentication, rate limits can be done on APIs. After being authorized, the user will be given a token to pass along with each request. 
- This ensures that we rate limit against a specific API with a valid authentication token. But what if the login API itself has a rate limit? 
- A hacker could launch a denial of service attack against a user by submitting incorrect credentials up to the limit; after that, the actual user will be unable to log in.

**How about if we combine the above two schemes?**

**Hybrid:** 

- A good strategy would be to implement both per-IP and per-user rate limitation, as both have flaws when performed separately. 
- However, this will result in more cache entries with more details per entry, requiring more memory and storage.