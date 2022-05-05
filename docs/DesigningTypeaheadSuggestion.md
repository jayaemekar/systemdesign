# Designing Typeahead Suggestion
## Problem Statement
Let's create a real-time suggestion service that suggests terms to consumers as they type text into a search box.

- Similar Services: Auto-suggestions, Typeahead search
- Difficulty: Medium

### What is Typeahead Suggestion?
- Users can use typeahead suggestions to find well-known and commonly searched terms. 
- As the user puts into the search box, it attempts to guess the query based on the characters entered and provides a list of possible answers. 
- Typeahead suggestions assist users in better articulating their search queries. 
- It's not about speeding up the search process; instead, it's about guiding people and assisting them in formulating their search query.
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
    <a href="https://ej2.syncfusion.com/showcase/angular/diagrambuilder/" target="_blank">Pratice on full Screen</a>
    <br><br>
	<iframe is="x-frame-bypass" src="https://ej2.syncfusion.com/showcase/angular/diagrambuilder/" width="725" height="500"></iframe>

    <br><br>
    <h2>Hints to solve the problem</h2>

    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#requirements-and-goals-of-the-system" target="_blank">1. Consider functional and non-functional requirements. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#capacity-estimation-and-constraints" target="_blank">2. Estimation of capacity and constraints, such as traffic, bandwidth, and storage. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#system-apis" target="_blank">3. Consider System APIs. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#database-design" target="_blank">4. How do you create a database system? </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#data-partitioning-and-replication" target="_blank">5. What about data replication and partitioning?</a>
    <br>
    <br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningURLShorteningService/#cache" target="_blank">6.  Consider Cache and Load Balancing </a>
    <br>
<br><br>
</body>
</html>


## <h1>Solution<h1>

### Requirements and Goals of the System
**Functional Requirements:**
Our service should provide the top 10 terms starting with whatever the user has written as they type their query.

**Non-function Requirements:**
Suggestions should be displayed in real time. Within 200 milliseconds, the user should be able to see the suggestions.

### Basic System Design and Algorithm
- The issue we're addressing is that we have a large number of'strings' that we need to store in a form that allows customers to search using any prefix. 
- Next terms that match the supplied prefix will be suggested by our service. 
- For example, if the user types in 'cap' and our database has the terms cap, cat, captain, or capital, our system should propose 'cap, captain, and capital.'
- We need a scheme that can efficiently store our data so that it can be queried rapidly because we need to serve a lot of queries with minimal latency. 
- We can't rely on a database for this; we'll have to store our index in memory using a fast data structure.
- The Trie (pronounced "try") is one of the most suited data structures for our needs. 
- A trie is a tree-like data structure for storing phrases, with each node storing a character in the phrase in order. If we need to store 'cap, cat, caption, captain, capital' in the trie, for example, it would look like this:

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/typeahead.JPG">
  </a>
  </kbd>
</p>

Our service can now traverse the trie to the node 'P' to locate all the phrases that begin with this prefix if the user types 'cap' (e.g., cap-tion, cap-ital etc).

To conserve storage capacity, we can merge nodes with only one branch. This is how the above trie can be saved:

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/typeahead1.JPG">
  </a>
  </kbd>
</p>

Should case-insensitive trie be used? Let's pretend our data is case-insensitive for the sake of simplicity and searchability.

**How do I discover the best recommendation?**

- How can we determine the top 10 terms for a particular prefix now that we've found all of the terms for that prefix? 
- One simple option is to keep track of the number of searches that ended at each node; for example, if users searched for 'CAPTAIN' 100 times and 'CAPTION' 500 times, we can keep track of this number by the phrase's last character. 
- We now know that the top most searched word under the prefix 'CAP' is 'CAPTION' if the user inputs 'CAP'. We may then explore the sub-tree beneath it to find the top choices for a given prefix.
- How long will it take to traverse the sub-tree of a given prefix? We should expect a massive tree given the amount of data we need to index. 
- Even traversing a sub-tree might take a long time; for example, the term "system design interview questions" has 30 levels. 
- We need to improve the efficiency of our solution because we have very severe latency constraints.

**Can each node store the top suggestions?**

- This will undoubtedly speed up our searches, but it will necessitate a significant amount of more storage. At each node, we can save the top 10 suggestions and return them to the user. 
- To reach the desired efficiency, we must bear a significant increase in our storage capacity.
- Instead of saving the complete phrase, we can save space by storing only references to the terminal nodes. We must go back using the parent reference from the terminal node to find the proposed phrases. 
- To keep track of top ideas, we'll need to save the frequency with each reference.

