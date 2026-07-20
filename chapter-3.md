# CHAPTER 3 METHODOLOGY

## 3.1 System Design

### 3.1.1 Project Flow
The development of the platform's overall system including the backend utilizes the Agile method following the requirements set by the DECO3801 course. Using this method the project is split into 3 sprints called as "Interims" over the span of 13 weeks where each interim has their different set of goals.
[IMAGE]

### 3.1.2 Materials and Equipment Used
[Change to table]
The author uses a MacBook Pro with 16 GB of memory and 512 GB of storage. The Integrated Development Environment (IDE) Visual Studio Code is used as a platform to write code and manage files. GitHub is used for storing the code and managing repository versions. For the backend, Python programming language is used with importing libraries like FastAPI for creating API routes and websockets and SQLAlchemy for managing database.

### 3.1.3 General View of The Platform
As an ice-breaking platform, participants have opportunities to network and get to know one another before deciding if they wish to further share contact information on their own. The platform serves as the intermediate step between being complete strangers to considering the exchange of contact information. To expedite this ice-breaking process, the platform matches users according to their shared interests and area of expertise. This way, users (regardless of their mode of attendance) will have a common ground when conversing with one another. This eases most of the initial awkwardness and tension.

While waiting for the remaining participants to join the website, existing participants can roam freely in a virtual lobby and engage with others. Subsequently, they will be grouped accordingly and put into breakout rooms where they participate in a Bingo networking game. This Bingo networking game provides multiple opportunities for users to converse with others that have similar interests and areas of expertise. For users that are offline, they will be prompted to meet face to face. Otherwise, a Zoom meeting will automatically commence for them to join and meet. Instead of having to find a topic to strike up a conversation with someone completely unknown, users get to interact directly with their matches.

## 3.2 System Architecture
The platform has 2 sides of users, a host side and a player side. The host can manage the meeting and the players while the players can network with each other. After a host creates a meeting it creates a game session with a game code that the participants can use to join.
[INSERT: flowchart of the app for host and player]

### Backend Routes Requirements
To support the required features of the platform the following routes are designed to be built.
| HTTP Method | URI | Description |
| - | - | - |
| `POST` | `/api/host/{host_id}/create` | Creates a new game session for an authenticated host using the provided Zoom meeting details and generates a unique game ID. |
| `POST` | `/api/players/join/{game_id}` | Allows a participant to join an open game, creates a player record, and stores the player's profile, expertise, and interests. |
| `POST` | `/api/games/{game_id}/status` | Updates the game's status (e.g., `"OPEN"`, `"CLOSED"`, `"PLAYING"`, `"FINISHED"`) and regenerates player matches when required.|
| `POST` | `/api/games/{game_id}/start` | Starts the game by changing its status to `"PLAYING"` and recording the game start time. |
| `POST` | `/api/players/pair-add-score` | Updates the scores of two matched players, records their interaction, and stores the completion time if either player reaches the winning score. |
| `WebSocket` | `/api/ws/games/{game_id}` | Establishes a real-time communication channel for synchronizing player movement, avatar updates, and game events between the host and connected players. |

### Database Requirements and Design
The platform requires two entities, game and player. The relationship between them is that a game can have many players while a player must belongs to a game.

To manage the meeting session, a table called `Game` is required [TABLE X.X]. `Game` contains a unique identifier of the game which the players can use to join. Each game must have a status that declares whether the game is open for players to join.

#### Game Table
| Field           | Type       | Description                   |
| --------------- | ---------- | ----------------------------- |
| `id`            | `string`   | Unique game identifier        |
| `zoom_host_id`  | `string`   | Zoom host ID                  |
| `zoom_meet_id`  | `string`   | Zoom meeting ID               |
| `zoom_join_url` | `string`   | Zoom join URL                 |
| `status`        | `string`   | Status of game                |
| `time_start`    | `datetime` | Start time of game            |
| `time_end`      | `datetime` | End time of game              |
| `expertise`     | `JSON`     | Expertise data in JSON format |
| `created_at`    | `datetime` | Game created at               |
| `updated_at`    | `datetime` | Game last updated at          |

Since there are many players that can join a session, the table `Player` is necessary to store the players with unique identifiers [TABLE X.X]. When a participant joins a game using the code, a new Player is created in the database containing it's relation with the Game table using the game id. Players in offline setting and online setting must also be distinguished.

#### Player Table
| Field                 | Type          | Description                                                           |
| --------------------- | ------------- | --------------------------------------------------------------------- |
| `id`                  | `integer`     | Unique player identifier                                              |
| `zoom_participant_id` | `string`      | Zoom participant ID                                                   |
| `name`                | `string`      | Player's name                                                         |
| `game_id`             | `string`      | Associated game ID                                                    |
| `expertise_id`        | `integer`     | Expertise ID                                                          |
| `score`               | `integer`     | Player's score                                                        |
| `time_finished`       | `datetime`    | Time when the player finished                                         |
| `matches`             | `MutableList` | List of a player's matches (`k = 5`)                                  |
| `interacted_with`     | `MutableList` | List of players this player has interacted with during the bingo game |
| `created_at`          | `datetime`    | Player created at                                                     |
| `updated_at`          | `datetime`    | Player last updated at                                                |
| `role`                | `string`      | Player role                                                           |
