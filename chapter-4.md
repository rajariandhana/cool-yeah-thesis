# CHAPTER 4 RESULTS AND DISCUSSION

## 4.1 Research Results
[CONSIDER: change to something else]

## 4.2 Discussion
[CONSIDER: change to something else]

## /Backend Directory Structure
At the root of the backend directory exists the following files and directories each with their purposes.
[MAKE IT INTO TABLE]
```sh
/routes # defines API routes and WebSocket routes
/utils # constant definitions, ...
database.py # endpoint for accessing database
main.py # program entry point
meeting.db # SQL file containing database records
models.py
schemas.py
```

## /API Implementation
### Starting Point
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

## Accessing Database
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

## Accessing User
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

## Defining Models
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

## Lobby Session
### Creating New Game Session
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

### Joining Game Session as a Player
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