**How would we construct this trie?**

- We can efficiently construct our trie from the bottom up. 
- To compute their top suggestions and counts, each parent node will recursively call all of the child nodes. To select their top choices, parent nodes will combine the top proposals from all of their children.

**How should the trie be updated?**

- Assuming five billion searches each day, this equates to about 60K queries per second. 
- If we try to update our trie after every query, it will consume a lot of resources and may slow down our read requests. One way to deal with this might be to update our trie offline after a set period of time.
- We can log new requests as they come in and track their frequency. 
- We can either log every query or sample and log the 1000th query. 
- It's okay to log every 1000th searched term if we don't want to show a term that has been searched less than 1000 times.
- We can put up a Map-Reduce (MR) system to handle all of the logging data on a regular basis, say once an hour. 
- These MR tasks will calculate the frequency of all terms searched in the previous hour. We may then add this additional information to our trie. 
- We can update the trie's current snapshot with all of the new terms and their frequency. We should perform this offline because we don't want update trie requests to obstruct our read queries. 

- There are two possibilities:

- To update the trie offline, we can make a copy of it on each server. After that, we can start using it and throw away the old one.
- Another alternative is to set up each trie server as a master-slave arrangement. While the master is serving traffic, we can update the slave. 
- We can make the slave our new master after the update is complete. We can then upgrade our old master, which will then be able to serve traffic as well.

**How can the frequency of typeahead suggestions be updated?**

- Because the frequencies of our typeahead suggestions are stored with each node, we must also update them! Rather than recreating all search phrases from beginning, we can merely update changes in frequency. 
- If we want to maintain track of all the phrases searched in the last 10 days, we'll have to deduct the counts from the time period that is no longer included and add the counts for the new time period. Based on the Exponential Moving Average (EMA) of each term, we can add and subtract frequencies. 
- The most recent data is given more weight in EMA. The exponentially weighted moving average is another name for it.
- We'll proceed to the phrase's terminal node and boost its frequency after we've added a new term to the trie.
- Because each node saves the top 10 searches, it's likely that this search term leaped into the top 10 queries of a few additional nodes. 
- The top 10 queries of those nodes must then be updated. 
- We must return from the node all the way to the root. We check if the current query is in the top 10 for each parent.
- If so, we update the corresponding frequency. 
- If not, we check if the current query’s frequency is high enough to be a part of the top 10. 
- If so, we insert this new term and remove the term with the lowest frequency.

**How can we get a term out of the trie?**

Let's imagine we need to eliminate a term from the trial due to a legal concern, hatred, or piracy, for example. When the normal update occurs, we can totally delete such phrases from the trie; in the meantime, we can add a filtering layer to each server that will remove any such terms before transmitting them to users.

**What different ranking criteria may there be for suggestions?**

In addition to a simple count, other criteria such as freshness, user location, language, demographics, personal history, and so on must be considered when ranking phrases.

### Permanent Storage of the Trie

**How can we keep trie in a file so that we can easily reconstruct our trie when the system restarts?**

Periodically, we can take a snapshot of our trie and save it to a file. If the server goes down, we'll be able to reconstruct the trie. Starting with the root node, we can save the trie level by level. We can keep track of the characters each node includes and how many children it has. We should put all of the children of each node right after it. Let's pretend we've got the following trie:

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/typeahead2.JPG">
  </a>
  </kbd>
</p>

This trie will be stored in a file using the above-mentioned scheme: "C2,A2,R1,T,P,O1,D." We can easily reconstruct our trie with this information.

We don't store top ideas and their counts with each node, as you may have seen. It's difficult to preserve this information since, because our trie is organized top down, we don't have any child nodes generated before the parent, therefore there's no convenient method to keep track of their references. This requires recalculating all of the top terms with counts. This can be done as the trie is being constructed. Each node will calculate and provide its top recommendations to its parent. To determine its top suggestions, each parent node will combine the findings of all of its offspring.

### Scale Estimation
If we establish a service on the same scale as Google, we can expect 5 billion searches each day, or about 60K requests per second.

We can predict that just 20% of the 5 billion inquiries will be unique due to the large number of duplicates. We can exclude a lot of less often searched queries if we only want to index the top 50% of search phrases. Assume there are 100 million distinct phrases for which we wish to create an index.

**Storage Estimation:**
If each query has an average length of 3 words and a word has an average length of 5 characters, the average query size will be 15 characters. If a character takes two bytes to store, an average query will take 30 bytes to store. So, in total, we'll require:

                    100 million * 30 bytes => 3 GB
