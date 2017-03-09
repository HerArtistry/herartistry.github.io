---
layout: post
title: Scalability and SOA - Part 2 (Event-Driven SOA)
comments: true
tags: [SOA, Service Oriented Architecture, Scalability]
---

In the [previous post](http://www.ashrafmageed.com/Scalability/) in this series, I wrote about scaling up vs out, why we chose scaling out and why layered service models are not the answer. In this post, I will delve deeper into vertical services and how they help with scaling out strategies.


#### Services in Service Oriented Architectures ####

If you are working with SOAs, then I highly recommend attending Udi Dahan's Advanced Distributed Systems Design course, or alternatively buying the recordings. 

So, what is a service?  as Udi states it: "a service is the technical authority for a specific business capability". What does this mean? It means the UI, presentation logic, business logic and data storage for that business capability is contained within the boundaries of the service and the actual data does not leak out of the service boundaries. Why? There are numerous advantages for doing this:

- Defining service boundaries according to business capabilities results in stable boundaries as these are unlikely to change. This reduces the risk and overhead of needing to migrate data/models/events from one service to another.

- When data leaks out of a service, business logic almost always follows. This would lead to code duplication in terms of both models/entities and business logic resulting in low cohesion and poor encapsulation. Additionally, inter-service dependencies creates chatty interfaces between services.

- The service has no dependency on other services when fulfilling a business transaction. Hence, there are no calls over the network, no hindrance due to the [fallacies of distributed systems](http://en.wikipedia.org/wiki/Fallacies_of_distributed_computing). The entire transaction is carried out in-process.

- As every service has its own data that it does not share with any other service, only the service responsible for the data can alter the data. This ensures that the data can only ever be changed according to the business rules defined by this service (encapsulation).

- Since every service contains all the data it needs to service requests, services are not coupled to each other. This is crucial when building a scalable system.


Adopting this approach means the overall system is divided into vertical slices, each representing a service. Defining the service boundaries, in our experience, is tricky and daunting especially at the start of the project when you do not know enough about the system and its requirements. Udi Dahan has a nice article about discovering services: [7 simple question about Service Selection](http://www.udidahan.com/2008/05/16/7-simple-questions-for-service-selection/) that we followed - I will go into details in a future post.

**So how do services communicate?** Through messages. There are three types of messages, namely: event messages, command messages and document messages. 

**Document messages** simply transfer data between services but do not dictate what the receiver should do with the data. They usually represent business documents, say an IncidentReport or a PurchaseOrder, and contain all the information pertaining to that document from all the relevant services, each service will act on and/or fill in the parts it knows about. As stated above, data should not leak out of the service boundaries; this immediately rules out document messages for communication between services. Moreover, there are numerous advantages in having thin messages that I will cover in a future post.

#### Command Messages vs Event Messages ####
Events represent things that have already happened and, therefore, cannot be undone (e.g. UserCreated) - well, can be undone through a corrective action/command, e.g. InactivateUser. Commands are instructions to carry out an action and can be rejected (e.g. CreateUser). Command messages create coupling between services as the command instigator service knows which service can fulfil the command and what that service should do in processing a command. As bill Poole states [here](http://bill-poole.blogspot.co.uk/2008/04/avoid-command-messages.html):

>The fact that Service A is instructing Service B what to do means that Service A determines what shall be done, whereas Service B decides how it shall be done. This is a subtle but important form of coupling.

>Because Service B is performing an operation for Service A, as Service A evolves we may find that Service B is no longer able to meet the needs of Service A. Now we must update Service B due to a change in the needs of Service A. This is the essence of coupling.

>The root of the problem here is that Service B's behaviour is being governed by Service A. Service A is making an assumption regarding Service B's behaviour, introducing a dependency.

A system based on command messages as the form of communication will quickly become overly complex with numerous services governing each other and dictating what other services can do. As Bill explains it:

>Consider the case where Service C now must perform some action in the context of Service A's activities. We now need to update Service A to send a command message to Service C as well. Imagine how the complexity here can grow when we have a large number of services!

This is exactly why Event messages are preferred as they eliminate this coupling. Using the above example, if Services A and B need to collaborate in a business process, then service A fires an event that service B is subscribed to. Service B then carries out its part using information stored within it when it handles that event. There are no assumptions being made and no governance is introduced, in fact, service A is not even aware of service B's existence. If another service is now needed, it simply subscribes to the event and executes its part when the event is fired.

#### Event-Driven Architecture and SOA ####
As stated on this MSDN article, "Event-driven architecture is an architectural style that builds on the fundamental aspects of event notifications to facilitate immediate information dissemination and reactive business process execution." Event-Driven architecture and SOA are two different and independent architectural styles. You can do any of the two without the other but combining them yields great benefits. Basically, services in Event-Driven SOA generally communicate through events. A service fires an event describing a meaningful state change, and other interested service subscribe to such events and carry out any necessary actions required when such an event is fired. This is a very powerful architecture that allows significant scalability and decouples the services in the system as the service that fires the event does not know, and does not care, which other services are consuming this event and why. This result in low temporal and behavioural coupling.

**Temporal coupling** is where dependant services have to be available when a message is sent to them; while **Behavioural Coupling** is when services determine what other services should do and how. Ian Robinson wrote a wonderful article about [temporal and behavioural coupling](http://iansrobinson.com/2009/04/27/temporal-and-behavioural-coupling/) that I strongly recommend reading. He depicts temporal coupling and behavioural coupling on a matrix with event-based systems occupying the quadrant with low coupling on both and command-based systems on the low temporal coupling but high behavioural coupling.

So far we have established that services should communicate through events and data should not leak outside the services boundaries; so what should an event contain and how is data disseminated? This will be the topic of the next post in this series.
