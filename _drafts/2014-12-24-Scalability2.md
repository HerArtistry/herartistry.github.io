---
layout: post
title: Scalability - Part 2
---

In the previous post in this series [add link], I spoke briefly about scaling up vs out and why we chose scaling out. In this post, I will delve deeper into services that aid scaling out strategies.

### Services in Service Oriented Architecture ###
If you are working with SOAs, then I highly recommend attending Udi Dahan's Advanced Distributed Systems Design course, or alternatively buying the recordings. 

So, what is a service?  as Udi states it: "a service is the technical authority for a business capability". What does this mean? It means a service is responsible for everything related to a specific business capability. It means UI, presentation logic, business logic and data storage for that business capability is contained within the boundaries of the service and the actual data does not leak out of the service boundaries. Why? There are numerous advantages for doing this:

- When data leaks out of a service, logic almost always follows. This would lead to code duplication in terms of both models/entities and business logic. Additionally, inter-service dependencies creates chatty interfaces between services.

- The service has no dependency on other services when fulfilling a business transaction. Hence, there are no calls over the network, no hindrance due to the [fallacies of distributed systems](http://en.wikipedia.org/wiki/Fallacies_of_distributed_computing). The entire transaction is carried out in-process.

- As every service has it's own data that it does not share with any other service, only the service responsible for the data can alter the data. This ensures that the data can only ever be changed according to the business rules defined by this service (encapsulation).

- Since every service contains all the data it needs to service requests, services are not coupled to each other. This is crucial when building a scalable system.


Adopting this approach means the overall system is divided into vertical slices, each representing a service. So how do services communicate?

###Event-Driven Architecture and SOA###
As stated on this MSDN article, "Event-driven architecture is an architectural style that builds on the fundamental aspects of event notifications to facilitate immediate information dissemination and reactive business process execution." Basically, services in SOA generally communicate through events. A service fires an event describing a meaningful state change, and other interested service subscribe to such events and carry out any necessary actions required when such an event is fired. This is a very powerful architecture that allows significant scalability and decouples the services in the system as the service that fires the event does not know, and does not care, which other services are consuming this event and why.