Every day, we can expect some growth in this data, but we should also remove some terms that are no longer searched. If we assume that we receive 2% new requests per day and that we keep our index for the last year, total storage should be:

                    3GB + (0.02 * 3 GB * 365 days) => 25 GB
### Data Partition
Even though our index can easily fit on a single server, we can still split it to fulfill our efficiency and latency requirements. How can we split our data efficiently so that it may be distributed across numerous servers?

**a. Range Based Partitioning:**

- What if we divided our phrases into groups based on their first letter? As a result, we save all phrases beginning with the letter 'A' in one partition, those beginning with the letter 'B' in another, and so on. 
- We can even merge a few letters that aren't used very often into one division. This partitioning scheme should be created statically so that we can always store and search phrases in a predictable manner.
- The biggest issue with this strategy is that it can result in unbalanced servers, such as if we decide to place all terms beginning with the letter 'E' into one partition, only to discover later that we have too many terms beginning with the letter 'E' to fit into one partition.
- We can see that the problem described above will occur with any statically stated scheme. It is impossible to determine whether each of our partitions will fit on a single server in a static manner.

**b. Partition based on the maximum capacity of the server:**

Let's imagine we divide our trie according to the servers' maximum memory capacity. We can continue to store data on a server as long as it has memory. We break our partition there to allocate that range to this server and go on to the next server to continue this process whenever a sub-tree cannot fit into a server. 

Assume that our first trie server can store all terms from 'A' to 'AABC,' implying that our next server will store phrases from 'AABD' onwards. If our second server has enough space to store 'BXA,' the third server will begin with 'BXB,' and so on. To conveniently retrieve this partitioning strategy, we can keep a hash table:

- Server 1, A-AABC
- Server 2, AABD-BXA
- Server 3, BXB-CDA

If the user types 'A,' we must query both server 1 and server 2 to discover the top choices. We still have to query servers 1 and 2 when the user types 'AA,' but when the user types 'AAA,' we only have to query server 1.

In front of our trie servers, we can put a load balancer that can store this mapping and divert traffic. Also, if we're querying many servers, we'll either have to merge the results on the server side to get the overall top results, or we'll have to rely on our clients to do it. 

If we wish to achieve this on the server side, we'll need to add another layer of servers (let's call them aggregators) between load balancers and trie servers. The top results from several trie servers will be returned to the client by these servers.

Partitioning based on maximum capacity might still lead to hotspots; for example, if there are a lot of queries for terms that start with 'cap,' the server that holds it will be under a lot of strain relative to others.

**c. Partition based on the hash of the term:**

Each phrase will be passed via a hash function, which will generate a server number, which will be used to store the word. As a result, our word distribution will be random, reducing hotspots. The downside of this technique is that we have to ask all of the servers for typeahead suggestions for a term and then aggregate the responses.

### Cache
- We must understand that caching the most often searched terms will be incredibly beneficial to our business. A small fraction of inquiries will be responsible for the majority of the traffic. 
- Separate cache servers can be placed in front of the trie servers to store the most commonly searched phrases and typeahead suggestions. Before hitting the trie servers, application servers should check these cache servers to see if they have the desired searched terms. 
- This will allow us to travel the tri faster.

We can also create a simple Machine Learning (ML) model that uses simple counting, personalisation, or trending data to forecast interaction on each proposal and cache these terms in advance.

### Load Balancer and Replication
Our trie servers should have replicas for load balancing as well as fault tolerance. A load balancer that maintains track of our data partitioning method and redirects traffic depending on prefixes is also required.

### Fault Tolerance
What happens if one of the trie servers goes down? We can have a master-slave arrangement, as mentioned previously; if the master dies, the slave can take over after failover. Any server that restarts can recreate the trie using the most recent snapshot.

### Typeahead Client
On the client side, we can make the following changes to improve the user experience:

1. The client should only attempt to contact the server if the user has been idle for 50 milliseconds.
2. The client can cancel in-progress requests if the user is constantly typing.
3. The client can initially wait till the user types a few characters.
4. Clients can save time by pre-fetching some data from the server.
5. Clients can keep track of recent suggestions locally. The rate of reuse in recent history is extremely high.
6. One of the most crucial things is establishing an early connection with the server. The client can establish a connection with the server as soon as the user visits the search engine page. As a result, the client does not waste time establishing the connection when the user types the first character.
7. For efficiency, the server can push some of their cache to CDNs and Internet Service Providers (ISPs).

### Personalization
Users will receive typeahead suggestions based on previous searches, location, language, and other factors. We can save each user's personal history on the server separately and cache it on the client. Before transmitting the final set to the user, the server can include these customised phrases. Personal searches should always take precedence over others.