---
layout: post
title: Scalability - Part 1
---

In this series of posts, I will go through the challenges experienced developing a highly scalable and highly available SaaS product. This was especially challenging because the .Net world seems to lag behind, once again, in terms of information regarding architectures supporting high scalability as well as frameworks/libraries.

## Scaling up vs scaling out ##
The first question you might want to ask yourself when striving for scalability is whether you want to scale out or up? When scaling up (vertical scaling), you simply throw in more hardware. While scaling out (horizontal scaling), is where you add more servers. 

**Scaling up** is generally easier to implement as you have one application to develop, test, host and deploy. I say generally because I have seen, and unfortunately inherited, systems that are just a bunch of web services (JBOWS) hosted in one server. In my opinion, if you have no intention of scaling out the system, don't abstract parts of the system behind a web service. If on the other hand, you do want to scale out, then JBOWS is not the answer (more on that in a future post). 

Scaling up also means less licensing cost. However, the hardware costs are very high and there is a single point of failure (although you can have two instances of the server load-balance but the pricey hardware this approach enforces could weigh heavily on the project's budget).

**Scaling out** is the way to go if scalability, availability and high fault-tolerance is the goal. Alas, it does have its disadvantages. More licensing fees (albeit, more and more products now have elastic licenses that help reduce this cost), relatively harder implementations and a more complicated, and carefully thought-out, architecture. Nevertheless, it is much cheaper than scaling up and hugely fault-tolerant. Additionally, if done with the right architecture, then the advantages far outweigh the disadvantages - especially if you are after a highly scalable/available/fault-tolerant system.

In order to maximize the benefits and scale out effectively, the system needs to be divided into self-contained, low coupled services. This way, you can replace, scale (run multiple instances of the same service) and upgrade services individually without affecting the overall system.

In the next post, I will delve deeper into services and SOA.
 