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
- tech stack
  - React front end
  - Rails back end w/ SQLite database
  - front end and back end are separate applications that are hosted separately
- optional nice-to-haves
  - let player choose from more than one image / photo

## Unclear Things To Clarify

- where and how are the images stored?
  - in database as blob files?
  - on backend?
  - on frontend?
- how to define the data model for the backend?
  - image metadata (e.g. dimensions, etc.) if needed?
  - image character positions
  - different images
  - game session scores / run times
- where and how to track the stopwatch timer for each game session?

## Service / Feature Specs

- TODO

## User Workflows

- TODO

## Backend Data Models

- TODO

## Backend Routes

- TODO

## Frontend Routes

- TODO
