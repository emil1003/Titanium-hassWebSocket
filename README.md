# Titanium-hassWebSocket
[Home Assistant WebSocket](https://developers.home-assistant.io/docs/api/websocket/)
implementation for ComputerCraft applications written with
[Titanium](https://gitlab.com/hbomb79/Titanium) and [CC:Tweaked](https://tweaked.cc).

Automatically takes care of tracking connection, requests, subscriptions, command ID and heartbeat,
leaving a lean API for your application.

## Setup
Include the contents of `src` as class source in `.tdt_projects.cfg`:
```lua
class_sources = {
    [ "Titanium-hassWebSocket/src" ] = true,
}
```

## Usage
Create WebSocket instance:

```lua
webSocket = WebSocket("wss://<Home Assisant address>/api/websocket", "<API Token>")

-- Connect WebSocket to your Application for events
webSocket:hook(Application)

-- Initialize connection
webSocket:connect()
```

WebSocket uses callbacks to communicate state to your application:

```lua
-- Run function when WebSocket connection status changes
webSocket:on(WebSocket.CALLBACK_CONNECTION_CHANGED, function(self, isOpen)
    print("Connection was " .. isOpen and "opened" or "closed")
end)

-- Run function when authentication is requested (if API Token not set)
webSocket:on(WebSocket.CALLBACK_AUTH_REQUESTED, function()
    webSocket:authenticate("<API Token>") -- Manually authenticate
end)

-- Run function when successfully authenticated on server
webSocket:on(WebSocket.CALLBACK_AUTHENTICATED, function()
    print("Authenticated") -- Only when authenticated can messages be sent
end)
```

Send a request:

```lua
-- Create a Ping request
request = WebSocketRequest(WebSocketRequest.PING)

-- Listen for result in callback
request:on(WebSocketRequest.CALLBACK_RESULT, function(self, result)
    print(result.type) -- Pong
end)

-- Send request (must be authenticated)
webSocket:request(request)

-- WebSocketServiceCall makes a "call_service" request easy
scRequest = WebSocketServiceCall(
        Domains.MEDIA_PLAYER,
        Services.media_player.MEDIA_NEXT_TRACK,
        "media_player.example")
```

Subscribe to serverside events:

```lua
-- Subscribe to "state_changed" events
subscription = WebSocketSubscription(WebSocketSubscription.EVENT_STATE_CHANGED)

-- Filter events based on 'entity_id'
-- Here, only allow events from 'light.living_room' entity and all in 'media_player' domain
subscription:filter("light.living_room", "media_player.")

-- Get events in callback
subscription:on(WebSocketSubscription.CALLBACK_EVENT, function(self, event)
    print(event:getEventData()["entity_id"]) -- entity_id of the entity that changed
end)

-- Request subscription (must be authenticated)
webSocket:subscribe(subscription)
```
