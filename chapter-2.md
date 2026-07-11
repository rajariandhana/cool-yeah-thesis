# CHAPTER 2 LITERATURE REVIEW

## 2.1 Previous Research
https://publications.csiro.au/publications/publication/PIcsiro:EP2023-4298/RI1/RT13
https://doi.org/10.1088/2514-3433/acfcb6

Previous literature study by TFOM community provided really great insight on possible solutions for hybrid meetings each provided their own strengths and weaknesses.

[Disadvantages of Physical Meetings]
The significant disadvantage of current physical meetings like conferences are without doubt the sustainability aspect. When participants are required to attend physically, carbon footprints are often neglected. With most vehicles still depend on burning fossil fuels, each participant in each meeting could cost an average of 1–3 tons CO2e for them to travel. It needs to be just 2.3 tons to reach The Paris Agreement that targets to limiting global warming by 2030 and is impossible to achieve when long-distance travel are required. Aside of environmental issues, accessiblity must also be addressed. An individual's disablity, fincancial problems, and geopolitical problems often becomes a factor that makes physical conferences not an option for some people. Physical conferences also tend to lack inclusivity, making people feel safe and welcomed. Safety of junior researchers have been the recent concern of physical conferences, especially for females. While online conferences solves the problem of carbon footprint, accessibility, and safety, online solutions are often less favored as they lack the human connection.

[Current Solutions]

[DRAFT: sentences from TFOM paper]
- unstructured sessions such as games and shared breaks can be used to keep the online participants connected
- virtual reality to provide a social venue and exhibition space
- The conference organizers were keen to provide an area where attendees could interact
- The venue could be accessed via a computer or VR headset
- What worked well: The exhibition space was particularly successful at encouraging people to attend the virtual socials, and the models and exhibits were good ice breakers for seeding conversation.
- Limitations: A few participants were not able to access the virtual space from their computer, due to struggles with installing the software. This could have been solved by stronger encouragement to install the software in advance, or adopting a platform that was browser-based.
- The most promising of these, Extended Reality (XR), brings huge improvements in interaction quality, particularly in making users feel connected and making them feel like they are really ‘there,’ while also opening up opportunities to go beyond what is possible in person.
- But the biggest issue with XR spaces is that most people’s experience with them is via a backwards compatibility option (like a laptop or phone), and much like a video call where 90% of the attendees are dialed in by landline, the benefits of the better technology are not apparent unless the majority of users are connected using the new approach. The technology is also new to many, causing its own learning curve issues and triggering risk aversion strategies when doing larger events. In short, while XR conferences can solve many of the issues we have with online meetings, they unfortunately (much like video call conferences and meetings before the pandemic) currently suffer from a critical mass problem.
- Already one can have a decent fully-functioning standalone headset for a few hundred USD, or a spectacular headset with a supporting PC for a few thousand

// TFOM conducted case studies in the design of online and hybrid meetings.

There are features that have been implemented in order to [add/place/fit/fix/mimick/...] the missing human connection.

By introducing a virtual 3D space, participants could move around the room and interact with each other [just like: find good synonym] in physical meetings. Instead of showing their actual faces, participants would choose and create their own avatar to represent themselves. One big room can be created as a space where all participants could be at to meet and network with anyone. Smaller rooms can be created to host smaller meetings with their own agendas. It also makes presenting a 3D model much easier when participants are already in the same 3D space.

While this brings interactivity just like in physical medium, in execution participants expressed their struggle when installing the software. They needed additional time to be familiar with the platform. Additional expense must also be spent when using hardware device like VR headsets for a more immersive experience.

Another feature that 

While this bridges the awkward social interaction current meetings
TFOM XR

browser based

...blablabla... thus it is necessary for... requires...
easy to install, plug and play kind off

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

## FastAPI
https://books.google.com.au/books?hl=en&lr=&id=XZmGEAAAQBAJ&oi=fnd&pg=PP1&dq=fastapi&ots=4mU_W_r9vB&sig=jFZVpxOoHYKA-kDtaevXVcT1fyE&redir_esc=y#v=onepage&q=fastapi&f=false

FastAPI is a web framework in Python created by Sebastian Ramirez for building REST APIs and microservices. FastAPI has it's own 

## SQLAlchemy
https://books.google.com.au/books?hl=en&lr=&id=XZmGEAAAQBAJ&oi=fnd&pg=PP1&dq=fastapi&ots=4mU_W_r9vB&sig=jFZVpxOoHYKA-kDtaevXVcT1fyE&redir_esc=y#v=onepage&q=fastapi&f=false

TO CONSIDER:
- is FastAPI and SQLAlchemy necessary? FastAPI is just an application of REST, SQLAlchemy is just ORM
