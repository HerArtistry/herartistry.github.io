---
layout: post
title: Scalability
---

In this post, I will go through the challenges experienced developing a highly scalable and highly available SaaS product. This was especially challenging because the .Net world seems to lag behind, once again, in terms of information regarding architectures supporting high scalability as well as frameworks/libraries.

## Scaling up vs scaling out ##
The first question you might want to ask yourself when striving for scalability is whether you want to sale out or up? When scaling up (vertical scaling), you simply throw in more hardware. While scaling out (horizontal scaling), is where you add more servers. 

**Scaling up** is generally easier to implement as you have one application to develop, test, host and deploy. It also means less licensing cost. However, the hardware costs are very high, a single point of failure (although you can have two instances of the server load-balance, the pricey hardware this approach enforces could weigh heavily on the project's budget).

**Scaling out** is the way to go if scalability, availability and high fault-tolerance is the goal. Alas, it does have its disadvantages. More licensing fees (albeit, more and more products now have elastic licenses that help reduce this cost), relatively harder implementations and a more complicated, and carefully thought-out, architecture. Nevertheless, the advantages outweigh the disadvantages if you are after a highly scalable/available/fault-tolerant system. It is much cheaper than scaling up and hugely fault-tolerant. Additionally, if done with the right architecture
