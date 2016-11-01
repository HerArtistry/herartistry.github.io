---
layout: post
title: DDD Aggregates: Focus on Business Logic
comments: true
tags: [DDD]
---
Primitive obsession in DDD aggregates usually results in the aggregate itself validating the values been passed in rather than focus on the business rules it needs to validate. This results in a mesh of concerns and dilutes the responsibilities of the aggregate. Even if these value are validated somewhere else what guarantees that these values have not been altered by the command handler, or through a bug in the code, before being passed into the domain. 
