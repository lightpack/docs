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

Lightpack already ships with few route definitions for making it easy to work with message polling or presence channels. You can look for `routes/cable.php` file for more details. Following sections gives you a quick overview to get started with realtime features.

### 1. Configure Cable

You can view available configurations in `config/cable.php` file.

**Choose a driver:**
   - Database (default, simple, persistent storage)
   - Redis (high-performance, temporary storage)

### 2. Run Migration

Run the following migration command if using MySQL as realtime backend:

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

// Subscribe to `message:new` event on notifications channel 
socket
    .subscribe('notifications')
    .on('message:new', function(data) {
        console.log('New message:', data.text);
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

Your frontend client can subscribe to emitted events without requiring to manually poll for new messages:

```javascript
const subscription = socket.subscribe('another-channel');

subscription.on('event-one', function(data) {
    console.log('Event one:', data);
});

subscription.on('event-two', function(data) {
    console.log('Event two:', data);
});
```

### Custom Configurations

You can customize following option when initializing a **cable** connection.

```javascript
const socket = cable.connect(
    endpoint: '/cable/poll',
    pollInterval: 3000,
    reconnectInterval: 5000,
    maxReconnectAttempts: 5
)
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

Presence channels allow you to easily track which users are online in real-time.

**Note**: You must define the following meta tag inside <head> tag. This is automatically used by the `cable.js` to pass CSRF token in request header for presence endpoints.

```html
<meta name="csrf-token" content="<?= csrf_token() ?>">
```

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


### Presence Channel Configuration

```javascript
// Join a presence channel with custom endpoint
presence.join('/custom/join/endpoint');

// Leave a presence channel with custom endpoint
presence.leave('/custom/leave/endpoint);

// Start heartbeat with custom interval and endpoint
presence.startHeartbeat(30000, '/custom/heartbeat/endpoint');

// Get users with custom endpoint
presence.getUsers('/custom/users/endpoint');
```

## Message Batching

You can batch multiple messages together to promote better performance:

1. Fewer database writes
2. Better transaction handling
3. Reduced server load
4. More efficient client updates

### Server-Side Batching

**Without Batching (BAD)**

```php
public function notifyUsers(Cable $cable)
{
    // Sending 100 notifications = 100 database writes
    foreach($users as $user) {

        $cable->to('notifications')->emit('message:new', [
            'user' => $user->id,
            'message' => 'Hello'
        ]);

    }
}
```

**With Batching (GOOD)**

```php
public function notifyUsers(MessageBatcher $batcher)
{
    $batcher->channel('notifications')->batchSize(100);

    // Sending 100 notifications = 1 database writes
    foreach($users as $user) {

        $batcher->add('message:new', [
            'user' => $user->id,
            'message' => 'Hello'
        ]);

    }
}
```

> The default batch size is 100 which you can override. You never need to manually flush the batch of messages. it is automatically done by the **MessageBatcher** instance.

### Client-Side Batching

```javascript
const socket = cable.connect();
const channel = 'user-tracking';

// Configure client-side batching
socket.setBatchOptions({
    batchSize: 5,           // Max events per batch
    batchInterval: 2000,     // Flush interval in ms
    batchEndpoint: '<?= url()->route('api.batch-events') ?>',
    csrfToken: '<?= csrfToken() ?>',
});

// Track events (added to batch)
socket.emitBatched(channel, 'event:button-clicked', { data: 'Subscribe Button' });
socket.emitBatched(channel, 'event:mouse-moved', { data: 'FAQ Section' });
socket.emitBatched(channel, 'event:button-hovered', { data: 'Buy Now Button' });
...
...
...
socket.emitBatched(channel, 'event:button-clicked', { data: 'Buy Now Button' });
```

> Once the batch size exceeds or batch interval passes, the batch endpoint is automatically called with all the even data.

If required, you can manually flush the batch of events:

```javascript
socket.flushOutgoingBatch();
```

## Notification Sounds
An interesting feature of Lightpack `cable.js` client is that it can play sounds for subcribed events:

```javascript
cable.playSound('/sounds/notify.mp3');
```


**For example:**

- You can put the **notify.mp3** file in `public/sounds` folder.
- Optionally, set the volume by passing a value between 0 to 1.

```javascript
socket
    .subscribe('notifications')
    .on('message', function(data) {
        cable.playSound('/sounds/notify.mp3', 0.7);
    });
```

```