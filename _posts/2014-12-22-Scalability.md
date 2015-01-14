---
layout: post
title: Scalability and SOA - Part 1
---

In this series of posts, I will go through the challenges experienced developing a highly scalable and highly available SaaS product using a Service Oriented Architecture. 

The first question you might want to ask yourself when designing a scalable system is whether you want to scale out or up? When scaling up (vertical scaling), you simply throw in more hardware. While scaling out (horizontal scaling), is where you add more servers. 

**Scaling up** is generally easier to implement as you have one application to develop, test, host and deploy. I say generally because I have seen, and unfortunately inherited, systems that are just a bunch of web services (JBOWS) hosted in one server. In my opinion, if you have no intention of scaling out the system, don't abstract parts of the system behind a web service as, not only does this add no value, it drags you into the trap of the fallacies of distributed systems. If on the other hand, you do want to scale out, then JBOWS is not the answer (more on that in a future post). 

Scaling up also means less licensing cost. However, the hardware costs are very high and there is a single point of failure (although you can have multiple instances of the server load-balance but the pricey hardware this approach enforces could weigh heavily on the project's budget). Hence, scalability is somewhat limited. This does not mean this startegy cannot be employed as proven by 'Plenty of Fish'. You can read Jeff Atwood's views about it [here](http://blog.codinghorror.com/my-scaling-hero/).

**Scaling out** is the way to go if scalability, availability and high fault-tolerance is the goal. Alas, it does have its disadvantages. More licensing fees (albeit, more and more products now have elastic licenses that help reduce this cost), relatively harder implementations and a more complicated, and carefully thought-out, architecture. Nevertheless, it is cheaper than scaling up, especially with cloud solutions like Azure, and hugely fault-tolerant.

We have decided to go with a scale-out strategy as it gives us a lot more flexibility and it facilitates meeting our product's needs in terms of high availability (99.999% or five nines), scalability and resilience. 

###Architecting for a scaling-out strategy###

**Layered SOA is not the answer**

A layered service model is where the system is divided into horizontal slices. These are mainly three layers, namely: process, task and entity services. Bill Poole defines those as:

>The purpose of entity services is to provide an abstracted view of business document repositories. The actual structure of the underlying data and the platform holding the data are abstracted into the form of entity service contracts.

>Task services each represent a single atomic business action. Task services leverage entity services to manage state.

>Process services are then created in support of business processes. Each process service wires up any number of task services in support of a specific business process. Process services can also "invoke" other process services when there are sub-processes to be reused by those processes.

These service need to communicate with each other through synchronous RPC calls because non of the services can fulfil any non-trivial business processes on their own. [Synchronous request/reply is bad](http://bill-poole.blogspot.co.uk/2008/03/synchronous-requestreply-is-bad.html) and this layered service model consequently suffers in terms of performance, coupling, complexity to name but a few. This model introduces inter-dependencies between services that leads to transactions spanning multiple services (cross-service transactions) resulting in services not being as autonomous as they ought to be as they affect each others execution and the increase the chances of deadlocks spanning multiple services. Furthermore, layered service models have low cohesion and suffer from temporal coupling as all services need to be up and running in order for the system to perform properly. For a more detailed explanation, check out Bill Poole's [layered service models are bad](http://bill-poole.blogspot.co.uk/2008/05/layered-service-models-are-bad.html)

**Scaling out and vertical services**

In order to maximize the benefits and scale out effectively, the system needs to be divided into self-contained, low coupled, vertically sliced services. This way, you can replace, scale (run multiple instances of the same service) and upgrade services individually without affecting the overall system and its availability. Additionally, this also allows you to identify bottlenecks in your system at a more granular level and scale them separately; reducing the cost of scaling out. For example, instead of provisioning a new high-end server to address the whole business or data layers, you only need to spin up a regular server, or a low spec-ed one depending on requirements, that just targets the users service. 

In the next post, I will delve deeper into services in a service oriented architecture (SOA) and how to architecture SOA for scaling out.
 