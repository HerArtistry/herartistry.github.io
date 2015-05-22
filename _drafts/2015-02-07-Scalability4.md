---
layout: post
title: Scalability and SOA - Part 4 (Anatomy of a Service)
comments: true
tags: [SOA, Service Oriented Architecture, Scalability]
---

we have so far covered what should a service contain and how it communicates with other services but we have not discussed how the inside of a service looks like

Before digging deeper it is probably worth discussing how our system looks like as well as the internals of the services.

All our services are vertical slices of the system and communicate through events (low behavioural coupling) delivered by NServiceBus and MSMQ. MSMQ guarantees that messages will be delivered even when the subscriber (a service in our case) is not available (low temporal coupling). 

The UI is within the boundaries of the service and, hence, is allowed to have request/response interaction with the back-end. These contain both command and query requests. Queries are processed synchronously, while commands are validated synchronously but processed asynchronously. In order to get this to work, we
 
- invested in ensuring valid commands are always successful, and
- embraced eventual consistency.

Invalid commands return a message to the front-end immediately and it is up to the end-user to correct/fix the command. Valid ones are passed over to NServiceBus and a success message is returned to the user.

In the real world, system data changes constantly; hence, edge cases where a command may have been successful when it was validated but would violate business rules when processed sometimes appear. Some might have defined compensation actions in the business, while others might not. Take, for example, checking that username is unique when creating the user. There is a very slight chance that an identical username is created in the time between validating the CreateUser command and processing it. In order not to cause disruptions to the user, a unique human readable username placeholder is created, thus, allowing the user to use the system freely, but an e-mail is sent out to the user to correct it. This change only affects the service responsible for the username data as only IDs are allowed outside the service boundaries; limiting the ripple affect of this change. 

>Revisit: Therefore, the messages displayed to the user should reflect that. For example, "Request was received. Please await e-mail confirmation".

NServiceBus and RavenDB were used, along with MSMQ, to ensure robustness as they all supports distributed transactions (DTC). DTCs have garnered a bad name due to their performance implications but do not get put off by that as most systems do not require the level of performance that necessitates avoiding DTC. If you are in doubt, benchmark it. Write performance tests according to your agreed performance requirements and start with the simplest thing first: turn DTC on. Do not prematurely try to optimize for better performance when you don't need to. If you start hitting the thresholds you have established, then start addressing them. Later in this article, I will discuss alternative to DTC. In our case, our requirements were strict on data loss, and, thus, DTCs and products that support it was more important for us. 

What happens when there is no DTC? Imagine a situation where a message is processed and the database has been updated and other events fired but an acknowledgement was not sent back to the queue because the machine crashed. When the machine is up and running, the queue will try to send the message again, which will consequently be reprocessed; leaving the system in an undesired state. This, depending on the business, can have major repercussions.

>Revisit: Transactions should not span service boundaries nor should they span autonomous components boundaries within a service. This is fundamental for scalability. If you are in a situation were you need a transaction that involve more than one service, then your service boundaries are not correct and they need to be revised. Changing service boundaries is tricky when the system is in production but fairly cheap before that, hence, you need to spend appropriate time discovering your services and setting their boundaries and factoring that into your sprints, if doing agile. There is a misconception in agile that you should not have big upfront design but that does not mean no upfront design at all.

One alternative to DTC is to ensure messages are idempotent and prepare corrective/compensation actions for any side effects. Some messages are naturally idempotent, say MakeCustomerGoldMember,as it does not matter how many times you processes this message, the outcome is always the same. However, we are not looking at the big picture here and in real life it is rarely this simple. There is probably a complex business process that this command kick starts, for example, it could lead to a CustomerMadeGoldMember event to fire and, as we stated earlier, any other service can subscribe to this event and perform actions. Therefore, this model adds considerable complexity in terms of implementation, design, testing, business process re-modelling (for corrective/compensation actions) and maintainability.


There are other alternatives to DTC, however, it is not something that we have looked at as we have already made the trade-off of choosing simplicity and robustness over performance and throughput. For further information about not using DTC, Jimmy Bogard has an excellent article about [ditching two phased commits](http://lostechies.com/jimmybogard/2013/05/09/ditching-two-phased-commits/) where he also links to [this](http://www.enterpriseintegrationpatterns.com/docs/IEEE_Software_Design_2PC.pdf) paper by Gregor Hohpe.


