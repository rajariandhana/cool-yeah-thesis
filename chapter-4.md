# CHAPTER 4 RESULTS AND DISCUSSION

## 4.1 Backend Starting Point Implementation
[maybe change the definition]
The root of the backend starts from `main.py` file. All application that utilizes FastAPI as their backbone imports and calls the `FastAPI()` function before defining routes. The convention of using a `/api` prefix is added to isolate API traffic from static frontend assets. To maintain a clean structure, each router is grouped into its own file inside the `routes` directory and is added another prefix according to the group. For example the routes for accessing the players are created as `playersRouter` within the `routes/players.py` file. It also uses the `/players` prefix. Suppose a route called `/test` is defined under the players routers, then it will generate a full URI endpoint as `/api/players/ping` that can be accessed.

```py main.py
from fastapi import FastAPI
app = FastAPI()

apiRouter = APIRouter(prefix="/api")

from routes.players import playersRouter
apiRouter.include_router(playersRouter)
app.include_router(apiRouter)
```

```py routes/players.py
playersRouter = APIRouter(prefix="/players")
@playersRouter.get("test")
async def test():
	return "hello world"
```

[TABLE]
| router | file | prefix |
| - | - | - |
| playersRouter | routes/players.py | /players |
| hostRouter | routes/host.py | /host |
| gamesRouter | routes/games.py | /games |

### 4.1.1 Backend Directory Structure
At the root of the backend directory exists the following files and directories each with their purposes.
[MAKE IT INTO TABLE]
```sh
/routes # defines API routes and WebSocket routes
/utils # constant definitions, ...
database.py # endpoint for accessing database
main.py # program entry point
meeting.db # SQL file containing database records
models.py
```

### 4.1.2 Accessing Database
`database.py` configures SQLAlchemy for database access. SQLite database will be used as default database and stored as `meeting.db`. Database engine is created and `SessionLocal` is defined for managing database operations. `get_db()` function can be called from other modules to provide a database session for each request ensuring it is properly closed after use and preventing connection leaks.
```py database.py
load_dotenv()
DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./meeting.db")

if DATABASE_URL.startswith("sqlite"):
    engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
else:
    engine = create_engine(DATABASE_URL)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 4.1.3 Accessing User
The frontend of the platform utilizes Zoom Software Development Kit (SDK) to access interanal Zoom functionality. When requests comes from the frontend it will provide a request header with `role` key that determines whether it comes from a zoom client or not and whether it is the host or meeting participants. `get_db()` now can be used to bypass requests and retrieve the correct user type.
```py utils/auth.py
PARTICIPANT_ZOOM = "PARTICIPANT_ZOOM"
PARTICIPANT_WEB = "PARTICIPANT_WEB"
HOST = "HOST"

def get_user(User: str = Header(None)):
    if not User:
        return None
    try:
        user = json.loads(User)
        if user.get("role") == "host":
            user["role"]= HOST
        elif user.get("role") == PARTICIPANT_WEB:
            user["role"]= PARTICIPANT_WEB
        else:
            user["role"]= PARTICIPANT_ZOOM
        return user
    except json.JSONDecodeError:
        return None
```

### 4.1.4 Defining Models
The `models.py` file defines the application's database schema using SQLAlchemy's Object Relational Mapping (ORM). Each class represents a database table, with attributes mapped to table columns and relationships defining how tables are connected. The `Game` model stores information about each game session, including Zoom meeting details, status, timestamps, and its associated players. The `Player` model stores participant information such as name, role, expertise, score, match history, and interests, while linking each player to a specific game.
```py models.py
class Game(Base):
    __tablename__ = "games"

    id = Column(String, primary_key=True, index=True)
    zoom_host_id = Column(String, nullable=False)
    zoom_meet_id = Column(String, nullable=False)
    zoom_join_url = Column(String, nullable=False)
    status = Column(String, default="CLOSED")
    time_start = Column(DateTime, nullable=True)
    time_end = Column(DateTime, nullable=True)
    created_at = Column(DateTime, default=datetime.now)
    updated_at = Column(DateTime, default=datetime.now, onupdate=datetime.now)

    players = relationship("Player", back_populates="game", cascade="all, delete-orphan")

