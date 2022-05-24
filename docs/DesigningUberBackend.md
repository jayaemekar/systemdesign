# Designing Uber backend

## Problem Statement
Let's design a ride-sharing service like Uber, which connects passengers who need a ride with drivers who have a car.

- Similar Services: Lyft, Didi, Via, Sidecar, etc.
- Difficulty level: Hard
- Prerequisite: Designing Yelp

### What is Uber?
Customers may book drivers for taxi rides using Uber. Uber drivers drive consumers around in their own automobiles. The Uber app allows consumers and drivers to communicate with one another via their smartphones.

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

    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#requirements-and-goals-of-the-system" target="_blank">1. Consider functional and non-functional requirements. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#capacity-estimation-and-constraints" target="_blank">2. Estimation of capacity and constraints, such as traffic, bandwidth, and storage. </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#basic-system-design-and-algorithm" target="_blank">3. Consider Basic System Design and Algorithm </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#ranking" target="_blank">4. How do you create a ranking system? </a>
    <br><br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#fault-tolerance-and-replication" target="_blank">5. What about data replication and partitioning?</a>
    <br>
    <br>
    <a href="https://jayaemekar.github.io/systemdesign/DesigningUberBackend/#advanced-issues" target="_blank">6.  Consider Advanced Issues</a>
    <br>
<br><br>
</body>
</html>


## <h1>Solution<h1>

### Requirements and Goals of the System
Let’s start with building a simpler version of Uber.

In our system, there are two sorts of users: 1) Motorists 2) Clientele.

1. Drivers must inform the service of their current position and availability to pick up customers on a frequent basis.
2. Passengers can see all available drivers in their immediate vicinity.
3. Customers can request a ride, and neighboring drivers will be notified.
4. Once a consumer accepts a ride, the driver and customer can see each other's current location until the trip is completed.
5. When the driver arrives at his destination, he marks the ride as completed and becomes ready for the next ride.

### Capacity Estimation and Constraints
1. Assume we have 300 million consumers and one million drivers, with one million daily active customers and 500 thousand daily active drivers.
2. Assume 1 million daily rides.
3. Assume that every three seconds, all active drivers broadcast their present location.
The system should be able to contact drivers in real time if a customer requests a ride.

### Basic System Design and Algorithm
We'll alter the solution outlined in Designing Yelp to make it work for the "Uber" use cases mentioned above. The most significant difference we have is that our QuadTree was not designed with frequent updates in mind. So, with our Dynamic Grid solution, we have two issues:

1. We need to change our data structures to reflect the fact that all active drivers are reporting their whereabouts every three seconds. It will take a lot of time and resources to update the QuadTree for every change in the driver's location. We must locate the correct grid based on the driver's previous location to update a driver to its new location. We must delete the driver from the current grid and move/reinsert the user to the right grid if the new position does not correspond to the current grid. If the new grid exceeds the maximum number of drivers after this relocation, we must repartition it.

2. We need a fast way to communicate the current location of all surrounding drivers to every active customer in the vicinity. Also, when a ride is in progress, our system must inform both the driver and the passenger of the car's present location.
Although our QuadTree assists us in swiftly locating nearby drivers, a speedy update in the tree is not guaranteed.

**Do we need to modify our QuadTree every time a driver reports their location?** 

- QuadTree will have some old data with each driver update and may not accurately reflect the current position of drivers. If you recall, the QuadTree was designed to quickly locate nearby drivers (or locations). 
- There will be a lot more updates to our tree than asking for nearby drivers because all current drivers submit their location every three seconds. 
- So, what if we save all drivers' most recent positions in a hash table and update our QuadTree less frequently? Assume that the current location of a driver will be reflected in the QuadTree within 15 seconds. 
- Meanwhile, we'll keep a hash table to keep track of the present location drivers have reported; This will be known as DriverLocationHT.

How much memory/RAM does DriverLocationHT require? In the hash table, we must store DriveID, as well as their current and previous locations. To store one record, we'll need a total of 35 bytes:

1. DriverID (3 bytes - 1 million drivers)
2. Old latitude (8 bytes)
3. Old longitude (8 bytes)
4. New latitude (8 bytes)
5. New longitude (8 bytes) Total = 35 bytes

If we have 1 million total drivers, we need the following memory (ignoring hash table overhead):

            1 million * 35 bytes => 35 MB
**How much bandwidth will our service consume to receive location updates from all drivers?** 

If we get It will be (3+16 => 19 bytes) for DriverID and their location. We will receive 9.5MB per three seconds if we receive this information every three seconds from 500K daily active drivers.

**Do we need to distribute DriverLocationHT onto multiple servers?** 

Although our memory and bandwidth requirements don't necessitate it because all of this data can be stored on a single server, we should divide DriverLocationHT among numerous servers for scalability, performance, and fault tolerance. To make the distribution entirely random, we can distribute depending on the DriverID. The machines that hold DriverLocationHT are referred to as the Driver Location server. Each of these servers will do two things in addition to storing the driver's location:

