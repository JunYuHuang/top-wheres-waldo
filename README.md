# Where's Waldo Game

![Gameplay demonstration of a player playing the Where's Waldo game](/game-demo.gif)

This is a web game implementation of the classic children's puzzle book series Where's Waldo. Players need to find all the missing characters in a given photo. Once all characters are found in a photo, the player can submit their score or run time to the game leaderboard.

[Play the game hosted on GitHub Pages](https://junyuhuang.github.io/top-wheres-waldo-client/).

Links:

- [Frontend client application](https://github.com/JunYuHuang/top-wheres-waldo-client)
- [Backend server application](https://github.com/JunYuHuang/top-wheres-waldo-server)

## Architecture Design Overview

This game system follows the basic client-server architecture; both client and server applications are hosted on their own different server machines.

The state of each game session is stored as a cookie-based session. The server creates the cookie and sends it to the client to store. The client never modifies the cookie but only sends it to the server with each request to update the game state. The server is the authoritative source of truth for the game state; only the server modifies the game session and sends the current (possibly updated) game state to the client.

The frontend application is a dummy client that does the following:

- Receives and renders data from the server application.
- Makes requests to the server on behalf of the player including:
  - Updating the start time of the game to when the client finished loading the assets (e.g. images).
  - Correctly and wrongly identified positions of missing characters.
  - Submitting the run time and name for a player who beat the game.
- Maintains local state not relevant to the server (e.g. last clicked position or coordinates in the photo by the player).

The backend application is an API-only server that does the following:

- Maintains and does CRUD operations on the game state on behalf of the client.
- Hosts and serves the static assets (primarily images) that the client needs to render.
- Maintains, stores, retrieves and creates score records on behalf of the client.
- Manages and stores persistent data (primarily the game scores or run times in its on-disk SQLite database).

## Misc

This is a project part of The Odin Project curriculum's [React Course](https://www.theodinproject.com/lessons/react-new-where-s-waldo-a-photo-tagging-app)

Progress of the project is tracked in the [Todos document](./Todos.md)
