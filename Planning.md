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

```json
{
  /* session id string */
  "_session?": true,

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

## Service / Feature Specs

- TODO

## User Workflows

- TODO