1. As soon as the server receives an update on a driver's position, it will broadcast it to all clients who are interested.
2. The server must notify the appropriate QuadTree server in order to update the driver's location. As previously stated, this can occur every 10 seconds.

**How can we efficiently broadcast the driver’s location to customers?** 

- We can utilize a Push Model, in which the server sends the positions to all users who are interested. We could create a dedicated Notification Service that broadcasts the current location of drivers to all clients that are interested. 
- We can create a publisher/subscriber model for our Notification service. When a customer launches the Uber app on their smartphone, the server searches for nearby drivers. 
- Before providing the list of drivers to the client, we shall subscribe the customer to all updates from those drivers on the server side. We can keep a list of clients (subscribers) who want to know where a driver is and, if there is an update in DriverLocationHT for that driver, we can broadcast the driver's current location to all subscribers. 
- Our system ensures that the customer is always aware of the driver's current location.

**How much memory will we need to store all these subscriptions?** 

As previously said, we expect 1 million daily active consumers and 500 thousand daily active drivers. Assume that five consumers subscribe to one driver on average. Let's pretend we keep all of this data in a hash table so we can update it quickly. To keep the subscriptions running, we need to keep track of the driver and customer IDs. We'll require 21MB of memory if we need 3 bytes for DriverID and 8 bytes for CustomerID.

                    (500K * 3) + (500K * 5 * 8 ) ~= 21 MB
How much bandwidth will we need to broadcast the driver’s location to customers? For every active driver, we have five subscribers, so the total subscribers we have:

                    5 * 500K => 2.5M
To all these customers we need to send DriverID (3 bytes) and their location (16 bytes) every second, so, we need the following bandwidth:

                    2.5M * 19 bytes => 47.5 MB/s
**How can we efficiently implement Notification service?** 

We can either use HTTP long polling or push notifications.

**How will the new publishers/drivers get added for a current customer?** 

- Customers will be subscribed to nearby drivers when they open the Uber app for the first time, as we proposed above; however, what will happen if a new driver enters the area the client is looking at? We need to keep track of the region the client is watching in order to dynamically add a new customer/driver subscription. 
- This will complicate our solution; how about clients pulling this information from the server instead of pushing it?

**How about if clients pull information about nearby drivers from the server?** 

- Clients can send their current position to the server, which will locate any nearby QuadTree drivers and return them to the client. 
- When the client receives this information, they can change their screen to reflect the drivers' current positions. To reduce the amount of round trips to the server, clients might query every five seconds. 
- In comparison to the push paradigm discussed above, this solution appears to be easier.

**Do we need to repartition a grid as soon as it reaches the maximum limit?** 

- We can have a buffer to allow each grid to grow slightly beyond the limit before partitioning it. 
- Let's say our grids can expand or contract by 10% more before we separate or merge them. 
- On high-traffic grids, this should reduce the demand for a grid split or merge.

<p align="center"> 
  <kbd>
  <a href="https://github.com/jayaemekar/systemdesign" target="_blank"><img src="../docs/images/uberbackend.JPG">
  </a>
  </kbd>
</p>

**How would “Request A Ride” use case work?**

1. The customer will make a ride request.
2. One of the Aggregator servers will accept the request and send it to the QuadTree servers, who will then return nearby drivers.
3. The Aggregator server gathers all of the results and organizes them according to their ratings.
4. The Aggregator server will send a notification to the top (say three) drivers at the same time, and the ride will be assigned to the driver who accepts the request first. A cancellation request will be sent to the other drivers. The Aggregator will request a ride from the next three drivers on the list if none of the three drivers answer.
5. The customer is notified when a driver accepts a request.

### Fault Tolerance and Replication
What happens if a Driver Location or Notification server goes down? We'd need duplicates of these servers so that if the primary goes down, the backup can take over. We can also store this data in some persistent storage, such as SSDs with quick IOs, so that we can recover the data from the persistent storage if both the primary and secondary servers fail.

### Ranking
**How about if we want to rank the search results not just by proximity but also by popularity or relevance?**

How can we find the best drivers within a certain radius?

Assume we maintain track of each driver's overall ratings in our database and QuadTree. In our system, an aggregated number can represent this popularity, such as how many stars a driver receives out of 10. We can ask each partition of the QuadTree to return the top 10 drivers with the highest rating while looking for the top 10 drivers within a defined radius. The aggregator server can then select the top ten drivers from all of the drivers returned by the various partitions.

### Advanced Issues
1. How will we deal with clients on sluggish or unstable networks?
2. What happens if a client is disconnected during a ride? In this case, how will we handle billing?
3. What if clients pull all of the data rather than servers always pushing it?