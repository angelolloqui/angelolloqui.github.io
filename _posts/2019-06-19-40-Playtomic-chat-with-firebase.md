---
layout: post
title:  "Playtomic's chat solution with Firebase Realtime DB"
date:   2019-06-19 11:30:34
categories: 
    - firebase
    - chat
    - architecture    
    - playtomic
permalink: /blog/:title
---

A few weeks ago we posted a question in our [Playtomic dev blog](https://dev.to/playtomic) and our developer network asking for chat solutions. Comments were broad, and we finally opted for a combination of [Firebase](https://firebase.google.com) + [MessageKit](https://github.com/MessageKit/MessageKit)/[ChatKit](https://github.com/stfalcon-studio/ChatKit). But let me make a quick recap of the different solutions available for each of the components (backend + apps):

## Existing solutions

### Backend

#### Chat specialized backends
Products like [Twilio](https://www.twilio.com/chat), [Sendbird](https://sendbird.com/) or [Pusher](https://pusher.com/chatkit) offer a chat based API and set of libraries to handle the chat connection, messages, online presence, etc. 

#### Synched Cloud Databases
In parallel to chat backend solutions there are also some general purpose online databases with real time synching that allow you to implement any feature, including chats, with your own data model. Some examples are [Firebase Realtime DB](https://firebase.google.com/products/realtime-database), [Firestore](https://firebase.google.com/products/firestore), [Realm Platform](https://realm.io/products/realm-platform) or [AppSync](https://aws.amazon.com/appsync/)

#### Own Service
You can always create your own chat server infrastructure using some real time transport technology such us WebSockets or XMPP.

### Frontend / Mobile apps:

#### Open source UI components
You can use existing chat libraries for your UI instead of implementing your own. Two of the most popular and mature options are [MessageKit](https://github.com/MessageKit/MessageKit) for iOS and [ChatKit](https://github.com/stfalcon-studio/ChatKit) for Android. They provide most UI components you would expect from a chat.

#### Own UI
Chat screens are, in general, not that complex from a UI perspective. As such, you could decide to implement your own views and handle text messages, images, locations,... yourself.

### Full product
Lastly, there are also some products like [ChatSDK](https://chatsdk.co/), [Cometchat](https://www.cometchat.com/), [Chat21](http://www.chat21.org/) or [Chatcamp.io](https://chatcamp.io/) that provide a fully featured solution, from backend to frontend. They normally provide a mobile SDK with full UI ready to be added to your project in a very quick and easy way, together with some degree of UI customization and public APIs to manage the data. 


## Solution choice for Playtomic

For [Playtomic](https://playtomic.io), we wanted a solution that fulfills our needs:

- **Pricing**: Due to the nature of our app we have many thousands of “connected” users, but where most of them are just using the app with a different purpose than chatting. However, since every app user can potentially have unread messages, they will all need to connect to check for unreads, even if in the vast moyority of them will have none. Therefore, we need a solution where we can have thousands of readers per month with a very small amount of writers in a price efficient way, preferably within a free tier.

- **Customization**: We started the chat as an experiment, but we foresee a future where the chat can enable much more complex interactions that just writing/reading messages. For example, after a match, we could use the chat to suggest users to rematch, rate the other players, fill in the results, suggest similar open matches… Choosing a solution that enables us to customize messages and program rich bot interactions will be important if the experiment succeeds

- **Complexity**: As mentioned, our chat feature is an experiment, a new channel for players to communicate with each other within the app, but it is far from our product’s core. For that reason, it was important to have an MVP in reasonable development time (1 sprint / 2 weeks) and with low operational cost.

We started exploring ChatSDK as it provided what it looked like a good balance between **pricing** (Firebase Realtime DB is used as backed, which has a decent free tier), **customization** (being open source and a general purpose DB on backend allows for any change needed) and **complexity** (full product, backend and frontend, should be quick to integrate). However, after a couple of days integrating it in our existing app we found some issues (crashes, bad designed code, difficulties customizing appearance, platform differences,...) which made us pivot to what it is now our current setup: [Firebase Realtime DB](https://firebase.google.com/products/realtime-database) + [MessageKit](https://github.com/MessageKit/MessageKit) / [ChatKit](https://github.com/stfalcon-studio/ChatKit)


### Data Model + Firebase Functions

When using Firebase RealtimeDB (RTDB) you have to model your data with some constraints in mind:

- **Access control**: RTDB provides some granular access control rules, where you can specify restrictions using user’s auth information, accessing fields on child nodes, parents or siblings, etc. However, note that it does not work from `bottom -> top` but the other way `top -> bottom`. This means that if you have a rule granting access to, for example, `/threads/` to user A, even if you make another rule inside `/thread/thread-1` denying it, the user A will still be able to fetch it since it has access to the parent node. This restriction is especially important when you consider the data model, since it forces you to split your data in different collections if you want to provide a fine grained access control like private chats while still be able to fetch all public threads for example.

- **Querying**: RTDB has some basic s   orting and filtering functions, but very far from what anyone would expect for a database. In particular, you can filter or sort on a concrete value and paginate results with limit options, but you can not make complex queries combining multiple fields or expressions. Similarly to access control, this limitation can be minimized by denormalizing your model into one that already fits your filtering like, for example, creating user indexes, public indexes,...

- **Data Transfer**: One of the most costly mistakes when using RTDB is not caring about transferring. Remember that you can not make complex queries, aggregations, retrieve partial documents or restrict access on a child level. As a result, if you would access to something like `/threads` to count the amount of existing threads, you would download the whole tree content and therefore consume enormous amounts of data on your clients (which will be later accordingly charged by Firebase). Once again, denormalization is normally the way to go to create partial documents, precalculated aggregations, user indexes, etc.

![Denormalization everywhere](/images/posts/40/denormalize.jpg){:class="img-responsive"}

We based our data model on the one provided by [ChatSDK](https://chatsdk.co/), with a few modifications to solve the restrictions mentioned above. Our final data model looks like:

- `/devices`: Here we store information of push notification tokens per device and per user
- `/messages/{threadId}`: We have our messages in its own index to allow low level access control per thread and to permit consumption of threads without downloading all the message list (to show user’s thread list UI)
- `/threads`: Here we store threads metadata like the name, users, last message,... It is our “main” document for threads and used by the denormalization processes. It has restricted access control on a per thread level, and it is the one modified by our backend when chat updates like new players joining a match happens.
- `/user-threads/{userId}`: This our main entry point for all our users. It consists of a denormalized index of the /threads collection containing only those that are “viewable” by the user. Each time there is a new thread or an update on an existing thread under /threads, there is a denormalization function making a “copy” on all participants indexes.
- `/users/{userId}`: In a similar way than user-threads, this denormalized index contains information like the unread count or the user’s online presence. Once again, when there is a new message on the /messages collection, all users in the affected thread get their unread count increased (unless they are online and viewing the thread). This allows us to read this only document to fetch chat status when the application starts, and configure the unread badge on the app tabbar with the proper number without consulting the whole user thread list.

For the bookkeeping of all the denormalization mentioned above, **Firebase provides Functions**, which are basically javascript functions that can get triggered in different circumstances (like insert/delete/update of RTDB nodes) and perform actions like sending push notifications, modifying data,.... In particular, we have the following functions:

- **Create `/messages/{threadId}/{messageId}`**: Triggered when new messages are written, this function is in charge of sending push notifications, copying this last message into the thread’s last message denormalization, make the denormalized copy on user-thread index and increase user aggregations for unread counts.
- **Create `/threads/{threadId}/users/{userId}`**: Triggered when a new user joins a thread, it creates a denormalized copy on the user-threads index
- **Delete `/threads/{threadId}/users/{userId}`**: Triggered when a user leaves a thread, it deletes any denormalized copy from the user-threads index and refreshes the user unread count accordingly.


### Analytics
After a few weeks since going live with the chat, let's quickly explore some usage analytics for the last 30 days:

#### Number of users
- ~3000 chat openings from match detail, where approximately 30% are not players (visitors)
- ~2400 chat openings from profile

#### Number of messages: 

![Number of messages](/images/posts/40/messages.png){:class="img-responsive"}

- ~2700 total messages, 
- ~90 message per day average
- Peaks of almost 250 messages per day. 
- Increasing trend

#### Number of chats: 

![Number of chats](/images/posts/40/threads.png){:class="img-responsive"}

- 436 chats with messages. 
- 70 chats having at least 10 messages 
- 45 chats representing around half of the total messages sent.


### Firebase consumption
After explaining the current chat usage and the denormalized data model used to reduce the Firebase bill, let's take a look on some of the [most critical quotas](https://firebase.google.com/pricing) for the last 30 days to see how we made so far:

#### Function invocations

![Function invocations](/images/posts/40/functions.png){:class="img-responsive"}

With a free quota of 125K/month and a price of 0.4$ per million, we are using around 5% of it and very far from paying any noticeable cost from this concept.

#### RTDB storage

![Storage](/images/posts/40/storage.png){:class="img-responsive"}

This is probably one of the most critical, since the amount of space used can only grow with time (unless we delete old chats) and it increases faster than chat usage due to denormalization. Our current use is about 8MB, with a free tier of 1GB and 1$ per GB extra. So, right now, we are below 1% of usage, with an increasing ratio of around 5MB a month, very far as well from exceeding quota at this time but probably out of quota in a not so distant future if the chat usage continues its growing trend.

#### RTDB transfer

![Download transfer](/images/posts/40/downloads.png){:class="img-responsive"}

Download traffic is the most worrying quota and by far the most abused one by other projects, in some cases causing billings of [many thousands dollars per month](https://hackernoon.com/how-we-spent-30k-usd-in-firebase-in-less-than-72-hours-307490bd24d). However, in our case, with all the denormalization we made, we have used 800Mb out of the 10Gb/month free tier. This leaves us with room to grow about 10x within the free quota and with a potential cost of 1$/GB after that limit. We expect to exceed the quota in a future, but hopefully keeping the cost relatively low.


## Conclusion

So far we are **very satisfied** with the results. The combination of Firebase RTDB + open source UI kits, allowed us to build a **customizable solution** within our **2 week development** timebox. Besides, chats are behaving nicely so far, with **low latency** and **offline support**, while we are still pretty far from leaving the **free tier**. Of course, we did not implement many of the advanced options that other chats provide, things like read-checks, mute chat, typing indicators,... will have to wait, but we have the basis ready to continue development if wanted. 

Chances are in the future we will run into other issues due to denormalization, but so far we have a decent solution in the hands of our users while we explore if the expirement is worth to continue investing on it (as it looks like so far based on usage).