---
layout: post
title: Refactoring Big Aggregates
comments: true
tags: [DDD, Event Sourcing, Aggregates]
---

Aggregates grow over time as you begin to gain more insight into your problem domain. Event sourcing somewhat limits this growth as you only keep the state you require to make decisions. But this could also increase the more logic you add into your aggregate. In this post, I will cover an option to refactor aggregates and reduce all that noise.

#### Separate State

State can be maintained separately, so you basically divide the aggregate into two classes, the aggregate and the aggregate's state.
