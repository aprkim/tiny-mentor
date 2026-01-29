# MiniChatCore Developer Guide v0.26 January 28, 2026 3:56pm

**Add video chat to your app with minimal JavaScript**

---

## What is MiniChatCore?

MiniChatCore is a high-level wrapper that handles all the complexity of WebRTC video chat. You create one `MiniChatCore` object, set up event handlers, and call simple methods. The library manages:

- User authentication (existing accounts or anonymous guests)
- Channel/room creation and joining
- Camera, microphone, and screen sharing
- Real-time member presence and media state synchronization

**You write UI code. MiniChatCore handles the video infrastructure.**

---

## Core Concepts

### 1. Event-Driven Architecture

MiniChatCore uses a callback pattern. You assign functions to event properties, and MiniChatCore calls them when things happen:

```javascript
const chat = new MiniChatCore(config);

chat.onLogin = (user) => { /* user logged in */ };
chat.onMemberJoined = (memberId) => { /* someone joined */ };
chat.onMemberStreamStart = (memberId, streamType) => { /* video stream arrived */ };
```

This keeps your code reactiveâ€”you respond to events rather than polling for state.

### 2. Three Entry Paths

Users can enter a video chat session three ways:

| Path | Use Case | Methods |
|------|----------|---------|
| **Login** | Registered users with existing channels | `login()` â†’ `loadChannels()` â†’ `selectChannel()` |
| **Create Room** | Anonymous user starts a new room | `signupAnonymous()` â†’ `createChannel()` â†’ `joinByChannelId()` |
| **Join Room** | Anonymous user joins via room code | `signupAnonymous()` â†’ `joinByRoomCode()` |

The room code (a short string like `"K7xmQ"`) is shareableâ€”give it to others so they can join.

### 3. Video Element Management

MiniChatCore manages `<video>` element `srcObject` bindings automatically:

```javascript
// Local camera preview
chat.setLocalVideoElement(document.getElementById('myVideo'));

// Remote member's stream (call this when onMemberStreamStart fires)
chat.setMemberVideoElement(memberId, streamType, videoElement);
```

You create the `<video>` elements; MiniChatCore attaches the media streams.

---

## Quick Start Pattern

### Step 1: Setup

Include the import map and create the chat instance:

```html
<script type="importmap">
{
  "imports": {
    "minichat-core": "https://proto2.makedo.com:8883/v02/scripts/minichat-core.js",
    "config": "https://proto2.makedo.com:8883/v02/scripts/configServer.js"
  }
}
</script>

<script type="module">
import MiniChatCore from 'minichat-core';

const chat = new MiniChatCore({
    contextId: 'YOUR_CONTEXT_ID',
    contextAuthToken: 'YOUR_CONTEXT_TOKEN'
});
</script>
```

### Step 2: Wire Up Events

At minimum, handle these events:

```javascript
chat.onJoined = () => { /* you're live in the room */ };
chat.onLeft = () => { /* you left the room */ };

chat.onMemberStreamStart = (memberId, streamType) => {
    // Create a <video> element for this member
    const video = document.createElement('video');
    video.autoplay = true;
    document.body.appendChild(video);
    
    // MiniChatCore attaches the stream automatically
    chat.setMemberVideoElement(memberId, streamType, video);
};

chat.onMemberStreamEnd = (memberId, streamType) => {
    // Remove their video element
};
```

### Step 3: Join a Room

For the simplest anonymous flow:

```javascript
await chat.signupAnonymous('MyName');
await chat.joinByRoomCode('ROOM_CODE');  // Join by room code
// OR: await chat.joinByChannelId(channelId);  // Join by channel ID
await chat.join();  // Actually connect to WebRTC
```

### Step 3b: Leave or Exit

```javascript
chat.leave();        // Disconnect but keep camera/mic running (quick rejoin)
chat.exitChannel();  // Full teardown: stops all tracks, releases camera (light turns off)
```

Use `exitChannel()` when the user is done with the session. It directly stops all media tracks, ensuring the camera indicator light turns off immediately.

### Step 4: Control Media