class Player(Base):
    __tablename__ = "players"

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    zoom_participant_id = Column(String, index=True, nullable=True)
    name = Column(String, nullable=False)

    game_id = Column(String, ForeignKey("games.id"), index=True, nullable=False)
    game = relationship("Game", back_populates="players")

    expertise_id = Column(Integer, ForeignKey("expertise.id"), nullable=True)
    expertise = relationship("Expertise", back_populates="players")

    score = Column(Integer, default=0)
    time_finished = Column(DateTime, nullable=True)

    created_at = Column(DateTime, default=datetime.now)
    updated_at = Column(DateTime, default=datetime.now, onupdate=datetime.now)

    matches = Column(MutableList.as_mutable(JSON), default=list, nullable=False)
    interacted_with = Column(MutableList.as_mutable(JSON), default=list, nullable=False)

    interests = relationship("Interest", secondary="player_interests", back_populates="players")
    role = Column(String, nullable=False)
    mode = Column(String, nullable=False)

```

## 4.2 Implementing Lobby Session
### 4.2.1 Creating New Game Session
Previously defined functions can now be used together for a real platform functionality. [SOURCE CODE X.X] defines the `POST /api/host/{host_id}/create` API route, which allows an authenticated host to create a new game session. It first receives the request body containing the Zoom meeting information, authenticates the user through the `get_user` dependency, and retrieves a database session using `get_db`. The endpoint verifies that the authenticated user has the `"HOST"` role and that the `host_id` in the URL matches the user's Zoom participant ID, preventing unauthorized users from creating games on behalf of others. It then generates a unique game ID using `generate_unique_game_id()`, creates a new `Game` object with the provided Zoom meeting details and an initial status of `"OPEN"`, and stores it in the database. Finally, the new game is committed to the database, refreshed to retrieve any updated values, and returned to the client as the API response.
```py routes/host.py
@hostRouter.post("/{host_id}/create")
def create_game(
    body: dict = Body(...),
    user=Depends(get_user),
    host_id: str = Path(..., description="The ID of the host"),
    db: Session = Depends(get_db),
):
    if user["role"] != "HOST":
        raise HTTPException(status_code=401, detail="Unauthorized")
    if host_id != user.get("participantUUID"):
        raise HTTPException(status_code=403, detail="Forbidden")

    game_id = generate_unique_game_id(db)
    if not game_id:
        raise HTTPException(status_code=500, detail="Could not generate unique game ID")

    game = models.Game(
        id=game_id,
        zoom_host_id=user["participantUUID"],
        zoom_meet_id=body["zoom_meet_id"],
        zoom_join_url=body["zoom_join_url"],
        status="OPEN",
        created_at=datetime.now(),
        updated_at=datetime.now()
    )
    db.add(game)
    db.commit()
    db.refresh(game)

    return {"game": game}
```

### 4.2.2 Joining Game Session as a Player
After the host creates a game session, the game ID will be displayed on the host's side so that the players can use it to join. [SOURCE CODE X.X] `POST /join/{game_id}` API route, which allows a participant to join an existing game. It first retrieves the game from the database using the provided `game_id` and verifies that the game exists, is currently open, and that the authenticated user is not a host, since hosts are not allowed to join as players. If these checks pass, a new `Player` record is created using the participant's information, including their name, Zoom participant ID, selected mode, and role, and is saved to the database.
```py routes/players.py
@playersRouter.post("/join/{game_id}")
async def join_game(
    body: dict = Body(...),
    user=Depends(get_user),
    game_id: str = Path(..., description="The ID of the game"),
    db: Session = Depends(get_db)
):
    game = db.query(Game).filter(Game.id == game_id).first()
    if not game:
        return {"message": "Game not found"}
    if game.status != "OPEN":
        return {"message": "Game is not open"}
    if user["role"] == HOST:
        return {"message": "Hosts cannot join as players"}
    player = Player(
        name=body["name"],
        zoom_participant_id=user["participantUUID"],
        game_id=game_id,
        created_at=datetime.now(),
        updated_at=datetime.now(),
        role=user["role"],
        mode=body["mode"]
    )
    db.add(player)
    db.commit()
    db.refresh(player)

    survey = body["survey"]
    if survey:
        if survey.get("expertise_id"):
            exp = db.query(Expertise).filter(Expertise.id == survey["expertise_id"]).first()
            if exp:
                player.expertise = exp

        player.interests = _get_or_create_interests(db, survey.get("interests", []))

        player.updated_at = datetime.now()
        db.commit()
        db.refresh(player)

    return {"game": {"id": game.id}, "player": player}
