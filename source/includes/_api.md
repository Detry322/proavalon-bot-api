# API Methods

## Get bot information

> Your API should return a response like this:

```json-doc
{
  "name": "DeepRole",
  "chat": false,
  "capabilities": [
    {
      "numPlayers": [5],
      "ladyOfTheLake": true,
      "roles": ["percival", "mordred", "oberon"]
    },
    {
      "numPlayers": [6, 7],
      "ladyOfTheLake": false,
      "roles": []
    }
  ]
}

// This response indicates:
//    - Your bot's name is DeepRole.
//    - Your bot cannot interact with chat.
//    - Your bot can support the following game types:
//       - Games with 5 players
//          - May include lady of the lake
//          - May include Percival, Mordred, or Oberon as special roles.
//       - Games with 6 or 7 players
//          - May *not* include lady of the lake
//          - May *not* include any special roles like Oberon, Percival, or Mordred.
```

This provides ProAvalon with the capabilities and limitations of your bot.

### HTTP Request

`GET /v0/info`

### Request Object Schema

No request object is passed to this endpoint.

### Response Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`name` | String | The name of your bot to be displayed on the site.
`chat` | Boolean | If set to true, this indicates your bot can interact with chat messages.
`capabilities` | Array of Capability | An array describing the capabilities of your bot. ProAvalon can include your bot in games that match any of the descriptions.

### Capability Schema

Parameter | Type | Description
--------- | ------- | -----------
`numPlayers` | Array of integer | The number of players that this capability schema applies to. Multiple values are allowed.
`ladyOfTheLake` | Boolean | If set to true, indicates that your bot can handle games with lady of the lake.
`roles` | Array of string | A list of special roles that your bot can handle. Valid values are: `percival`, `mordred`, `morgana`, `oberon`. It is assumed your bot can handle the Merlin, Assassin, Loyal Servant of Arthur, and Minion of Mordred roles.

### HTTP Response Codes

Code | Meaning
------------------ | -------
`200 OK` | This endpoint should always return HTTP Response 200.

<aside class="success">
Make sure to test your bot to ensure it can handle all of the capabilities you list!
</aside>

## Start a game session

> ProAvalon will send a payload like this:

```json-doc
{
  "numPlayers": 6,
  "roles": ["percival", "morgana"],
  "ladyOfTheLake": false,
  "teamLeader": 2,
  "players": [
    "ExamplePlayer1",
    "CoolPlayer",
    "AwesomeDude212",
    "DeepRole#23432",
    "CoolPerson",
    "Detry322"
  ],
  "name": "DeepRole#23432",
  "role": "merlin",
  "see": {
    "spies": ["CoolPlayer", "CoolPerson"],
    "merlins": []
  }
}

// This indicates ProAvalon wishes to start a game session:
//    - With 6 players.
//    - With the Percival and Morgana special roles in play.
//    - Where lady of the lake is *not* in play.
//    - Where the team leader (first proposer of the game) is player 2.
//    - Where your bot's username is DeepRole#23432.
//    - Where the players above are playing.
//    - Where you are merlin and you perceive "CoolPlayer" and "CoolPerson" to be evil.
```

> Your API should return a response like this:

```json-doc
{
  "sessionID": "be565d891243703c22e1"
}

// This sessionID will be included in subsequent requests referring to this game.
```

This endpoint creates a game session for use with your bot. It will be called at the beginning of each game, before the game starts.

### HTTP Request

`POST /v0/session`

### Request Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`numPlayers` | Integer | The number of players in this game
`roles` | Array of string | The list of special roles in use during this game. See "Get Bot Information" endpoint for valid values.
`ladyOfTheLake` | Boolean | If set to true, the Lady of the Lake is in play during this game session.
`teamLeader` | Integer | The (0-indexed) index of the first player to propose during this game.
`players` | Array of string | The list of players participating in this game, ordered by seat number.
`name` | String | The name of your bot during this game.
`role` | String | The role of your bot in the game. This also determines your alliance. One of: `servant`, `merlin`, `percival`, `minion`, `assassin`, `mordred`, `morgana`, `oberon`
`see` | Object | Your perspective about other players.

### `see` Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`spies` | Array of string | The list of players you perceive to be spies.
`merlins` | Array of string | The list of players you perceive to be merlin.

### Response Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`sessionID` | String | A unique session ID describing this game. To be used in subsequent calls to this API.

### HTTP Response Codes

Code | Meaning
------------------ | -------
`200 OK` | Your API successfully created a session
`400 Bad Request` | Your API was unable to create a session, since your bot does not support this game type.
`401 Unauthorized` | Your API was provided an incorrect `Authorization` header.

## Get bot action

> ProAvalon will send a payload like this:

```json-doc
{
  "sessionID": "be565d891243703c22e1", // The session ID from create session endpoint
  "gameInfo": {} // The large game info object - TODO: Add an example object
}
```

> During proposal phases, your bot should return a response like this:

```json-doc
{
  "buttonPressed": "yes",
  "selectedPlayers": [
    "ProNub",
    "Detry322"
  ]
}

// This proposes a team with players "ProNub" and "Detry322"
```
> During voting phases, your bot should return a response like this:

```json-doc
{
  "buttonPressed": "yes", // or "no"
}

// This approves the proposed mission.
```
> During mission phases, your bot should return a response like this:

```json-doc
{
  "buttonPressed": "yes", // or "no"
}

// This succeeds the mission.
```
> During merlin selection phase, your bot should return a response like this:

```json-doc
{
  "buttonPressed": "yes",
  "selectedPlayers": [
    "Detry322"
  ]
}

// This selects Detry322 as the merlin.
```


Using this endpoint, you return you bot's action to ProAvalon. This endpoint is called every time your bot needs to make an action.

### HTTP Request

`POST /v0/session/act`

### Request Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`sessionID` | String | The session ID describing the game being played.
`gameInfo` | GameInfo | A large object describing the full state of the game.

### Response Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`buttonPressed` | `"yes"` or `"no"` | Specifies the button your bot will press. During different game phases, this will mean different things.
`selectedPlayers` | Array of string | If the current game phase requires your bot to select players, this specifies the players to be selected.

### HTTP Response Codes

Code | Meaning
------------------ | -------
`200 OK` | Your API successfully returned a move.
`401 Unauthorized` | Your API was provided an incorrect `Authorization` header.
`404 Not Found` | Your API could not find a session with the given session ID.


## End a game session

> ProAvalon will send a payload like this:

```json-doc
{
  "sessionID": "be565d891243703c22e1", // The session ID from create session endpoint
  "reason": "complete",
  "gameInfo": {} // The final Game Info object for this game.
}
```

> There is no need for your API to return a response other than 200 OK

This endpoint is used to help clean up the information used during play. It is called whenever a game is finished.

### HTTP Request

`POST /v0/session/delete`

### Request Object Schema

Parameter | Type | Description
--------- | ------- | -----------
`sessionID` | String | The session ID describing the game being played.
`reason` | `"complete"` or `"error"` | The reason this game was ended.
`gameInfo` | GameInfo | A large object describing the full state of the game.

### Response Object Schema

No response object is necessary.

### HTTP Response Codes

Code | Meaning
------------------ | -------
`200 OK` | Your API successfully cleaned up the data.
`401 Unauthorized` | Your API was provided an incorrect `Authorization` header.
`404 Not Found` | Your API could not find a session with the given session ID.
