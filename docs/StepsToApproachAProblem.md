# Steps To Approach A Problem

It is suggested that you solve the problem by following the procedures below.

## Feature Expectations (First 2 Minutes): 
As previously said, there is no such thing as a bad design. There are only excellent and terrible designs, and the same solution may be good for one use case but awful for another. As a result, it is critical to have a thorough understanding of the question's requirements.

## Approximations ( 2-5 mins )
The next step is usually to calculate the system's scale. The purpose of this step is to figure out how much sharding is needed (if any) and to narrow down the system's design goals.
For example, if all of the system's data fits on a single machine, we might not need to worry about sharding of the other issues that come with a distributed system architecture.
Alternatively, if the most frequently used data fits on a single machine, caching might be done there.

## Design Objectives ( 1 mins )
Determine what the system's most critical objectives are. It's possible that some systems are latency systems, in which case a solution that ignores this could result in poor design.

## The design's skeleton ( 4 - 5 mins )
There isn't enough time to go over each component in depth in 30-40 minutes. As a result, a solid method is to talk with the interviewer at a general level and then dive into the components that the interviewer has asked about.

## Extensive research ( 20-30 mins )
Using low level and high level design, go over the problem in detail.

The more detailed question needs to think and asked can be seen in next session

**In the next session, we'll look at the more detailed questions that need to be considered and asked.**