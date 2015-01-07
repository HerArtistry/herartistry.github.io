---
layout: post
title: Scalability and SOA - Part 2: Event-Driven SOA
---

In the previous post in this series [add link], I spoke briefly about scaling up vs out and why we chose scaling out. In this post, I will delve deeper into services that aid scaling out strategies.

#### Services in Service Oriented Architecture ####
If you are working with SOAs, then I highly recommend attending Udi Dahan's Advanced Distributed Systems Design course, or alternatively buying the recordings. 

So, what is a service?  as Udi states it: "a service is the technical authority for a business capability". What does this mean? It means a service is responsible for everything related to a specific business capability. It means UI, presentation logic, business logic and data storage for that business capability is contained within the boundaries of the service and the actual data does not leak out of the service boundaries. Why? There are numerous advantages for doing this:

- When data leaks out of a service, logic almost always follows. This would lead to code duplication in terms of both models/entities and business logic. Additionally, inter-service dependencies creates chatty interfaces between services.

- The service has no dependency on other services when fulfilling a business transaction. Hence, there are no calls over the network, no hindrance due to the [fallacies of distributed systems](http://en.wikipedia.org/wiki/Fallacies_of_distributed_computing). The entire transaction is carried out in-process.

- As every service has it's own data that it does not share with any other service, only the service responsible for the data can alter the data. This ensures that the data can only ever be changed according to the business rules defined by this service (encapsulation).

- Since every service contains all the data it needs to service requests, services are not coupled to each other. This is crucial when building a scalable system.


Adopting this approach means the overall system is divided into vertical slices, each representing a service. So how do services communicate?

####Commands vs Events####
Events represent things that have already happened and, therefore, cannot be undone (e.g. UserCreated). Commands are instructions to carry out an action and can be rejected (e.g. CreateUser). Command messages create coupling between services as the command instigator service knows which service can fulfil the command and what that service should do in processing a command. As bill Poole states [here](http://bill-poole.blogspot.co.uk/2008/04/avoid-command-messages.html):

>The fact that Service A is instructing Service B what to do means that Service A determines what shall be done, whereas Service B decides how it shall be done. This is a subtle but important form of coupling.

>Because Service B is performing an operation for Service A, as Service A evolves we may find that Service B is no longer able to meet the needs of Service A. Now we must update Service B due to a change in the needs of Service A. This is the essence of coupling.

>The root of the problem here is that Service B's behaviour is being governed by Service A. Service A is making an assumption regarding Service B's behaviour, introducing a dependency.

A system based on command messages as the form of communication will quickly become overly complex with numerous services governing each other and dictating what other services can do. As Bill explains it:

>Consider the case where Service C now must perform some action in the context of Service A's activities. We now need to update Service A to send a command message to Service C as well. Imagine how the complexity here can grow when we have a large number of services!

This is exactly why Event messages are preferred as they eliminate this coupling. Using the above example, if Service A's activity requires service B. Then service A fires an event that service B is subscribed to. Service B then carries out its part using information stored within it when it handles that event. If another service is now needed, it simply subscribes to the event and executes its part when the event is fired.

####Event-Driven Architecture and SOA####
As stated on this MSDN article, "Event-driven architecture is an architectural style that builds on the fundamental aspects of event notifications to facilitate immediate information dissemination and reactive business process execution." Basically, services in SOA generally communicate through events. A service fires an event describing a meaningful state change, and other interested service subscribe to such events and carry out any necessary actions required when such an event is fired. This is a very powerful architecture that allows significant scalability and decouples the services in the system as the service that fires the event does not know, and does not care, which other services are consuming this event and why.

**What to include in an event?** Events should only have IDs as data should not leak outside the service boundaries for the reasons specified at the beginning of this article. If more information is needed, say for auditing, then a local auditing handler should be created and [polymorphic message dispatch](http://www.udidahan.com/2011/01/13/polymorphism-and-messaging/) should be used.

If events only include IDs and event handlers should only use local data, then **how is the relevant data disseminated?** This is done through UI Composition and I will be covering that in the next part of this series.
