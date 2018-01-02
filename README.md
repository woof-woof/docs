# Home Automation System Specification

## Layers

### Low level layer

MQTT Protocol

#### Agents
##### Temperature sensors
Detects the temperature in an area
Outputs time of reading, temperature and humidity

Listen on `sensors/temperature/in`  
Respond on `sensors/temperature/out`
```
{
  "id": "<String>",
  "timestamp": "<ISODate>",
  "temperature": "<Float>",
  "humidity": "<Float>",
}
```


Binary heating control (on/off because Ciceu does not understand fancy words)





