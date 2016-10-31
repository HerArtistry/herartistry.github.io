---
layout: post
title: DTC-Enforced Constraints
comments: true
tags: [SOA, Service Oriented Architecture, Scalability, Transactions, DTC, MSDTC]
---

There are numerous blogs and talks out there talking about why you should or should not use distributed transactions coordinators (DTC) and they cover a myriad of topics, detailing the pros and cons. However, what most of them fail to mention - or not give enough air time to - is the fact that DTC locks you down in terms of the technology you can use. This is an issue that is constantly overlooked and can cause you a lot of issues down the line.

I was involved in developing/architecting a SOA system on top of NServiceBus and MSMQ and we were contemplating whether to go down the DTC route or not. After considering the two alternatives - and significant research - we decided to choose DTC. This is due to: 
- Time constraints
- Simplicity in terms of development
- Performance/scalability requirements (which meant we would not hit the DTC limits)

Not all products support DTC and some of the ones that do; have issues/bugs in their implementation. Hence, your architecture decision doubles up as a technology contraint, which in some cases is not a problem. But why would you unnecessary restrict yourself? 

In our case, we saw the warning signs early on when we had to choose RavenDB over MongoDB due to its DTC support. Then MSMQ over RabbitMq for the exact same reason. We also knew the services would eventually be hosted in the cloud, so we went for IaaS as this allowed us (at the time) to utilise MSMQ and continue relying on DTC. This last choice proved problematic as we soon realised that we would have to perform a lot of the grunt work when it comes to provisioning and managing nodes. This was more than just an inconvenience for us as we did not have either the resources or the time to resolve this. We needed to move to a PaaS and shift these responsibilities to the cloud host but due to various legal and regulatory red tape, this meant Azure PaaS was the only choice we had. And, at the time, DTC was not something that was supported in Azure's infrastructure. 

Hence, we were left between a rock and a hard place. On one hand, we have to go back to the drawing board and significantly alter the system to move away from DTC, which would be a major set back. Or, alternatively, stick with DTC and invest in provisioning, scripting and managing nodes in an IaaS setup.

In conclusion, try to ditch two-phase commits from early on in your architecture but if you choose to rely on it, then ensure that you are aware of all the constraints you introduce into your system and factor that into the decisions you make and the technologies you use.
