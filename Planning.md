# Planning

## MVP Requirements

- TLDR: full stack web game that involves tagging objects in a photo
- game loop
  - initialization tasks (done once per game session at the start):
    - image loads
    - stopwatch timer starts after image loads
    - while the game has not ended, update the stopwatch timer every second
    - display the list of characters or objects in the photo that need to be found by the player
      - each list item shows the thumbnail image and the text name of the character or object
  - if player has found all characters in the photo,
    - end the game
    - disable all gameplay controls
    - exit this game loop
    - let player submit their time / score via a pop-up form
    - refresh the table of high scores or run times
  - player clicks a point `targetPos` in the image
  - move the target box so that its center is `targetPos`
  - move the target box options so that is directly below the target box
  - if player clicks another point in the image,
    - go to the start of the game loop
  - player clicks `name` which is one of the target box options
    - the target box options are a list of the names of characters or objects in the photo that the player must identify
  - `area` = rectangular area that identifies the points that a named character or object occupies in the photo
  - if `targetPos` is in the `area` associated with `name`,
    - if `area` not been found by the player before,
      - mark `area` as found by the player
      - update the visual list of characters that need to be found
        - hide or "cross out" the character found
      - if the player has found all characters (by `areas`) in the photo,
        - end the game (see above at start of game loop)
  - close / hide the target box and its options
- store image(s) locally on backend (not database)?
- tech stack
  - React front end
    - fetches the image hosted on the backend via an API call
    - starts the timer only after the image has loaded
    - sends a GET `/photos/:photo_id/objects/` to the backend to check if a click is within a character's target area or not with the following query string params:
      - `name`, `position_x`, `position_y`
  - Rails back end
    - store images manually in `assets` folder?
    - API for GET `/photos/:id` returns JSON containing the full server path to where the image file is so that the frontend client can access it
      - returned JSON with following fields:
        - `id`, `image_path`, `width`, `height`
    - TODO: mechanism for tracking each non-auth'd player's game session's start and end times
  - SQLite database
    - stores multiple character positions for each image
      - only need top left and bottom right positions for each character's occupied rectangular area on the image
    - stores scores (name, runtime in seconds)
  - front end and back end are separate applications that are hosted separately
- optional nice-to-haves
  - let player choose from more than one image / photo

## Unclear Things To Clarify

- where and how are the images stored?
  - on backend manually under `assets` folder
- how to define the data model for the backend?
  - image metadata (e.g. dimensions, etc.) if needed?
  - image character positions
  - different images
  - game session scores / run times
