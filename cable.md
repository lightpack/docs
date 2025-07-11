# Lightpack Cable – Real-Time Communication for Everyone

**Cable** is Lightpack’s **HTTP Polling** based soft real-time communication layer. With **Cable**, you can build live dashboards, notifications, presence channels, chat messages and more. It is a great fit for your relatime notifications where the complexities of `websockets` or `SSE` is not justified.

It support `MySQL` and `Redis` based backends for enabling realtime support and ships with a **JavaScript** client `cable.js` to make it painless to build on it. Under the hood, here’s how it works:

1. **Event Emission (PHP Side):**
    - Your PHP code emits events to a channel using Cable’s API (e.g., `$cable->to('chat:42')->emit('message:new', [...])`).
    - These events are stored by a driver—either in a database table or Redis sorted set, depending on your configuration.

2. **Polling (Browser Side):**
    - The browser uses `cable.js` to subscribe to one or more channels.
    - Instead of keeping a persistent WebSocket connection, the client periodically (e.g., every 3 seconds) polls a lightweight HTTP endpoint (e.g., `/cable/poll`) to fetch new events for its channels.
    - Each poll includes the last received message ID, so only new events are delivered.

3. **Event Delivery:**
    - The backend responds with all new events since the last poll.
    - The client triggers your JavaScript handlers for each event (e.g., updating the UI, playing a sound, etc.).

4. **Presence & Batching:**
    - Presence channels track which users are online, using the same driver-based approach.
    - Message batching reduces backend writes by grouping many events into a single batch.

## Cable Event Flow (Lifecycle)

```
→ PHP emits event 
→ Driver stores event 
→ Browser polls 
→ New events delivered 
→ JS handler runs
```

---

## Core Concepts

- **Channel:** A logical stream (e.g., `chat:42`, `dashboard`, `presence:room1`).
- **Event:** A named action (e.g., `message:new`, `dom-update`, `presence:update`).
- **Payload:** Arbitrary data sent with each event.
- **Driver:** Backend for message storage/delivery (Database, Redis, custom).
- **Presence:** Tracks online users in a channel.
- **Batching:** Efficiently emits many events at once.

---

## Quick Start

### 1. Server-Side Setup

**Choose a driver:**
   - Database (default, simple, persistent)
   - Redis (high-performance, temporary)
    
You can view available configurations in `config/cable.php` file.

### 2. Emitting Events

In your controller's method:

```php
// Send a message to the 'chat:42' channel
$cable->to('chat:42')->emit('message:new', [
    'user' => 'alice',
    'text' => 'Hello, world!'
]);
```

### 3. Receiving Events in the Browser

Include the client:

```html
<?= asset()->load('js/cable.js') ?>
```

Subscribe and handle events:

```js
cable.connect().subscribe('chat:42', {
    'message:new': (payload) => {
        console.log(payload.user + ': ' + payload.text);
    },
    'dom-update': (payload) => {
        document.querySelector(payload.selector).innerHTML = payload.html;
    }
});
```

---

## Cable API Reference

### Cable Class & Methods

#### 1. `to(string $channel): self`
Targets a specific channel for subsequent events.

```php
// Targeting a chat channel
$chat = $cable->to('chat:42');
```

#### 2. `emit(string $event, array $payload = [])`
Sends an event (with payload) to the current channel.

```php
// Emitting a new chat message
$cable->to('chat:42')->emit('message:new', [
    'user' => 'alice',
    'text' => 'Hello, world!'
]);
```

#### 3. `update(string $selector, string $html)`
Emits a `dom-update` event, allowing you to update parts of the UI in real time.

```php
// Live update a status element on all dashboards
$cable->to('dashboard')->update('#status', '<span>Online</span>');
```

#### 4. `getMessages(?string $channel = null, ?int $lastId = null): array`
Retrieves new messages for a channel since a given message ID. Used by polling clients (normally not called directly).

```php
// Fetch new messages for chat:42 since message ID 100
$messages = $cable->getMessages('chat:42', 100);
foreach ($messages as $msg) {
    echo $msg->event . ': ' . json_encode($msg->payload);
}
```

