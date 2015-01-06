---
layout: post
title: Scalability - Part 2: Event-Driven SOA
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

##Part 3: UI Composition (should be separate post)#

###UI Composition###
If you have not heard of the term before, then I recommend reading [this](http://www.udidahan.com/2012/06/23/ui-composition-techniques-for-correct-service-boundaries/) article by Udi Dahan.

Every service would provide partial views to present the data it has. When a particular screen contains data from different services, then these partial views are mashed-up together to create one "Composite UI" view.

In order to work effectively with event-driven SOA, the business processes are re-modelled as an asynchronous series of events. For example, a user registration process may consist of creating users, creating their log-in credentials, e-mailing the users a link to re-set their password. Traditionally, this may be presented as a wizard with a linear synchronous process where each step needs to be completed in order to move to the next step. In Event Driven SOA, the users' details and credentials are  asynchronously saved at the same time and they share the user ID. Once the user is created by the users service, a UserCreated event containing only the user ID  is fired. The credentials service, which is subscribed to this event, then uses the user ID contained in the event to retrieve the user's credentials, it then creates a reset password token (just a Guid ID) and composes the credentials' specific password reset e-mail. It then sends a command to the e-mail service to send the e-mail. The reason we can fire a UserCreated event with just an ID, is that all the user's relevant credentials information is already saved in the credentials service through the **Composite UI**.

> **Note:** You need to be judicious in your use of commands as they create coupling. More on this in a future post.

Composite UIs lead to loosely coupled services as even services that seem to collaborate in order to create one screen actually do not know about each other. Take the above users registration for example, the user service provides input fields to capture and store the users' details, while the credentials service provides similar inputs for the users' credentials.  Each view knows how to persist itself to its associated service; well, to be more precise, the view delegates this to a client-side controller/service (Note: all the client-side code lives within the service boundaries). In other words, when the users saves the form, each view sends a request to its service, which, in turn, saves the data to its data store. When the users click save, a client side event is fired and each partial view then persists itself.

**Client-side generated IDs:** In order to associate relevant data saved to disparate services, the ID is generated client side when you get to the composite view and then used in all services. E.g. when the end user navigated to the "Create User" screen, a new user ID is generated and used by both the users and credentials services. This way, when the user service is done persisting the user details, it fires an event containing nothing but the user ID. The credentials service can use this ID to pull up the credential information associated with that user.

> **Note:** If the user created event is fired before the credentials service has stored the user's credentials, then you just need to retry again as it will eventually be created. This is how an eventual consistent system works and you need to cater for this. NServicebus has an exponential back-off retry feature that comes in handy in such situations.

>**Note:** You should not enforce referential integrity across service boundaries because it re-introduces coupling.