- where and how to track the stopwatch timer for each game session?
  - use session on backend to track guest players
  - store session in cache (memory on the machine that Rails runs on)
  - sequence: starting the timer
    - client requests the image metadata from the server
    - server creates a session with an id that uniquely identifies the requesting client
    - server updates the session data's `started_at` field with the current time's timestamp
    - server sends the response data with a cookie to the client
    - client gets the response data and with the cookie
    - client renders the image using the response data
    - client sends a request ping to the server that the client finished loading the image
    - server gets the request ping
    - if the request's payload data's timestamp (via the query string data representing the client's timestamp when it finished loading the photo) comes after the session (associated with the client) data's `started_at` value,
      - server updates that session's `started_at` field to the one sent by the client
  - once client loads the photo, the client keeps track and updates is own timer every second equal to the current time's timestamp - the timestamp recorded when the photo loaded on the client
  - sequence: stopping the timer
    - TODO
- how to use session storage on Rails backend that only serves as an API-only app to generate session id's for a separate front-end only client app?
  - something something HTTP session token?
- questions about sessions
  - for guest (unauthenticated) users, do I need to generate the session id for them manually or does Rails make it automatically?
  - how does the previous question apply for a monolith Rails app vs a Rails API-only backend that communicates with a React front-end?
  - how to update the session for a guest user after the frontend client sends some HTTP request? (e.g. user found a character in the photo)
  - how to dispose of the session if the guest user leaves before completing the game?

## Backend Data Models

Models:

- Photo
- Object
- Score

### `Photo` Model

```
photo_name:string [unique, 1-32 chars, present]
id:integer
created_at:datetime
updated_at:datetime

has_many objects
has_many scores
```

### `Object` Model

```
name:string [1-32 chars, present]
image_name:string [unique, 1-32 chars, present]
top_left_x:integer [present]
top_left_y:integer [present]
bot_right_x:integer [present]
bot_right_y:integer [present]
photo_id:integer [present] (FK of `Photo.id`)
id:integer
created_at:datetime
updated_at:datetime

belongs_to photo
```

### `Score` Model

```
player_name:string [unique?, 1-128 chars, present]
started_at:datetime
finished_at:datetime
photo_id:integer [present] (FK of `Photo.id`)
id:integer
created_at:datetime
updated_at:datetime

belongs_to photo
```

## Backend Routes

```
GET     /api/v1/photos/:id
GET     /api/v1/photos/:photo_id/objects (with query string params)
GET     /api/v1/scores
POST    /api/v1/scores
```

## Session / Cookie Data

```js
{
  /* when the player started the game in unix epoch time (in MS) */
  "started_at": 1709669721060,

  /*
     When the player finished the game in unix epoch time (in MS).
     May be null while player is still in-game.
  */
  "finished_at": 1709669868750,

  /* id of the photo that the player is playing on */
  "photo_id": 1,

  /*
     Dict that maps the string name of each object in the photo to a boolean that indicates if the player found the object or not yet.
  */
  "is_object_found": {
    "waldo": false,
    "woof": true,
    "wenda": false
  }
}
```

## Specs

### `Version 0.1` Spec

Definitions:

- Assumes the frontend `client` app exists.
- Assumes the backend `server` app exists.
- Assumes both frontend and backend apps run separately.
- Assumes the frontend and backend apps talk to each via HTTP as defined in a shared API spec.
- Assumes the following frontend duties:
  - Fetches and loads the photo from the backend when it loads.
  - Displays the photo retrieved from the backend.
  - Stores the following local state (game data):
    - original photo metadata
      - url path, original width, original height, id in database
    - rendered photo metadata
      - current width, current height
    - last clicked position (x, y)
      - in the rendered photo
      - normalized to the original photo
  - After the player clicks on a position in the photo, displays and moves a target box with a list of button options to where the position is such that the clicked position is at the center of the target box.
  - Clicking any of the target box options / buttons hides both the options and the target box on the screen.
- Assumes the following backend duties:
  - Hosts the photo(s) (NOT in the database).
  - Has a `Photo` model for interacting with the `photos` table in the database.
  - Has an API route GET `/api/v1/photos/:id` that returns JSON data as an object that includes the path to where the photo is hosted on the server for each photo.

Constraints:

- None

### `Version 0.2` Spec

Includes everything in `Version 0.1` Spec but with new changes as detailed below.

Definitions:

- Assumes the following frontend duties:
  - Fetches and displays both the thumbnail images and names associated with each object / character to find in each photo when the app loads.
  - Stores the following local state (game data):
    - `objects` hashmap of object hashmaps that maps the id of each object in the photo to a hashmap of its metadata e.g.
      - e.g. `1: { id: 1, name: "Waldo", img_path: "waldo.png" }`
    - `foundObjectIds` set containing the id's of each object in the photo found by the player
    - `isPlaying` boolean that indicates if the game is in session or not
  - Handle clicking on a option / button from the list of target box options in the following way:
    - If the game is over (`isPlaying` is true),
      - do nothing and return from the function.
    - Hide the target box and its options UI elements.
    - `res` = response from GET `/api/v1/photos/:photo_id/objects/?id=:id&pos_x=:pos_x&pos_y=:pos_y` as an array of objects
    - If the position did NOT identify an object in the photo (`res` is not an array of size 1),
      - do nothing and return from the function.
    - If the player found this object before (`id` is in `foundObjectIds`),
      - do nothing and return from the function.
    - Add `id` to `foundObjectIds` to mark that object as found by the player.
    - Re-render the list of missing objects / characters.
    - Create a flag / marker UI element at the clicked position to indicate that the object has been found by the player.
    - If the player has found all the missing objects in the photo (size of `foundObjectsIds` equals size of keys in `objects`),
      - end the game & disable any additional player input (i.e. click handlers) on the photo
        - set `isPlaying` to false
- Assumes the following backend duties:
  - Hosts the thumbnail images for each object in each photo (NOT in the database).
  - Has an `Object` model for interacting with the `objects` table in the database.
  - Has an API route GET `/api/v1/photos/:photo_id/objects` that returns JSON data in the form of an array of objects / hashmaps given certain query string parameters.
    - Should return an array of size 1 if the player submitted a valid position that correctly identifies an object in the photo.
    - Returns all rows in the `objects` table that meet the following filters:
      - `object.id` == `params[:id]`
      - `params[:pos_x]` is in the range [`minX`, `maxX`]
        - `minX` = `object.top_left_x`
        - `maxX` = `object.bot_right_x`
      - `params[:pos_y]` is in the range [`minY`, `maxY`]
        - `minY` = `object.top_left_y`
        - `maxY` = `object.bot_right_y`

Constraints:

- TODO

### `Version 0.3` Spec

Includes everything in `Version 0.2` Spec but with new changes as detailed below.

Definitions:

- Assumes the following frontend duties:
  - Fetches and displays the current elapsed time since the game started (i.e. when the photo finished loading)
  - Stores the following local state (game data):
    - TODO
- Assumes the following backend duties:
  - TODO

Constraints:

- TODO