#### 5. `cleanup(int $olderThanSeconds = 86400)`
Deletes old messages from the backend (maintenance/housekeeping).

```php
// Remove messages older than 1 hour
$cable->cleanup(3600);
```

#### 6. `getDriver(): CableDriverInterface`
Returns the current driver instance (for advanced usage).

```php
$driver = $cable->getDriver();
```

---

### MessageBatcher
Efficiently emits many events as a single batch (reduces DB/Redis writes):

```php
use Lightpack\Cable\MessageBatcher;

// Create a batcher for the 'metrics' channel, batching up to 100 events
$batcher = new MessageBatcher($cable, 'metrics', 100);

// Add events to the batch
$batcher->add('metric:update', ['cpu' => 42]);
$batcher->add('metric:update', ['cpu' => 43]);

// Manually flush the batch (or let it auto-flush on shutdown)
$batcher->flush();
```

---

## Drivers Explained

### CableDriverInterface
Any driver must implement:
- `emit($channel, $event, $payload)`
- `getMessages($channel, $lastId)`
- `cleanup($olderThanSeconds)`

### Database Driver
- Stores messages in a table (default: `cable_messages`).
- Good for small/medium scale, persistent history, easy setup.
- Example schema:
  ```sql
  CREATE TABLE cable_messages (
      id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
      channel VARCHAR(255) NOT NULL,
      event VARCHAR(255) NOT NULL,
      payload TEXT,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  );
  ```
- Cleanup old messages regularly.

### Redis Driver
- Stores messages in Redis sorted sets (keyed by channel).
- Ultra-fast, ephemeral (auto-expires), scales for high-traffic.
- Use for chat, dashboards, presence at scale.
- Set up Redis and register the driver:
  ```php
  use Lightpack\Cable\Drivers\RedisCableDriver;
  $driver = new RedisCableDriver($redis);
  $cable = new Cable($driver);
  ```
- Automatic expiry and cleanup.

### Custom Drivers
Implement `CableDriverInterface` for any backend (e.g., Kafka, SQS, WebSockets).

---

## Presence Channels

Presence lets you track which users are online in a channel—essential for collaborative apps, group chat, and live dashboards.

### Using Presence
```php
use Lightpack\Cable\Presence;
use Lightpack\Cable\Drivers\DatabasePresenceDriver;
$presenceDriver = new DatabasePresenceDriver($db);
$presence = new Presence($cable, $presenceDriver);

// User joins a channel
$presence->join($userId, 'room1');

// User leaves
$presence->leave($userId, 'room1');

// Heartbeat to stay online
$presence->heartbeat($userId, 'room1');

// Get users present
$users = $presence->getUsers('room1');
```

### PresenceDriverInterface
Implements presence tracking for users in channels. Example usage:

```php
use Lightpack\Cable\Presence;
use Lightpack\Cable\Drivers\DatabasePresenceDriver;

$presenceDriver = new DatabasePresenceDriver($db);
$presence = new Presence($cable, $presenceDriver);

// User joins a channel
$presence->join($userId, 'room1');

// User leaves
$presence->leave($userId, 'room1');

// Heartbeat to keep user present
$presence->heartbeat($userId, 'room1');

// Get users currently present
$users = $presence->getUsers('room1');

// Get all channels a user is present in
$channels = $presence->getChannels($userId);

// Clean up stale presence records
$presence->cleanup();
```

#### DatabasePresenceDriver
- Tracks users in a table (default: `cable_presence`).
- Uses upsert for join/heartbeat.
- Cleans up stale sessions.

#### RedisPresenceDriver
- Tracks users in Redis sets (per channel/user).
- Ultra-fast, auto-expires.

---

## Batching Events

Batching reduces backend writes and network load by grouping many events into one message.

- Use `MessageBatcher` when emitting many events rapidly (e.g., analytics, metrics).
- Auto-flushes on shutdown, or call `flush()` manually.
- Client receives a `batch` event with all grouped events.

