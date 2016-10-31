---
layout: post
title: Ditching DTC - Event Sourcing on SQL Server and Websphere MQ
comments: true
tags: [Event Sourcing, Transactions, DTC, MSDTC]
---

I have been working on an event-sourced system where we, due to red tape, had no option but to use the technologies already utilised by the company, namely: Sql Server and Websphere MQ. In an event-sourced system commands come in and at least one event comes out. Commands are handled by a command handler that invokes methods on an aggregate, the aggregate determines whether the command can be processed and creates the events to fire. When that aggregate is saved, it is both persisted to Sql Server and a message is published over websphere MQ.

The immediate problem here, is how to ensure that these operations either all succeed or fail. On way of doing this is to use DTC but we wanted to stay away from that as we did not want to be constrained to technologies that support DTC and also inhibit out throughput. Granted we may never hit the limits of DTC but why even risk at as the demands of system we were working on were predicted (?) to go through the roof by projected business growth. ????? REVIEW ALL 
