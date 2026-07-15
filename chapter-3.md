# CHAPTER 3 METHODOLOGY

## 3.1 Research Method
The development of the platform['s] overall system including the backend utilizes the Agile method following the requirements set by the DECO3801 course. Using this method the project is split into 3 sprints called as "Interims" over the span of 13 weeks.
[IMAGE]

## 3.2 Materials and Equipment Used
[Change to table]
The author uses a MacBook Pro with 16 GB of memory and 512 GB of storage. The Integrated Development Environment (IDE) Visual Studio Code is used as a platform to write code and manage files. GitHub is used for storing the code and managing repository versions. For the backend, Python programming language is used with importing libraries like FastAPI for creating API routes and websockets and SQLAlchemy for managing database.
ngrok?

## 3.3 Research Procedure

### Project Timeline
blablabla

### Database Design
The platform requires two entities, game and player. The relationship between them is that a game can have many players while a player must belongs to a game.

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