---

## Client-Side Integration: cable.js

Cable’s JavaScript client brings real-time to your browser with a Socket.io-like API, but uses efficient polling for broad compatibility and simplicity.

### Features
- Subscribe to channels and events
- Event filtering
- Batched event support
- DOM updates
- Notification sounds
- Robust reconnect and polling logic

### Example Usage
```js
cable.connect().subscribe('chat:42', {
    'message:new': (payload) => {
        // Handle new message
    },
    'dom-update': (payload) => {
        // Live update UI
    }
});

// Advanced: filter events
cable.subscribe('metrics').filter(payload => payload.cpu > 80).on('metric:update', payload => {
    alert('High CPU!');
});
```

### DOM Updates
Cable supports emitting `dom-update` events for live UI changes:
```php
$cable->to('dashboard')->update('#status', '<span>Online</span>');
```
Client-side:
```js
document.querySelector(payload.selector).innerHTML = payload.html;
```

### Notification Sounds
Cable.js can play sounds for events:
```js
cable.playSound('/sounds/notify.mp3');
```

---

## Best Practices & Advanced Usage

- **Security:** Use unique, non-guessable channel names for private data. Implement authentication/authorization in your app logic.
- **Cleanup:** Regularly call `cleanup()` on your driver to remove old messages/presence records.
- **Scaling:** For high-traffic, prefer Redis drivers, tune batch sizes, shard channels if needed.
- **Testing:** All Cable components are fully testable—use mocks for drivers, test event emission and presence logic.
- **Custom Drivers:** Implement the relevant interface for any backend; see source/tests for examples.
- **Client Polling:** Tune polling intervals for your use case (default: 3s).

---

## Troubleshooting & FAQ

- **Events not received?** Check channel names, driver config, and polling intervals.
- **High DB/Redis load?** Use batching, increase cleanup frequency, consider sharding.
- **Presence not updating?** Ensure heartbeats are sent regularly, check timeout settings.
- **Need push (WebSockets) support?** Implement a custom driver and JS client if needed—Cable is designed for extensibility.

---

## Extending Cable

- **Write your own driver:** Implement `CableDriverInterface` or `PresenceDriverInterface`.
- **Integrate with frameworks:** Cable is framework-agnostic—use in any PHP app, or integrate with React/Vue/Angular on the front-end.
- **Real-world Example:**
    - Collaborative whiteboard: use presence to track users, emit drawing events to a channel, update canvases in real time.
    - Live dashboard: batch metric updates, push to all dashboards instantly.

---

## Appendix

