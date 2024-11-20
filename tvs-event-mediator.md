---

## Handlers Overview

### Mediator

The `Mediator` acts as the core communication hub. It registers handlers and notifies or listens for events.

### MQTTHandler

Handles MQTT-based communication. This example shows how to subscribe to an MQTT topic and receive updates.

### PluginHandler

Enables interactions with plugins, including synchronous calls and callbacks.

---

## Example Usage

### 1. **Initialize Mediator and Handlers**

Set up the `Mediator`, `PluginHandler`, and `MQTTHandler`.

```typescript
import { Mediator } from "./mediator";
import { MQTTHandler, PluginHandler } from "./handlers";

const mediator = new Mediator();
const pluginHandler = new PluginHandler(mediator);
export const mqttHandler = new MQTTHandler(mediator, {
    brokerUrl: "mqtt://localhost:1883",
    clientId: "client-" + Math.random().toString(16).substr(2, 8),
    cypherCredentials: {
        publicKey: "-----BEGIN ... -----END",
        privateKey: "-----BEGIN ... -----END",
    },
});

mediator.registerHandler("mqtt", mqttHandler);
mediator.registerHandler("plugin", pluginHandler);
mqttHandler.connect('username', 'password');
```

### 2. **Subscribe to MQTT Topic**

Subscribe to an MQTT topic and listen for updates.

```typescript

mediator.notify("rs/control-center/events", {
    target: "mqtt",
    meta: { action: "subscribe", params: { qos: 1 } },
});

mediator.listen("rs/control-center/events", (data) => {
    console.log("GPS Location update received from mqtt:", data);
});

```

### 3. **Publish to MQTT Topic**

Publish a message to an MQTT topic.
Set `isEncrypted` to `true` to encrypt the message.

```typescript
mediator.notify("rs/control-center/events", {
    target: "mqtt",
    meta: { action: "publish", params: { message: "Hello, MQTT!" } },
    isEncrypted: false,
});
```

### 4. **Call Plugin**

Call a plugin and receive a response.

```typescript
const deviceId = await mediator.notify("deviceInfo", {
    target: "plugin",
    meta: {
        pluginName: "Device",
        methodName: "getId",
    },
});
console.log("Device ID:", deviceId);
```

### 5. **Handle Plugin Callback**

Handle a plugin callback, listenResponse must be set to true to listen for plugin updates.

```typescript
const callbackId = await mediator.notify("watchPosition", {
    target: "plugin",
    meta: {
        pluginName: "Geolocation",
        methodName: "watchPosition",
    },
    listenResponse: true,
});

mediator.listen("geolocation", (data) => {
    console.log("Geolocation update received:", data);
});
```
