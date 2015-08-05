---
layout: post
title: Eventual Consistency is a Business Concern
comments: true
tags: [SOA, Eventual Consistency]
---
Eventual consistency is not something you can retrofit onto an application as it affects the user directly and drastically alters the way your system behaves. 

It is not a decision to be taken lightly and cannot be taken by technical people alone. The business has to understand the impact this has on the product.

Sometimes, even an understanding is not enough as a lot of people don't fully grasp the impact until they see it and try it out. Hence, start with eventual consistency and iron out all the misconecptions early when it is easier to change as it is not trivial and gets harder the more you invest in it and the later you have to change it.

This can have an adverse effect on scalability (no eventual consistency) if not considered from the get go.
