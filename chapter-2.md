# CHAPTER 2 LITERATURE REVIEW

## 2.1 Previous Research
https://publications.csiro.au/publications/publication/PIcsiro:EP2023-4298/RI1/RT13
https://doi.org/10.1088/2514-3433/acfcb6

Previous literature study by TFOM community provided really great insight on possible solutions for hybrid meetings each provided their own strengths and weaknesses.

### 2.1.1 Disadvantages of Physical Meetings
The significant disadvantage of current physical meetings like conferences are without doubt the sustainability aspect. When participants are required to attend physically, carbon footprints are often neglected. With most vehicles still depend on burning fossil fuels, each participant in each meeting could cost an average of 1–3 tons CO2e for them to travel. It needs to be just 2.3 tons to reach The Paris Agreement that targets to limiting global warming by 2030 and is impossible to achieve when long-distance travel are required. Aside of environmental issues, accessiblity must also be addressed. An individual's disablity, fincancial problems, and geopolitical problems often becomes a factor that makes physical conferences not an option for some people. Physical conferences also tend to lack inclusivity, making people feel safe and welcomed. Safety of junior researchers have been the recent concern of physical conferences, especially for females. While online conferences solves the problem of carbon footprint, accessibility, and safety, online solutions are often less favored as they lack the human connection.

### 2.1.2 Current Solutions
By introducing a virtual 3D space, participants could move around the room and interact with each other imitating physical meetings. Instead of showing their actual faces, participants would choose and create their own avatar to represent themselves. One big room can be created as a space where all participants could be at to meet and network with anyone. Smaller rooms can be created to host smaller meetings with their own agendas. It also makes presenting a 3D model much easier when participants are already in the same 3D space.

While 3D space brings interactivity just like in physical medium, in execution participants expressed their struggle when installing the software. They needed additional time to be familiar with the platform. Additional expense must also be spent when using hardware device like VR headsets for a more immersive experience.

Participants that are set in a more relaxed environment are more likely to interact with each other. Game sessions and ice breaking sessions can be used to maintain connectivity between participants even for those in the online setting. Participants who are matched with those who are similar to them, such as having the same expertise or interests, can use those as a foundation to interact with.The chance of them to interact with each other after the meeting or event had ended becomes more probable.

## 2.2 Theoretical Foundation

### 2.2.1 REST API
https://doi.org/10.51219/JAIMLD/priyanka-gowda/202
https://urfjournals.org/open-access/best-practices-in-rest-api-design-for-enhanced-scalability-and-security.pdf

Representational State Transfer (REST) is a set of rules in designing web services to allow clients request, create, update, or delete data on a server. Since it uses standard HTTP methods, REST is suitable to be used by many different applications. To access or modify a resource, the client will send an HTTP request conforming with the available routes that have been set by the server.

There are naming conventions when creating API routes to make it easy and understandable. It uses nouns for the appropriate resources, for example use `/users` instead of `/getUsers` for routes that access a list of users. To access a certain user we can nest the previous with the unique identifier for that user like `/users/{user_id}`. HTTP methods like `GET`, `POST`, `PUT`, `DELETE` must also be used appropriately when using a service. The same Unique Resource Identifier (URI) like `/users/{user_id}` can be used but can have different results regarding the HTTP method, `GET` `/users/{user_id}` when accessing information of a certain user and `DELETE` `/users/{user_id}` when deleting a certain user.

### 2.2.2. Websocket
https://link.springer.com/chapter/10.1007/978-981-97-4152-6_29
https://doi.org/10.1007/978-981-97-4152-6_29

With the increasing demand of real time communications through the internet, a more sophisticated technology is required to support it. At first the solution was for the client to send requests every certain period of time to ask updates from the server (HTTP polling). Longer gap between requests means slower real time updates while using shorter gaps meaning faster updates but also creates unnecessary requests that congests network traffic.

Websocket protocol allows client and server to send and receive data at real time. Websocket works by establishing a persistent TCP connection between the two endpoints allowing to exchange lightweight Websocket frames when needed. Since it doesn't use the same method as HTTP polling, Websocket results in lower latency and reduced bandwidth usage.

In a chat room scenario application, Websocket can be used to ensure real time communications between the users. A room is created using Websocket for the users to join. Suppose we have user 1 and user 2 already established a Websocket connection in a room. When user 1 wants to chat with user 2, user 1 will send the message to the server, validated by them, and forward the message to user 2. When both users leaves the room, the Websocket room is destroyed and the Websocket connection is discontinued.

### 2.2.3 FastAPI
https://link.springer.com/content/pdf/10.1007/978-1-4842-9178-8.pdf
https://doi.org/10.1007/978-1-4842-9178-8

FastAPI is a web framework in Python created by Sebastian Ramirez for building REST APIs and microservices. Compared to other web frameworks FastAPI provides an easy way for beginners to create web applications especially having Python as it's programming language. FastAPI routings works by calling the FastAPI class' method decorator that corresponds to an HTTP method, provide it with the route name, and define the method that would process the route's request including it's response as the method's return statement. Not only to define HTTP routes, WebSocket protocols are also integrated within FastAPI. This allows developers to develop real-time web applications that scales.

With FastAPI, database management can be handled by Object-Relational Mappers (ORM) like SQLAlchemy. Instead of writing raw Structured Query Language (SQL) queries, each row in a table corresponds to an Python object that can be manipulated. Create Read Update Delete (CRUD) operations between table relations are executed by calling Python methods.
