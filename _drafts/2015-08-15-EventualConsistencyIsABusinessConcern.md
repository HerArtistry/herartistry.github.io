---
layout: post
title: Eventual Consistency is a Business Concern
comments: true
tags: [SOA, Eventual Consistency]
---
Eventual consistency is an important concept in distributed systems as it allows you to scale considerably and leverage the underlying infrastructure to do the heavy lifting. It prevents hogging all the resources until a write is complete and provides leeway to recover from such resources going down or being unavailable. 

Eventual consistency, however, affects the end users directly as it drastically alters the way the product is used and can sometimes be unintuitive. Therefore, it is a business decision and not something that the developers alone can decide and implement. Retrofitting eventual consistency onto a system is etremely diffiult, not from  tehnical prespetive but from  business one. Take for example an online booking system for sporting events, in a strongly consistent system, the customers is presented with a picture of the venue and the seats available and is allowed to highligh which seats they want and book them. In an evenually consistent system, the customers are asked how many tickets they want, in what category and they are given a confirmation that there request was successful and that they must await consiration of their booking by, let's say, e-mail. This represents a significant difference in terms of customers' experience, as in the first instance, customers have a confirmed booking; while in the second one they don't. Now imagine having t change your application from the first one to the second, is this something you can do without business approval? Even in case of greenfield projects, the business have to know what they are agreeing to. Would they be happy with th second approach? can you make that call on their behalf? 

Eventual consistency is not something you can retrofit onto an application as it affects the user directly and drastically alters the way your system behaves. 

It is not a decision to be taken lightly and cannot be taken by technical people alone. The business has to understand the impact this has on the product.

Sometimes, even an understanding is not enough as a lot of people don't fully grasp the impact until they see it and try it out. Hence, start with eventual consistency and iron out all the misconecptions early when it is easier to change as it is not trivial and gets harder the more you invest in it and the later you have to change it.

This can have an adverse effect on scalability (no eventual consistency) if not considered from the get go.
