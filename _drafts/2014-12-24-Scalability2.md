---
layout: post
title: Scalability - Part 2
---

In the previous post in this series [add link], I spoke briefly about scaling up vs out and why you should favour scaling out. In this post, I will delve deeper into services that aid scaling out strategies.

### Services in Service Oriented Architecture ###
If you are working with SOAs, then I highly recommend attending Udi Dahan's Advanced Distributed Systems Design course, or alternatively buy the recordings. 

So, what is a service?  as Udi states it: "a service is the technical authority for a business capability". What does this mean? It means a service is a vertical slice though the system and is responsible for everything related to a specific business capability. It means UI, presentation logic, business logic and data storage for that business capability is contained within the boundaries of the service. Why? There are numerous advantages for doing this:

**Cohesion:**
When data leaks out of a service, logic almost always follows. This would lead to code duplication in terms of both models/entities and business logic. Additionally, inter-service dependencies creates chatty interfaces between services.

**Performance**
The service has no dependency on other services when fulfilling a business transaction. Hence, there are no calls over the network, no hindrance due to the [fallacies of distributed systems](http://en.wikipedia.org/wiki/Fallacies_of_distributed_computing). The entire transaction is carried out in-process.

**Encapsulation:**
As every service has it's own data that it does not share with any other service, only the service responsible for the data can alter the data. This ensures that the data can only ever be changed according to the business rules defined by this service.

**Coupling:**
Since every service contains all the data it needs to service requests, services are not coupled to each other service in the system. This is crucial when building a scalable system.