```

## 4.3 Implementing Game Session

### 4.3.1 Starting Game Session
[SOURCE CODE X.X] manages the lifecycle of a game by updating its status and recording when gameplay begins. The `POST /{game_id}/status` endpoint retrieves the specified game from the database, validates that it exists, and checks that the requested status is one of the allowed values (`"OPEN"`, `"CLOSED"`, `"PLAYING"`, or `"FINISHED"`). After updating the game's status and timestamp, it clears all previously generated player matches for that game and calls `compute_and_persist_matches_for_game()` to generate and save a new set of player matches based on the current participants. This ensures that the matchmaking data remains consistent whenever the game status changes. The `POST /{game_id}/start` endpoint is responsible for officially starting a game by setting its status to `"PLAYING"`, recording the start time (`time_start`), updating the modification timestamp, and saving the changes to the database. Both endpoints return the updated game information to confirm that the requested operation was completed successfully.

```py routes/games.py
@gamesRouter.post("/{game_id}/status")
def update_game_status(
    body: dict = Body(...),
    game_id: str = Path(..., description="The ID of the game"),
    db: Session = Depends(get_db),
):
    game = db.query(models.Game).filter(models.Game.id == game_id).first()
    if not game:
        return {"message": "Game not found"}

    status = body.get("status", game.status)
    if status not in ["OPEN", "CLOSED", "PLAYING", "FINISHED"]:
        return {"message": "Invalid status"}

    game.status = status
    game.updated_at = datetime.now()
    db.commit()
    db.refresh(game)

    db.query(Player).filter(Player.game_id == game_id).update({Player.matches: []})
    db.commit()

    compute_and_persist_matches_for_game(db, game_id, k=5)

    return {"message": "Game status updated", "game": game}

@gamesRouter.post("/{game_id}/start")
def start_game(
    game_id: str = Path(..., description="The ID of the game"),
    db: Session = Depends(get_db),
):
    game = db.query(models.Game).filter(models.Game.id == game_id).first()
    if not game:
        return {"message": "Game not found"}

    game.status = "PLAYING"
    game.time_start = datetime.now()
    game.updated_at = datetime.now()
    db.commit()
    db.refresh(game)

    return {"message": "Game started", "game": game}
```

### 4.3.2 Updating Player Scores
[SOURCE CODE X.X] defines the `POST /pair-add-score` endpoint, it updates the scores of two players after they successfully complete a match. It first retrieves both the player and their opponent from the database using the provided IDs and returns a `404` error if either record does not exist. The endpoint then increases both players' scores by the predefined `MATCH` value while ensuring that neither score exceeds the maximum `BINGO` score. If a player reaches the maximum score for the first time, the current timestamp is recorded as `time_finished`, allowing the application to track when they completed the game. The endpoint also updates each player's `interacted_with` list by recording the other player's ID, ensuring that previous interactions are tracked and duplicate entries are avoided. Finally, it updates the modification timestamps, commits all changes to the database, refreshes the player objects, and returns the updated records for both participants.

```py routes/players.py
@playersRouter.post("/pair-add-score")
def increase_pair_score(
    body: dict = Body(...),
    db: Session = Depends(get_db)
):
    player = db.query(Player).filter(Player.id == body.get("player_id")).first()
    opponent = db.query(Player).filter(Player.id == body.get("opponent_id")).first()
    if not player or not opponent:
        raise HTTPException(status_code=404, detail="Player or opponent not found")

    player.score = min(player.score + MATCH, BINGO)
    opponent.score = min(opponent.score + MATCH, BINGO)
    if player.score == BINGO and not player.time_finished:
        player.time_finished = datetime.now()
    if opponent.score == BINGO and not opponent.time_finished:
        opponent.time_finished = datetime.now()
    
    if opponent.id:
        current = player.interacted_with or []
        if opponent.id not in current:
            current.append(opponent.id)
            player.interacted_with = current
    if player.id:
        current = opponent.interacted_with or []
        if player.id not in current:
            current.append(player.id)
            opponent.interacted_with = current
    
    player.updated_at = datetime.now()
    opponent.updated_at = datetime.now()
    db.flush()
    db.commit()
    db.refresh(player)
    db.refresh(opponent)
    return {"player": player, "opponent": opponent}
