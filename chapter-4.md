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
The root of the backend starts from `main.py` file. All application that utilizes FastAPI as their backbone imports and calls the `FastAPI()` function before defining routes. The convention of using a `/api` prefix is added to isolate API traffic from static frontend assets. To maintain a clean structure, each router is grouped into its own file inside the `routes` directory and is added another prefix according to the group. For example the routes for accessing the players are created as `playersRouter` within the `routes/players.py` file. It also uses the `/players` prefix. Suppose a route called `/test` is defined under the players routes then it will generate a full URI as `/api/players/ping`.

```py
from fastapi import FastAPI
app = FastAPI()

apiRouter = APIRouter(prefix="/api")
playersRouter = APIRouter(prefix="/players")
@playersRouter.get("test")
async def test():
	return "hello world"
apiRouter.include_router(playersRouter)
app.include_router(apiRouter)
```