```javascript
chat.toggleVideo();      // Start/stop camera
chat.toggleAudio();      // Start/stop microphone
chat.toggleScreencast(); // Start/stop screen share

chat.toggleMuteVideo();  // Hide video (keeps streaming, shows black)
chat.toggleMuteAudio();  // Mute mic (keeps streaming, sends silence)
```

---

## Key Events Reference

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| `onLogin` | `(user)` | After successful login/signup |
| `onChannelsLoaded` | `(channels[])` | After `loadChannels()` completes |
| `onUsersLoaded` | `(users[])` | After `loadUsers()` completes |
| `onChannelSelected` | `(channel)` | After `selectChannel()`, `selectUser()`, `joinByChannelId()`, or `joinByRoomCode()` |
| `onJoined` | none | Successfully connected to room |
| `onLeft` | none | Disconnected from room |
| `onMemberJoined` | `(memberId)` | New member entered the room |
| `onMemberLeft` | `(memberId)` | Member left the room |
| `onMemberStreamStart` | `(memberId, streamType)` | Member's video stream is available |
| `onMemberStreamEnd` | `(memberId, streamType)` | Member stopped streaming |
| `onMemberMediaChange` | `(memberId, streamType)` | Member muted/unmuted |
| `onLocalMediaChange` | none | Your own media state changed |
| `onError` | `(context, error)` | Something went wrong |

The `streamType` is either `"camera"` or `"screencast"`.

---

## Useful Getters

```javascript
chat.isLoggedIn        // boolean
chat.isInChannel       // boolean (joined WebRTC room)
chat.currentRoomCode   // string like "K7xmQ" for sharing
chat.localMediaState   // { audio, video, audioMuted, videoMuted }
chat.screencastState   // { video, videoMuted }

chat.getMember(memberId)           // { username, hasCamera, hasScreencast, ... }
chat.getMemberIds()                // array of member IDs in the room
chat.getMemberMediaStates(memberId) // detailed audio/video state

// Get channel info before joining (requires login first)
await chat.getChannelByRoomCode(roomCode)  // full channel: { id, room_code, title, description, members: [...], ... }
await chat.getChannelById(channelId)       // same, but by database ID
```

### Preview Room Before Joining

You can fetch full channel information before joining to check title, capacity, etc:

```javascript
await chat.signupAnonymous('Alice');

// Get room details before joining
const roomInfo = await chat.getChannelByRoomCode('K7xmQ');
console.log(`Room: ${roomInfo.title}`);
console.log(`Room code: ${roomInfo.room_code}`);

const memberCount = roomInfo.members ? roomInfo.members.length : 0;
if (memberCount >= 4) {
    alert('Room is full');
    return;
}

// Now join with confidence
await chat.joinByRoomCode('K7xmQ');
await chat.join();
```

---

## Working with Lists

### Channel List (for logged-in users)

Registered users may have multiple channels. After login, retrieve and display them:

```javascript
chat.onLogin = async (user) => {
    await chat.loadChannels();
};

chat.onChannelsLoaded = (channels) => {
    // channels = [{ id, name, description, room_code, ... }, ...]
    channels.forEach(ch => {
        dropdown.add(new Option(ch.name, ch.id));
    });
};
```

When the user selects a channel:

```javascript
dropdown.onchange = async () => {
    const channelId = dropdown.value;
    await chat.selectChannel(channelId);
    await chat.join();
};
```

**Key properties in each channel object:**
- `id` â€” unique channel identifier (use with `selectChannel()`)
- `name` (or `title`) â€” display name
- `room_code` â€” shareable room code for `joinByRoomCode()`
- `description` â€” optional channel description

### Creating a New Room

When you create a channel, the returned object contains the room code immediately:

```javascript
await chat.signupAnonymous('Alice');

const channel = await chat.createChannel({ 
    title: 'Alice\'s Room',
    allowsGuests: true 
});

// Room code is immediately available
console.log(`Share this code: ${channel.room_code}`);  // e.g., "K7xmQ"
console.log(`Channel ID: ${channel.id}`);        // Database ID
console.log(`Room title: ${channel.title}`);

// Join the room you just created
await chat.joinByRoomCode(channel.room_code);
await chat.join();

// Room code also available via getter after joining
console.log(chat.currentRoomCode);  // Same as channel.room_code
```

