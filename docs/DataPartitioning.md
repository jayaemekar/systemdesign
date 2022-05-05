# Data Partitioning

- Data partitioning is a method of dividing a large database (DB) into multiple smaller sections. 
- It is the process of dividing a database/table among numerous machines in order to increase an application's manageability, performance, availability, and load balancing.
- The reason for data partitioning is that after a certain scalability point, scaling horizontally by adding more machines is cheaper and more viable than scaling vertically by adding beefier servers.

## Partitioning Methods
There are a variety of approaches that can be used to divide an application database into many smaller databases. Three of the most common methods employed by various large-scale applications are listed below.

**Horizontal partitioning** is a technique for dividing a space horizontally (often called sharding). Each partition is a separate data store in this strategy, although they all share the same schema. A shard is a partition that holds a specific subset of data, for as all orders for a certain set of clients.

**Vertical Partitioning** is a term that refers to the division of space vertically. Each partition in this strategy stores a subset of the fields for objects in the data store. The fields are separated into groups based on how they are used. For example, frequently accessible fields could be grouped together in one vertical division while less frequently visited fields are grouped together in another.

**Functional partitioning** Data is aggregated in this technique based on how each bounded context in the system uses it. An e-commerce system, for example, might keep invoice data in one partition and product inventory data in the other.

**Directory Based Partitioning** Creating a lookup service that knows your current partitioning scheme and abstracts it away from the DB access code is a loosely connected solution to work around concerns highlighted in the above schemes. To determine the location of a specific data entity, we query the directory server that stores the mapping between each tuple key and its DB server. Because of this loose coupling, we can conduct operations such as adding servers to the DB pool or modifying our partitioning scheme without affecting the application.

## Partitioning Criteria

**a. Key or Hash-based partitioning:**
We use a hash function on some key features of the item we're storing to generate a partition number.

**b. List partitioning:**
Each partition has a set of values allocated to it, so if we want to add a new record, we'll look to see which partition holds our key and save it there. For example, we could specify that all users from Iceland, Norway, Sweden, Finland, or Denmark should be stored in a Nordic division.

**c. Round-robin partitioning:**
This is a straightforward approach for ensuring data distribution consistency. The I tuple is assigned to partition with 'n' partitions (i mod n).

**d. Composite partitioning:**
We combine any of the aforementioned partitioning schemes to create a new scheme in this scheme. For instance, apply a list partitioning strategy first, then a hash-based partitioning method. Consistent hashing is a hybrid of hash and list partitioning, in which the hash shrinks the key space to a size that can be listed.