```

### 4.3.3 Ending a Game Session
To end the game the host's client side will call the same `POST /{game_id}/status` route previously defined in [SOURCE CODE X.X]. The only difference is the host will provide the `"FINISHED"` value indicating that the game is finished whether because all players have finished playing or the host ends it on their own accords.

## 4.4 Implementing Real Time Updates With WebSockets
Events such as player joining a lobby, updating game status, moving players in the lobby, and chatting between players requires the WebSocket protocol that supports real time updates to enable a real time communication between host and players. 

The `ConnectionManager` class on `routes/websocket.py` maintains all active WebSocket connections by organizing them according to `game_id`, where each game contains a single host connection and multiple player connections. It also stores each player's latest state—including avatar, position, and facing direction—in the `player_states` dictionary so that newly connected players can immediately synchronize with the current game world. When a client connects, the server accepts the WebSocket connection, determines whether the client is a host or player, registers the connection, initializes the player's default state if necessary, and broadcasts the new player's information to all existing participants. Throughout the game, the server continuously listens for incoming JSON messages, updates player movement and avatar information whenever `"player_move"` or `"avatar_update"` events are received, and broadcasts these updates to all other connected players to keep the game synchronized in real time. 

The message routing logic supports communication between the host, all players, a specific player, or every connected client through a simple messaging protocol consisting of `src`, `dest`, and `message` fields. When a participant disconnects or an error occurs, the connection manager removes the player's WebSocket connection and stored state, and automatically cleans up the game session once no host or players remain connected. This implementation provides a lightweight multiplayer networking layer that ensures all connected clients share a consistent and up-to-date view of the game.

```py routes/websocket.py
webSocketRouter = APIRouter()

player_states = {}

class ConnectionManager:
    def __init__(self):
        self.active_games: Dict[str, Dict[str, WebSocket]] = {}

    async def connect(self, game_id: str, role: str, player_id: Optional[str], websocket: WebSocket):
        if game_id not in self.active_games:
            self.active_games[game_id] = {"host": None, "players": {}}
        if role == "host":
            self.active_games[game_id]["host"] = websocket
            print(f"Host connected to game {game_id}")
        else:
            self.active_games[game_id]["players"][player_id] = websocket
            print(f"Player {player_id} connected to game {game_id}")

        if game_id in player_states:
            db = next(get_db())
            for pid, state in player_states[game_id].items():
                if pid != player_id:
                    p = db.query(Player).filter(Player.id == pid).first()
                    if not p:
                        continue
                    await websocket.send_json({
                        "src": "server",
                        "dest": player_id,
                        "message": {
                            "type": "player_sync",
                            "player_id": pid,
                            "name": p.name,
                            "expertise": p.expertise.name if p.expertise else None,
                            "interests": [{"id": i.id, "name": i.name} for i in p.interests],
                            "avatarUrl": state.get("avatarUrl"),
                            "x": state.get("x"),
                            "y": state.get("y"),
                            "facing": state.get("facing", 0),
                        },
        })

    def disconnect(self, game_id: str, role: str, player_id: Optional[str]):
        if game_id in self.active_games:
            if role == "host":
                self.active_games[game_id]["host"] = None
            elif player_id in self.active_games[game_id]["players"]:
                del self.active_games[game_id]["players"][player_id]
            if (
                self.active_games[game_id]["host"] is None
                and len(self.active_games[game_id]["players"]) == 0
            ):
                del self.active_games[game_id]
                print(f"Cleaned up game {game_id}")

        if game_id in player_states and player_id in player_states[game_id]:
            del player_states[game_id][player_id]
            if not player_states[game_id]:
                del player_states[game_id]

    async def send_to_players(self, game_id: str, message: dict, exclude_player_id: Optional[str] = None):
        game = self.active_games.get(game_id)
        if not game:
            return
        for pid, ws in game["players"].items():
            if pid != exclude_player_id:
                await ws.send_json(message)

    async def send_to_host(self, game_id: str, message: dict):
        game = self.active_games.get(game_id)
        if game and game["host"]:
            await game["host"].send_json(message)