### Two Ways to Join a Channel

MiniChatCore provides two methods for joining channels:

**1. `joinByChannelId(channelId)` - The fundamental method**
```javascript
// Use when you have the channel database ID
const channel = await chat.createChannel({ title: 'My Room' });
await chat.joinByChannelId(channel.id);  // channel.id is the database ID
await chat.join();
```

**2. `joinByRoomCode(roomCode)` - Convenience wrapper**
```javascript
// Use when you have a shareable room code
await chat.joinByRoomCode('K7xmQ');  // Looks up channel by room code, then calls joinByChannelId()
await chat.join();
```

**Why two methods?**
- All channels have IDs (required), but room codes are optional
- Private/system channels may not have room codes
- `joinByRoomCode()` automatically fetches channel info and extracts the ID
- Both methods store the full channel object in `chat.selectedChannel`

### User List (Quick Chat)

Start a 1:1 video chat by selecting a user. This creates (or retrieves) a private channel between you and that user:

```javascript
chat.onLogin = async (user) => {
    await chat.loadUsers();
};

chat.onUsersLoaded = (users) => {
    // users = [{ id, username, avatar_pic, status, ... }, ...]
    users.forEach(u => {
        dropdown.add(new Option(u.username, u.id));
    });
};
```

When the user selects someone to chat with:

```javascript
dropdown.onchange = async () => {
    const userId = dropdown.value;
    await chat.selectUser(userId);  // Creates/gets quick chat channel
    await chat.join();
};
```

`selectUser()` handles everything: it calls `createQuickChatChannel` on the server, sets up the connection, and triggers `onChannelSelected` with the resulting channel. From there, the flow is identical to channel selection.

**Key properties in each user object:**
- `id` â€” unique user identifier (use with `selectUser()`)
- `username` â€” display name
- `avatar_pic` â€” profile image path
- `status` â€” user's current status

---

## Implementation Example

The file **`minichat-example.html`** demonstrates all three entry paths with complete working code:

- **Lines 40-75**: Three-block HTML layout for Login / Create Room / Join Room
- **Lines 140-170**: Event handler setup (onLogin, onJoined, onMemberStreamStart, etc.)
- **Lines 240-290**: Button handler functions showing the full flow
- **Lines 320-370**: Helper functions for creating/removing video tiles

Study this file to see how events flow and how little JavaScript is actually needed. The entire application is under 200 lines of JS, and most of that is UI manipulation.

---

## Tips for Your App

1. **Always call `join()` after selecting a channel** â€” `selectChannel()`, `joinByChannelId()`, and `joinByRoomCode()` only configure which room to enter; `join()` actually connects.

2. **Create video elements in `onMemberStreamStart`** â€” Don't pre-create them. The event tells you when a stream is ready.

3. **Use `onMemberMediaChange` for mute indicators** â€” When members mute, this event fires so you can update UI (show a muted icon, etc.).

4. **Check `chat.currentRoomCode` after joining** â€” Display this so users can share it with others.

5. **Check room capacity before joining** â€” Channel info includes a `members` array, so you can check capacity:
   ```javascript
   // Get channel info by room code
   const channel = await chat.getChannelByRoomCode(roomCode);
   const count = channel.members ? channel.members.length : 0;
   if (count >= 2) {
       alert('Room is full');
       return;
   }
   ```
   Note: You must be logged in (even anonymously) to get channel info.

6. **Handle `onError` for debugging** â€” During development, log all errors to understand what's happening.

---

## Next Steps

1. Open `minichat-example.html` in your browser and test the three entry flows
2. Open Developer Tools and watch the Event Log to understand the sequence
3. Copy the patterns into your own app, adapting the UI to your design

The MiniChatCore API is intentionally small. Master these ~15 methods and ~10 events, and you can build any video chat experience.

---

*For questions or to obtain context credentials, contact the Makedo team.*