---
layout: post
title: Embracing DTC - 6 months later
comments: true
tags: [SOA, Service Oriented Architecture, Scalability, Transactions, DTC, MSDTC]
---

How DTC locks you down in terms of technology you can use. And how this affected us when we tried to move our system to PaaS instead of IaaS.
How I would do it now: Idempotent messages and de-duplication.

I was involved in developing/architecting a SOA system on top of NServiceBus and MSMQ and we were faced with the decision of either going down the DTC route or implementing idempotency/de-duplication. After considering the two alternatives - and attending a presentation by Udi Dahan - we decided to choose DTC. This is due to: 
- Time constraints
- Simplicity in terms of development
- Lack of performance benchmarks

What people fail to tell you though is that DTC locks you down in terms of technology. Not all products support DTC and some of the ones that do have issues/bugs in their implementation. Hence, your architecture decision double up as a technology contraint, which in some cases aren't a problem. But why would you unnecessar restrict yourself? Your architecture needs to be agile enough to allow to deal with unforseen changes. We experience this problem when we were developing a SaaS system and decided to use Azure IaaS to deploy our system. This decision was solely made due to our decision to rely on DTC.
