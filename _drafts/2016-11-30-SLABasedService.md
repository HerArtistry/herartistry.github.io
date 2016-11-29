---
layout: post
title: SLA-Based Services
comments: true
tags: [SOA, SLA, Coupling]
---

There was a recent discussion at work around how to implement an e-mail service in SOA. The proposal was to move away from messaging into an HTTP-based endpoint. The rational behind it is that the e-mail service should be listening and reacting to every event out there and SLAs should be the responsiblity of the consumer. Basically, the consumer would call the HTTP endpoint and if that does not succeed, it's then up to the callers to determine what that means to their SLAs.

Proposal: SLA-based queues for command messages. An e-mail service is a shared service. It provides a functionality that is shared among many services; therefore, it's alrigh to be behaviouraly coupled to these kins of services. The consumer wants to send an e-mail and knows there is a service out there that sends e-mails. 

Why no prioritization?

Benefits of this approach

Implementation Overview