manager = ConnectionManager()

@webSocketRouter.websocket("/ws/games/{game_id}")
async def websocket_endpoint(websocket: WebSocket, game_id: str):
    await websocket.accept()
    role = None
    player_id = None
    try:
        init_msg = await websocket.receive_json()
        role = init_msg.get("role")
        player_id = init_msg.get("player_id") if role != "host" else None
        await manager.connect(game_id, role, player_id, websocket)

        if player_id:
            db = next(get_db())
            p = db.query(Player).filter(Player.id == player_id).first()

            if game_id not in player_states:
                player_states[game_id] = {}
            if player_id not in player_states[game_id]:
                player_states[game_id][player_id] = {
                    "avatarUrl": None,
                    "x": 400,
                    "y": 300,
                    "facing": 0,
                }

            state = player_states[game_id][player_id]

            await manager.send_to_players(
                game_id,
                {
                    "src": player_id,
                    "dest": "players",
                    "message": {
                        "type": "player_sync",
                        "player_id": player_id,
                        "name": p.name if p else None,
                        "expertise": p.expertise.name if p and p.expertise else None,
                        "interests": [{"id": i.id, "name": i.name} for i in p.interests] if p else [],
                        "avatarUrl": state["avatarUrl"],
                        "x": state["x"],
                        "y": state["y"],
                        "facing": state["facing"],
                    },
                },
                exclude_player_id=player_id,
            )

        while True:
            data = await websocket.receive_text()
            try:
                message = json.loads(data)
            except Exception:
                continue

            src = message.get("src")
            dest = message.get("dest")
            payload = message.get("message")
            
            if payload and payload.get("type") in ["player_move", "avatar_update"]:
                pid = src if src != "host" else None
                if pid:
                    if game_id not in player_states:
                        player_states[game_id] = {}

                    current_state = player_states[game_id].get(pid, {})
                    new_state = {
                        "avatarUrl": payload.get("avatarUrl") or current_state.get("avatarUrl"),
                        "x": payload.get("x", current_state.get("x", 0)),
                        "y": payload.get("y", current_state.get("y", 0)),
                        "facing": payload.get("facing", current_state.get("facing", 0)),
                    }
                    player_states[game_id][pid] = new_state

                    await manager.send_to_players(
                        game_id,
                        {
                            "src": pid,
                            "dest": "players",
                            "message": {
                                "type": "player_sync",
                                "player_id": pid,
                                "avatarUrl": new_state["avatarUrl"],
                                "x": new_state["x"],
                                "y": new_state["y"],
                                "facing": new_state["facing"],
                            },
                        },
                        exclude_player_id=pid,
                    )

            if dest == "all":
                await manager.send_to_players(game_id, message)
                await manager.send_to_host(game_id, message)
            elif dest == "host":
                await manager.send_to_host(game_id, message)
            elif dest == "players":
                exclude = src if src != "host" else None
                await manager.send_to_players(game_id, message, exclude_player_id=exclude)
            else:
                await manager.send_to_players(game_id, message)

    except WebSocketDisconnect:
        print("WebSocket disconnected")
        manager.disconnect(game_id, role, player_id)
    except Exception as e:
        print(f"Exception: {e}")
        manager.disconnect(game_id, role, player_id)
```
