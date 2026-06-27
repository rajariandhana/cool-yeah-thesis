# CHAPTER 2 LITERATURE REVIEW

## 2.1 Previous Research

## 2.2 Theoretical Foundation

### REST API
https://doi.org/10.51219/JAIMLD/priyanka-gowda/202
https://urfjournals.org/open-access/best-practices-in-rest-api-design-for-enhanced-scalability-and-security.pdf

Representational State Transfer (REST) is a set of rules in designing web services to allow clients request, create, update, or delete data on a server. Since it uses standard HTTP methods, REST is suitable to be used by many different applications. To access or modify a resource, the client will send an HTTP request conforming with the available routes that have been set by the server.

There are naming conventions when creating API routes to make it easy and understandable. It uses nouns for the appropriate resources, for example use `/users` instead of `/getUsers` for routes that access a list of users. To access a certain user we can nest the previous with the unique identifier for that user like `/users/{user_id}`. HTTP methods like `GET`, `POST`, `PUT`, `DELETE` must also be used appropriately when using a service. The same Unique Resource Identifier (URI) like `/users/{user_id}` can be used but can have different results regarding the HTTP method, `GET` `/users/{user_id}` when accessing information of a certain user and `DELETE` `/users/{user_id}` when deleting a certain user.

### Websocket

## FastAPI
FastAPI is a web framework in Python 

## SQLAlchemy
