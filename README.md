# Home Automation System Specification

## Layers
MQTT Protocol, two messaging types:
### Publish/subscription based (pub-sub)
Messages *to* this type of topics (e.g. `sensors\temperature\in`) are treated as strings, but can be anything (int, object)
Responses are given on the corresponding topic (e.g. `sensors\temperature\out`).

### Request/response based (req-res)
Messages *to* this type of topic will *always* be JSON Objects, with the following structure:
```js
{
  "replyTo": <String>,
  "requestId": <String>,
  "body": <Object|null>
}
```
Responses will be send on the `replyTo` topic specified in the request, and will have the following structure:
```js
{
  requestId: <String>,
  payload: <Object|null>
}
```


### Agents
#### Temperature sensors
Detects the temperature in an area
Outputs time of reading, temperature and humidity

Listen on `sensors/temperature/in`  
Respond on `sensors/temperature/out` with:
```js
{
  "id": <String>,
  "timestamp": <ISODate>,
  "temperature": <Float>,
  "humidity": <Float>,
}
```

#### Binary heating control (on/off)
Toggles heating (relay, switch) on/off  
Listen on `switches/heating/in` with: `on`, `off` or `state`  
Respond on `switches/heating/out` with: `1` or `0` (on any valid message received)


#### Thermostat API
Listens on `thermos/#`  
On state change, toggles heating if needed.  
Force update when state is outdated.  
Use `MongoDB` for storage. 

##### Target temperature(pub-sub)
`thermos/temperature/in` => `thermos/temperature/out`  
Response: `<float>`

#### Status
`thermos/status/in` => `thermos/status/out`  
Response:
```js
{
  "program": <String>,
  "target": <float>,
  "sensors": [{
      "id": <String>, //sensor id
      "temperature": <float>,
      "humidity": <float>,
      "timestamp": <ISODate>
    }, ...
  ],
  "overrides": [ //active or upcoming overrides
    {
      "id": <int>,
      "from": <ISODate>,
      "to": <ISODate>,
      "interval": <int>
    }
  ],
  "heating": {
    "value": <bool>,
    "timestamp": <ISODate>
  }
}
```

##### Override(req-res)
`thermos/override` => replyTo  
Override thermostat:
 - target => heating on until target temperature achieved
 - duration => heating on for given duration
 - target + duration => heating on until target or duration
`start` = activation time for override
Request:
```js
{
  "target": <float>, // temperature
  "duration": <int>, // seconds
  "start": <ISODate>, // optional, default 'now'
  // !At least target or duration must be present
}
```
Response:
```js
{
  "id": <int> //id of stored override
}
```

##### History(req-res)
`thermos/history` => replyTo  
Query with:
```js
{
  "from": <ISODate>,
  "to": <ISODate>,
  "interval": <int>, // seconds, default 600 = 10 minutes
  "sensors": <String>[] // sensor ids; optional (default all)
}
```
Respond with:
```js
{
  "meta": {
    "from": <ISODate>,
    "to": <ISODate>,
    "interval": <int>
  },
  "data": [
    {
      "target": <float>,
      "program": <String>,
      "sensors": [
        {
          "id": <String>,
          "temperature": <float>,
          "humidity": <float>
        }
      ],
      "heating": <bool>,
      "timestamp": <ISODate>
    }, ...
  ]
}
```

### Logger
Listens on `log/#` (`log/error`, `log/warn`, `log/debug`)
Other agents can publish logs to those topics.  
The logger can store them or take actions (e.g. send email) based on the received messages.
```js
{
  "agent": <string>, //agent name & id
  "timestamp": <ISODate>,
  "message": <String>,
  "data": <Object>, //optional
}
```

Listens on `logger/in`  
Outputs on `logger/out`  
TODO


