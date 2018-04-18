---
layout: post
title: Reference Data in Microservices
comments: true
tags: [EventStreams, SOA, Microservices, Events, shared data, data streams]
---

My current client has a requirement to provide access to reference data in the form of holidays by country, currency holidays and banking information such as refereces, BICs, addresses and so on. This information is required by a few microservices and I was asked to come up with an approach to tackle this.

I come across the question of how to deal with reference data frequently and it seems to cause a lot of confusion when it comes to figuring out how it fits in a microservices architecture. How owns this data? How is it dessiminated? Where does the logic that operate on this data reside? 

In this blog post, I will simply attempt to give an overview of the different options available for dealing with reference data and the advantages and disadvatages of each. The reference data I am talking about here is not static - some of these options do not apply to static data.

#### 1. Shared database
All reference data is stored in a shared database that all consumers can directly access and fetch data from. 

Pros | Cons
--- | ---
- Very simple | - Consumers are tied to database schema
- consumers can shape what data they query  | - Single point of failure
  | - Can become bottleneck
  | - temporal coupling

#### 2. Dedicated Service (API)
An API can be built on top of the shared database to break the external dependy on the the database schema and allow changing the way the data is stored locally. This seems to be the de-facto choice for most clients i have worked with and this client is no different.

Pros | Cons
--- | ---
- simple | - temporal coupling
- familiar | - single point of failure
- centralised business logic (if any) | - can become bottleneck
 | - profileration of endpoints to support consumer needs

Additionally to hiding the database schema, this approach allows re-use of any business logic associated with the reference data. One example, in our case, is calculating when an international payment can be made by checking currency holidays and country holidays for both destination and source countries/currencies. However, on the other hand, it is simply a database exposed over HTTP that suffers from all the disdvantages of a shared database in order to ecapsulate simple business logic.

#### 3. Separate Database Per Service
The idea here is to push down the data directly to the database of the interested services. In order to do this, a shared database schema can be agreed upon, and all the services are expected to create that schema locally. This means the data is available locally and there is no need to make a call over the wire, or be temporally coupled to any other service, when fetching the data.

Pros | Cons
--- | ---
- simple | - every new consumer must be registered with the source service              
- no temporal coupling as data is available locally | - every service must create a strict database schema dictated by the source
- isolation |

* **Separate shared db for every service:** 
this is a variation of the above where instead of creating schemas in the target services' databases, and pushing the data into them, you create a parallel database dedicated to each of the interested services. The data is pushed into these dedicated replicas in order to keep a separation between the two. This is especially relevant if you have multiple database servers with each service having a database server deployed on the same machine and you want to avoid making calls over the wire.
  
* **ETL by every service hosted/invoked by the source:** 
this is a variation of the separate database per service approach but has the advantage of allowing the consumers to control what data to fetch, how to shape it and hence store it. However, it suffers from tying the consumers to the producer of the data and having to modify the producer whenever a new consumer is added.

#### 4. Distributed File
In this approach, the reference data is written to a file, or a list of files, and dessiminated to the services. The services can then either use the files directly every time they require data from it or save it to their databases. This also has the advantages of allowing the services to shape the data the store as fits them best.

Pros | Cons
--- | ---
- simple | - every new consumer must be registered with the source service
- data available locally | - every service would need to be able to process these files
- consumers can transform data and store it as best suits them |
- no temporal coupling |
- no behavioural coupling |
- isolation |

#### 5. Streams
##### a. Kafka or similar technology

Pros | Cons  
--- | ---
- all data is available for every client | - not familiar
- no need to change the producer when there's a new consumer | - new technology is sometimes a hard sell    
- data can easily be pushed closer to the consumers |
- no temporal coupling |
- consumers can reshape/read the data without affecting the producer |
- performance |

##### b. HTTP (AtomPub / feed)
This is similar to the above but the data is published through an atom feed (Atom Pub protocol).

Pros | Cons  
--- | ---
- all data is available for every client | - AtomPub implementation can be confusing
- no need to change the producer when there's a new consumer | - relies on a polling pull model
- data can easily be pushed closer to the consumers |
- no temporal coupling |
- consumers can reshape/read the data without affecting the producer |
- performance |
- familiar underlying technology (HTTP) |

#### 6. Message Queues
##### a. Orchestration
The reference data can be broken down into chunks and sent to the services needing it.
    
Pros | Cons       
--- | ---
- familiar | - new consumers have to be explicitly registered with source
- replayed messages can be targeted to specific consumers  | - no catchup subscription (current consumers would trigger a replay if they want to remodel the data)
- no temporal coupling |         
    
##### b. Broadcast
Similar to the above but messages are broadcasted and cosumers simply subscribe to the messages they are interested in

Pros | Cons      
--- | ---                              
- familiar | - no catchup subscriptions
- new consumers do not have to be explicitly registered with source    | - replayed messages will be broadcasted to everyone 
- no temporal coupling |

### Conclusion
I feel like I am repeating myself here but as I stated in a previous post: as with every architecture decision, you need to weigh the advantages and disadvantages of the different approaches and make an informed decision. Everything decision is a tradeoff and you need to determine what matters most in your situation. There are no right answers, silver bullets or best practices.
