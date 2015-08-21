---
layout: post
title: Eventual Consistency is a Business Concern
comments: true
tags: [SOA, Eventual Consistency]
---
Eventual consistency is an important concept in distributed systems as it allows you to scale considerably and improves your systems' availability and robustness. There are numerous blog posts discussing this in details so I'm not going to do that here. What I want to emphasise in this post is that eventual consistency is a business concern. 

Eventual consistency affects the end users directly as it drastically alters the way the product is used and can sometimes diminish the user experience. Therefore, it is a business decision and not something that the developers alone can determine and implement. Retrofitting eventual consistency onto a system is extremely difficult, not just from a technical prespective but from a business one too. Take for example an online booking system for sporting events, in a strongly consistent system, the customers are presented with some sort of a graphical representation of the venue and the seats available and allowed to select which seats they want and book them. In an evenually consistent system, the customers are asked how many tickets they want, in what category and they are given a confirmation that there request has been received and that they must await confirmation of their booking by, let's say, e-mail. This represents a significant difference in terms of customers' experience, as in the first instance, customers have a confirmed booking; while in the second one they don't. Now imagine having to change your application from the first one to the second, is this something you can do without business approval? Even in greenfield projects, the business has to know what they are agreeing to. Would they be happy with the second approach? can you make that call on their behalf? 

This is not a decision to be taken lightly and cannot be determined by technical people alone. The business has to understand the impact this has on the product. In my experience, business people don't fully grasp the effect this has until they see it and try it out. Hence, start with eventual consistency and iron out all the misconecptions early when it is easier to alter as it is not trivial and it only gets harder the more you invest in it and the later you have to change it.

There are ways to decrease the effect eventual consistency has on the customers in terms of both technical solutions and business process ones but this is a topic for another post. 
