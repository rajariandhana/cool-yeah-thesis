# CHAPTER 2 LITERATURE REVIEW

## 2.1 Previous Research

## 2.2 Theoretical Foundation

### REST API
https://doi.org/10.51219/JAIMLD/priyanka-gowda/202
https://urfjournals.org/open-access/best-practices-in-rest-api-design-for-enhanced-scalability-and-security.pdf

Representational State Transfer (REST) is a set of rules in designing web services to allow clients request, create, update, or delete data on a server. Since it uses standard HTTP methods, REST is suitable to be used by many different applications. To access or modify a resource, the client will send an HTTP request conforming with the available routes that have been set by the server.

There are naming conventions when creating API routes to make it easy and understandable. It uses nouns for the appropriate resources, for example use `/users` instead of `/getUsers` for routes that access a list of users. To access a certain user we can nest the previous with the unique identifier for that user like `/users/{user_id}`. HTTP methods like `GET`, `POST`, `PUT`, `DELETE` must also be used appropriately when using a service. The same Unique Resource Identifier (URI) like `/users/{user_id}` can be used but can have different results regarding the HTTP method, `GET` `/users/{user_id}` when accessing information of a certain user and `DELETE` `/users/{user_id}` when deleting a certain user.

### Websocket
https://link.springer.com/chapter/10.1007/978-981-97-4152-6_29
https://doi.org/10.1007/978-981-97-4152-6_29

With the increasing demand of real time communications through the internet, a more sophisticated technology is required to support it. At first the solution was for the client to send requests every certain period of time to ask updates from the server (HTTP polling). Longer gap between requests means slower real time updates while using shorter gaps meaning faster updates but also creates unnecessary requests that congests network traffic.

Websocket protocol allows client and server to send and receive data at real time. Websocket works by establishing a persistent TCP connection between the two endpoints allowing to exchange lightweight Websocket frames when needed. Since it doesn't use the same method as HTTP polling, Websocket results in lower latency and reduced bandwidth usage.

In a chat room scenario application, Websocket can be used to ensure real time communications between the users. A room is created using Websocket for the users to join. Suppose we have user 1 and user 2 already established a Websocket connection in a room. When user 1 wants to chat with user 2, user 1 will send the message to the server, validated by them, and forward the message to user 2. When both users leaves the room, the Websocket room is destroyed and the Websocket connection is discontinued.

## FastAPI
FastAPI is a web framework in Python 

## SQLAlchemy
