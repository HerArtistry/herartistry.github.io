---
layout: post
title: Embracing DTC - 6 months later
comments: true
tags: [SOA, Service Oriented Architecture, Scalability, Transactions, DTC, MSDTC]
---

How DTC locks you down in terms of technology you can use. And how this affected us when we tried to move our system to PaaS instead of IaaS.
How I would do it now: Idempotent messages and de-duplication.

I was involved in developing/architecting a SOA system on top of NServiceBus and MSMQ and we were faced with the decision of either going down the DTC route or implementing idempotency/de-duplication. After considering the two alternatives - and attending a presentation by Udi Dahan - we decided to choose DTC. This is due to: 
# Time constraints
# Simplicity in terms of development
# Lack of performance benchmarks
