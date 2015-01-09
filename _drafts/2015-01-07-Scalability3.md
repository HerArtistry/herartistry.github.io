---
layout: post
title: Scalability and SOA - Part 3: UI Composition
---

###UI Composition###
If you have not heard of the term before, then I recommend reading [this](http://www.udidahan.com/2012/06/23/ui-composition-techniques-for-correct-service-boundaries/) article by Udi Dahan.

Every service would provide partial views to present the data it has. When a particular screen contains data from different services, then these partial views are mashed-up together to create one "Composite UI" view.

In order to work effectively with event-driven SOA, the business processes are re-modelled as an asynchronous series of events. For example, a user registration process may consist of creating users, creating their log-in credentials, e-mailing the users a link to re-set their password. Traditionally, this may be presented as a wizard with a linear synchronous process where each step needs to be completed in order to move to the next step. In Event Driven SOA, the users' details and credentials are  asynchronously saved at the same time and they share the user ID. Once the user is created by the users service, a UserCreated event containing only the user ID  is fired. The credentials service, which is subscribed to this event, then uses the user ID contained in the event to retrieve the user's credentials, it then creates a reset password token (just a Guid ID) and composes the credentials' specific password reset e-mail. It then sends a command to the e-mail service to send the e-mail. The reason we can fire a UserCreated event with just an ID, is that all the user's relevant credentials information is already saved in the credentials service through the **Composite UI**.

> **Note:** You need to be judicious in your use of commands as they create coupling. More on this in a future post.

Composite UIs lead to loosely coupled services as even services that seem to collaborate in order to create one screen actually do not know about each other. Take the above users registration for example, the user service provides input fields to capture and store the users' details, while the credentials service provides similar inputs for the users' credentials.  Each view knows how to persist itself to its associated service; well, to be more precise, the view delegates this to a client-side controller/service (Note: all the client-side code lives within the service boundaries). In other words, when the users saves the form, each view sends a request to its service, which, in turn, saves the data to its data store. When the users click save, a client side event is fired and each partial view then persists itself.

**Client-side generated IDs:** In order to associate relevant data saved to disparate services, the ID is generated client side when you get to the composite view and then used in all services. E.g. when the end user navigated to the "Create User" screen, a new user ID is generated and used by both the users and credentials services. This way, when the user service is done persisting the user details, it fires an event containing nothing but the user ID. The credentials service can use this ID to pull up the credential information associated with that user.

> **Note:** If the user created event is fired before the credentials service has stored the user's credentials, then you just need to retry again as it will eventually be created. This is how an eventual consistent system works and you need to cater for this. NServicebus has an exponential back-off retry feature that comes in handy in such situations.

>**Note:** You should not enforce referential integrity across service boundaries because it re-introduces coupling.