### Database Schema: cable_messages
```sql
CREATE TABLE cable_messages (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    channel VARCHAR(255) NOT NULL,
    event VARCHAR(255) NOT NULL,
    payload TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Database Schema: cable_presence
```sql
CREATE TABLE cable_presence (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    channel VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX (channel),
    INDEX (user_id),
    UNIQUE KEY channel_user (channel, user_id)
);
```

### Test Coverage
Cable and all drivers are comprehensively tested—see `tests/Cable/` for examples covering every method, edge case, and integration scenario.

---

## Case Study: Real-Time Chat Room with Presence

Let’s build a complete, minimal real-time chat room with user presence, using Lightpack Cable. This shows all major concepts in action, from backend to frontend.

### 1. Backend Setup

#### a) Register Providers
In your `App.php`, register the Cable provider (and Redis if using Redis drivers):
```php
protected function getFrameworkProviders(): array {
    return [
        // ...
        \Lightpack\Framework\Providers\RedisProvider::class, // If using Redis
        \Lightpack\Framework\Providers\CableProvider::class,
        // ...
    ];
}
```

#### b) Run Migrations
Create the required tables for messages and presence (if using the DB driver):
```sql
CREATE TABLE cable_messages (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    channel VARCHAR(255) NOT NULL,
    event VARCHAR(255) NOT NULL,
    payload TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE cable_presence (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    channel VARCHAR(255) NOT NULL,
    user_id VARCHAR(255) NOT NULL,
    last_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX (channel),
    INDEX (user_id),
    UNIQUE KEY channel_user (channel, user_id)
);
```

#### c) Configure Cable
Choose and instantiate your driver in a service or controller:
```php
use Lightpack\Cable\Cable;
use Lightpack\Cable\Drivers\DatabaseCableDriver;
use Lightpack\Cable\Presence;
use Lightpack\Cable\Drivers\DatabasePresenceDriver;

$cable = new Cable(new DatabaseCableDriver($db));
$presence = new Presence($cable, new DatabasePresenceDriver($db));
```

### 2. PHP: Emitting Chat Messages and Presence

**a) Sending a Chat Message:**
```php
// Called when a user sends a message
$cable->to('chat:main')->emit('message:new', [
    'user' => $user['name'],
    'text' => $input['text'],
    'timestamp' => time(),
]);
```

**b) Managing Presence:**
```php
// User joins the chat room
$presence->join($user['id'], 'chat:main');

// User leaves (e.g., on logout or disconnect)
$presence->leave($user['id'], 'chat:main');

// Heartbeat (call every minute to keep user online)
$presence->heartbeat($user['id'], 'chat:main');
```

### 3. Polling Endpoint (Controller Example)

Expose a polling endpoint for the JS client:
```php
// routes/web.php
$router->post('/cable/poll', function() use ($cable) {
    $channels = $_POST['channels'] ?? [];
    $lastIds = $_POST['lastIds'] ?? [];
    $result = [];
    foreach ($channels as $channel) {
        $result[$channel] = $cable->getMessages($channel, $lastIds[$channel] ?? null);
    }
    echo json_encode($result);
});
```

### 4. Frontend: HTML + Cable.js

**a) Include Cable.js**
```html
<script src="/assets/cable.js"></script>
```

**b) HTML Structure**
```html
<div id="chat">
    <div id="messages"></div>
    <input id="chat-input" type="text" placeholder="Type a message...">
    <button id="send-btn">Send</button>
    <div id="presence"></div>
</div>
```

**c) JavaScript: Subscribing and Handling Events**
```js
const username = prompt('Enter your name:');
const userId = Math.floor(Math.random() * 100000); // Example user ID

// Join presence (AJAX call to backend on page load)
fetch('/join', {
    method: 'POST',
    body: JSON.stringify({ userId, channel: 'chat:main' }),
    headers: { 'Content-Type': 'application/json' }
});

// Cable client
cable.connect().subscribe('chat:main', {
    'message:new': (payload) => {
        const msg = document.createElement('div');
        msg.textContent = `${payload.user}: ${payload.text}`;
        document.getElementById('messages').appendChild(msg);
    },
    'presence:update': (payload) => {
        document.getElementById('presence').textContent =
            `Online: ${payload.count} users`;
    }
});

document.getElementById('send-btn').onclick = function() {
    const text = document.getElementById('chat-input').value;
    // Send message to backend (AJAX call that emits via Cable)
    fetch('/send', {
        method: 'POST',
        body: JSON.stringify({ userId, user: username, text }),
        headers: { 'Content-Type': 'application/json' }
    });
    document.getElementById('chat-input').value = '';
};
```

### 5. Presence and DOM Updates

Whenever presence changes (user joins/leaves/heartbeat), the backend emits a `presence:update` event:
```php
// In Presence class (already built-in)
$cable->to('chat:main')->emit('presence:update', [
    'users' => $users, // List of user IDs or names
    'count' => count($users),
    'timestamp' => time(),
]);
```
The frontend updates the online user count automatically.

### 6. Cleanup & Best Practices
- Call `$cable->cleanup()` and `$presence->cleanup()` periodically (e.g., via cron) to remove old messages and stale presence.
- Use secure, non-guessable channel names for private rooms.
- Add authentication and authorization as needed for your app.

---

**Lightpack Cable is your foundation for building real-time, collaborative, and interactive PHP applications—without the complexity.**

> "Real-time isn’t magic. With Lightpack Cable, it’s just another tool in your kit—simple, explicit, and yours to command."