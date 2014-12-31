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

**What to include in an event?** There are two kind of events: internal and public. Internal events are ones that are only local to the service firing them. Public events are those that are broadcast to all services. As stated above, services should not share data, therefore, public events should have nothing but IDs. Internal events, on the other hand, can have data and can derive from the public events to make use of [polymorphic message dispatch](http://www.udidahan.com/2011/01/13/polymorphism-and-messaging/) - provided your service bus supports it.

###UI Composition###
**So.... how is the relevant data disseminated?** through UI Composition. If you have not heard of the term before, then I recommend reading [this](http://www.udidahan.com/2012/06/23/ui-composition-techniques-for-correct-service-boundaries/) article by Udi Dahan.

**TL;DR:** UI Composition is used when:

**displaying the data:** Every service would provide partial views to present the data it has. When a particular screen contains data from different services, then these partial views are mashed-up together to create one "Composite UI" view.

**capturing the data:** Similar to when displaying the data. However, the partial views allow the user to enter or edit data. Every service would provide forms to capture the data that is relevant to it. For example, when creating a user, we capture users' details as well as their credentials. The user service provides input fields to capture and store the users' details, while the credentials service provides similar inputs for the users' credentials.  Each view knows how to persist itself to its associated service; well, to be more precise, the view delegates this to a client-side controller/service (Note: all the client-side code lives within the service boundaries). In other words, when the users saves the form, each view sends a request to its service, which, in turn, saves the data to its data store. 

In order to associate relevant data saved to disparate services, the ID is generated when you get to the composite view and then used in all services. E.g. when the a new user is created, a user ID is generated and used by both the users and credentials services. This way, when the user service is done persisting the user details, it fires an event containing nothing but the user ID. The credentials service can use this ID to pull up the credential information associated with that user.

Note: If the user created event is fired before the credentials service has stored the user's credentials, then you just need to retry again as it will eventually be created. This is how an eventual consistent system works and you need to cater for this. NServicebus has an exponential back-off retry feature that come in handy in such situations.

Note: You should not enforce referential integrity across service boundaries.

