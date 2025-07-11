# Cable: Real-Time Communication for Lightpack

**Cable** is Lightpack's elegant solution for real-time communication between your server and clients. With a Socket.io-like API and a focus on simplicity and efficiency, **Cable** provides powerful real-time features without external dependencies.

You can build live dashboards, notifications, presence channels, chat messages and more with minimal friction. It uses an efficient `HTTP Polling` mechanism to provide real-time communication capabilities:

- **Socket.io-like API**: Familiar, event-based programming model
- **Channel-based messaging**: Target specific users or groups
- **Presence channels**: Track which users are online
- **DOM updates**: Directly update page elements
- **Driver architecture**: Support for database and Redis backends

Unlike WebSockets, Cable works with any hosting environment and doesn't require special server configurations.

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

### 1. Run Migration

Run the following migration command:

```cli
php console create:migration create_table_cable_system
```

Update following in your `up()/down()` migration methods:

```php
public function up(): void
{
    // table: cable_messages
    $this->create('cable_messages', function(Table $table) {
        $table->id();
        $table->varchar('channel', 255);
        $table->varchar('event', 255);
        $table->column('payload')->type('json')->nullable();
        $table->datetime('created_at')->nullable();
        
        $table->index('channel');
    }); 

    // table: cable_presence
    $this->create('cable_presence', function(Table $table) {
        $table->id();
        $table->varchar('channel', 255);
        $table->column('user_id')->type('bigint')->attribute('unsigned');
        $table->datetime('last_seen');

        $table->foreignKey('user_id')->references('id')->on('users')->cascadeOnDelete()->cascadeOnUpdate();
        
        $table->unique(['channel', 'user_id']);
        $table->index('channel');
        $table->index('last_seen');
    });
}

public function down(): void
{
    $this->drop('cable_messages');
    $this->drop('cable_presence');
}
```

### 2. Configure Cable

You can view available configurations in `config/cable.php` file.

**Choose a driver:**
   - Database (default, simple, persistent storage)
   - Redis (high-performance, temporary storage)

### 3. Define Route

```php
route()->post('/realtime/send-message', RealtimeController::class, 'triggerMessage');
```

This route will forward the `POST /realtime/send-message` request to **RealtimeController**'s triggerMessage() method.

### 4. Emit Events

In your controller's method, emit an event `message:new` to the `notifications` channel with required payload:

```php
class RealtimeController
{
    public function triggerMessage(Cable $cable)
    {
        $cable->to('notifications')->emit('message:new', [
            'text' => request()->input('message'),
        ]);
    
        return response()->json(['success' => true]);
    }
}
```

### 5. Receive Events

In your frontend, include the `cable.js` script file:

```html
<?= asset()->load('js/cable.js') ?>
```

Initialize `cable` client and subscribe to `notifications` channel events:

```javascript
// Connect to Cable
const socket = cable.connect();

// Subscribe to a channel
socket.subscribe('notifications', {

    // Handle events and payload data
    'message:new': function(data) {
        console.log('New message:', data.text);
    }

});
```

## Cable API Reference

### Cable Class & Methods

#### `to(string $channel): self`
Targets a specific channel for subsequent events.

```php
// Targeting a chat channel
$chat = $cable->to('chat:42');
```

#### `emit(string $event, array $payload = [])`
Sends an event (with payload) to the current channel.

```php
// Emitting a new chat message
$cable->to('chat:42')->emit('message:new', [
    'user' => 'alice',
    'text' => 'Hello, world!'
]);
```

#### `update(string $selector, string $html)`

Emits a `dom-update` event, allowing you to update parts of the UI in real time.

```php
// Live update a status element on all dashboards
$cable->to('dashboard')->update('#status', '<span>Online</span>');
```

#### `getMessages(?string $channel = null, ?int $lastId = null): array`
Retrieves new messages for a channel since a given message ID. Used by polling clients (normally not called directly).

```php
// Fetch new messages for chat:42 since message ID 100
$messages = $cable->getMessages('chat:42', 100);
foreach ($messages as $msg) {
    echo $msg->event . ': ' . json_encode($msg->payload);
}
```

#### `cleanup(int $olderThanSeconds = 86400)`
Deletes old messages from the backend (maintenance/housekeeping).

```php
// Remove messages older than 1 hour
$cable->cleanup(3600);
```

> You can run a schedule to routinely cleanup old messages.


## Channel-Based Communication

Channels allow you to organize your real-time communication. Your application emits events on a channel, whereas your frontend client subscribes to events on a channel.

### Emitting Events

```php
// Send to a specific user
$cable->to("user.{$userId}")->emit('private-message', [
    'text' => 'This is a private message'
]);
```
```php
// Send to a group
$cable->to('admin-notifications')->emit('system-alert', [
    'level' => 'warning',
    'message' => 'Disk space is low'
]);
```

```php
// Broadcast to everyone
$cable->to('broadcasts.all')->emit('announcement', [
    'message' => 'Site maintenance in 10 minutes'
]);
```

### Subscribing to Events

Your frontend client can subscribe to emitted events:

```javascript
socket.subscribe('my-channel', {
    // Single event handler
    'event-name': function(data) {
        console.log('Event received:', data);
    }
});
```

Or add handlers later:

```javascript
const subscription = socket.subscribe('another-channel');

subscription.on('event-one', function(data) {
    console.log('Event one:', data);
});

subscription.on('event-two', function(data) {
    console.log('Event two:', data);
});
```

## DOM Updates

Cable can directly update DOM elements without writing custom JavaScript:

```php
// Update a specific element by selector
$cable->to('dashboard')->update(
    '#user-count', 
    "<strong>{$userCount}</strong> users online"
);
```

```php
// Update multiple elements with the same selector
$cable->to('dashboard')->update(
    '.status-indicator', 
    '<span class="online"></span>'
);
```

> On the client-side, this is handled automatically - no additional code needed!

## Presence Channels

Presence channels allow you to track which users are online in real-time.

**Initialize the channel presence**

```javascript
// Define channel name
const channel = 'presence-room';

// Initialize Cable
const socket = cable.connect();

// Initialize presence channel
const presence = socket.presence(channel, userId);
```

Now you can utilize the presence channel capabilities as documented below:

### Join the presence channel

```javascript
presence.join()
    .then(() => {
        console.log('Joined presence channel');
        
        // Start heartbeat to maintain presence
        presence.startHeartbeat();
    });
```

### Leave presence channel

```javascript
presence.stopHeartbeat();
presence.leave()
    .then(() => {
        console.log('Left presence channel');
    });
```

### Get all users in channel

```javascript
presence.getUsers()
    .then(data => {
        console.log('Users in channel:', data.users);
    });
```

### Subscribe to presence channel events

```javascript
socket.subscribe(channel)

    // Handle presence updates
    .on('presence:update', function(data) {
        console.log('Presence update:', data);
        // updateOnlineUsers(data.users);
    })

    // Handle join events
    .on('presence:join', function(data) {
        console.log('Users joined:', data.users);
        data.users.forEach(id => {
            // addNotification(`User ${id} joined the chat`);
        });
    })

    // Handle leave events
    .on('presence:leave', function(data) {
        console.log('Users left:', data.users);
        data.users.forEach(id => {
            // addNotification(`User ${id} left the chat`);
        });
    })

    // Handle message events
    .on('message', function(data) {
        console.log('Received message event with data:', data);
        // addMessage(data.userId, data.text);
    });
```

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