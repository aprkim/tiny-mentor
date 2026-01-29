# MiniChatCore Developer Guide v0.26 January 28, 2026

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

This keeps your code reactive—you respond to events rather than polling for state.

### 2. Three Entry Paths

Users can enter a video chat session three ways:

| Path | Use Case | Methods |
|------|----------|---------|
| **Login** | Registered users with existing channels | `login()` → `loadChannels()` → `selectChannel()` |
| **Create Room** | Anonymous user starts a new room | `signupAnonymous()` → `createChannel()` → `joinByCode()` |
| **Join Room** | Anonymous user joins via room code | `signupAnonymous()` → `joinByCode()` |

The room code (a short string like `"K7xmQ"`) is shareable—give it to others so they can join.

**Important: Room codes are case-sensitive.** Always preserve the original case when storing, displaying, or sharing room codes. Do not convert to uppercase or lowercase.

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
await chat.joinByCode('ROOM_CODE');
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

### Step 5: Handle Screen Share

Screen share uses the same event pattern as camera, but with `streamType === 'screencast'`:

```javascript
// Handle remote screen share
chat.onMemberStreamStart = (memberId, streamType) => {
    if (streamType === 'screencast') {
        // Create or show screen share video element
        chat.setMemberVideoElement(memberId, 'screencast', screenShareVideo);
        screenShareTile.style.display = 'flex';
    }
};

chat.onMemberStreamEnd = (memberId, streamType) => {
    if (streamType === 'screencast') {
        screenShareTile.style.display = 'none';
    }
};

// Handle local screen share
chat.onLocalMediaChange = () => {
    const screenState = chat.screencastState;
    const isSharing = screenState.video && !screenState.videoMuted;

    if (isSharing) {
        chat.setLocalScreencastElement(screenShareVideo);
        screenShareTile.style.display = 'flex';
    } else {
        screenShareTile.style.display = 'none';
    }
};
```

**Key points:**
- `toggleScreencast()` prompts the browser's screen picker
- `streamType` is `'screencast'` (not `'camera'`)
- Use `chat.setLocalScreencastElement()` for local preview
- Use `chat.setMemberVideoElement(id, 'screencast', el)` for remote
- Check `chat.screencastState` for local screen share status

---

## Key Events Reference

| Event | Parameters | When It Fires |
|-------|------------|---------------|
| `onLogin` | `(user)` | After successful login/signup |
| `onChannelsLoaded` | `(channels[])` | After `loadChannels()` completes |
| `onUsersLoaded` | `(users[])` | After `loadUsers()` completes |
| `onChannelSelected` | `(channel)` | After `selectChannel()`, `selectUser()`, or `joinByCode()` |
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

// Check room capacity before joining (requires login first)
await chat.getChannelMemberCount(roomCode)  // returns number of members in room
```

---

## Timing & Synchronization Patterns

WebRTC member synchronization is inherently asynchronous. Events may not fire exactly when you expect, and member data may not be complete immediately. Use these patterns to build robust apps.

### 1. Room Codes Are Case-Sensitive

Room codes like `"K7xmQ"` must preserve their original case. **Never** convert to uppercase or lowercase:

```javascript
// WRONG - will fail to join
const code = userInput.toUpperCase();
await chat.joinByCode(code);

// CORRECT - preserve original case
const code = userInput.trim();
await chat.joinByCode(code);
```

When displaying room codes in your UI, keep the original case. If a user enters the code manually, accept it exactly as typed.

### 2. Display Room Code Immediately

Don't wait for `onJoined` to display the room code—it's available right after `createChannel()` or you already have it from `joinByCode()`:

```javascript
// Room creator
const channel = await chat.createChannel({ title: 'My Room' });
roomCodeDisplay.textContent = channel.pid;  // Show immediately!
await chat.joinByCode(channel.pid);
await chat.join();

// Room joiner
const code = roomCodeInput.value;
await chat.joinByCode(code);
roomCodeDisplay.textContent = code;  // Show immediately!
await chat.join();
```

This ensures users can share the room code while the connection is still establishing.

### 3. Detecting Existing Members

When you join a room, `onMemberJoined` does **not** fire for members who are already there. You must actively check for them:

```javascript
chat.onJoined = () => {
    checkExistingMembers();

    // Retry with delays - member data may sync slowly
    setTimeout(checkExistingMembers, 500);
    setTimeout(checkExistingMembers, 1500);
    setTimeout(checkExistingMembers, 3000);
};

function checkExistingMembers() {
    const memberIds = chat.getMemberIds();
    memberIds.forEach(memberId => {
        const member = chat.getMember(memberId);
        // Set up video tile for this member
        if (member?.hasCamera) {
            chat.setMemberVideoElement(memberId, 'camera', videoElement);
        }
    });
}
```

Also use `onMemberStreamStart` as a backup detection point:

```javascript
chat.onMemberStreamStart = (memberId, streamType) => {
    if (streamType === 'camera') {
        // This fires even for existing members when their stream arrives
        // Use it as backup detection if onMemberJoined didn't fire
        if (!remoteMemberDetected) {
            remoteMemberDetected = true;
            setupRemoteTile(memberId);
        }
        chat.setMemberVideoElement(memberId, streamType, videoElement);
    }
};
```

### 4. Detecting New Members (Room Creator)

Even for the room creator, `onMemberJoined` may not fire reliably when guests join. Use polling as a backup:

```javascript
let pollInterval = null;

function pollForGuests() {
    pollInterval = setInterval(() => {
        if (guestDetected) {
            clearInterval(pollInterval);
            return;
        }

        const members = chat.getMemberIds();
        if (members.length > 0) {
            checkExistingMembers();
        }
    }, 1000);  // Poll every second
}

// Start polling after room creator joins
await chat.join();
pollForGuests();
```

### 4b. Detecting Existing Members (Guest/Joiner)

When joining an existing room, the host is already there but `onMemberJoined` won't fire (it only fires for members who join *after* you).

**Important limitation:** `getMemberIds()` only returns members who are actively streaming (have video or audio on). If the host has video/audio off, they won't appear in the list.

This creates an asymmetry:
- Host detects guest via `onMemberJoined` → works even with video off
- Guest detects host via `getMemberIds()` → only works if host is streaming

**For video-off hosts**, the guest will only detect them when:
1. Host turns on video → `onMemberStreamStart` fires
2. Host turns on audio → `onMemberStreamStart` fires

Poll to detect the host (works when host is streaming):

```javascript
let hostPollInterval = null;

function pollForHost() {
    // Check immediately first
    checkExistingMembers();

    hostPollInterval = setInterval(() => {
        if (hostDetected) {
            clearInterval(hostPollInterval);
            return;
        }

        const members = chat.getMemberIds();
        if (members.length > 0) {
            checkExistingMembers();
        }
    }, 500);  // Poll every 500ms (faster than guest polling)

    // Stop after 30 seconds
    setTimeout(() => {
        if (hostPollInterval) {
            clearInterval(hostPollInterval);
        }
    }, 30000);
}

// Start polling after joining room
await chat.joinByCode(roomCode);
await chat.join();
pollForHost();
```

**Key differences from guest polling:**
- Poll faster (500ms vs 1000ms) since host should already be there
- Check immediately on join, then poll
- Add a timeout to prevent infinite polling

**Fallback for video-off hosts:** Rely on `onMemberStreamStart`:

```javascript
chat.onMemberStreamStart = (memberId, streamType) => {
    if (streamType === 'camera' && !hostDetected) {
        hostDetected = true;
        setupHostTile(memberId);
    }
};
```

When the host turns on video, this event fires and you can detect them. Until then, show an appropriate message like "Waiting for host to enable video..."

### 5. Member Names May Be Delayed

The `username` property on member objects may not be available immediately. Try getting it at multiple points:

```javascript
function updateMemberName(memberId) {
    const member = chat.getMember(memberId);
    const name = member?.username || member?.name;

    // Only update if we got a valid name
    if (name) {
        nameElement.textContent = name;
    }
    // Otherwise keep the current name or show a placeholder
}

// Try at multiple event points
chat.onMemberJoined = (memberId) => {
    updateMemberName(memberId);
};

chat.onMemberStreamStart = (memberId, streamType) => {
    updateMemberName(memberId);  // Data may be more complete now
};

chat.onMemberMediaChange = (memberId) => {
    updateMemberName(memberId);  // Another chance
};
```

**Never overwrite a good name with a fallback.** Only update when you have valid data.

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
- `id` — unique channel identifier (use with `selectChannel()`)
- `name` — display name
- `room_code` — shareable code for `joinByCode()`
- `description` — optional channel description

### User List (Quick Chat)

Start a 1:1 video chat by selecting a user. This creates (or retrieves) a private channel between you and that user:

```javascript
chat.onLogin = async (user) => {
    await chat.loadUsers();
};

chat.onUsersLoaded = (users) => {
    // users = [{ pid, username, avatar_pic, status, ... }, ...]
    users.forEach(u => {
        dropdown.add(new Option(u.username, u.pid));
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
- `pid` — unique user identifier (use with `selectUser()`)
- `username` — display name
- `avatar_pic` — profile image path
- `status` — user's current status

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

1. **Always call `join()` after selecting a channel** — `selectChannel()` and `joinByCode()` only configure which room to enter; `join()` actually connects.

2. **Create video elements in `onMemberStreamStart`** — Don't pre-create them. The event tells you when a stream is ready.

3. **Use `onMemberMediaChange` for mute indicators** — When members mute, this event fires so you can update UI (show a muted icon, etc.).

4. **Display room code immediately** — Don't wait for `onJoined`. Show the code right after `createChannel()` returns or after the user enters it for joining.

5. **Check room capacity before joining** — Use `getChannelMemberCount(roomCode)` to see how many people are in a room. Useful for limiting room sizes:
   ```javascript
   const count = await chat.getChannelMemberCount(roomCode);
   if (count >= 2) {
       alert('Room is full');
       return;
   }
   ```
   Note: You must be logged in (even anonymously) to check member count.

6. **Handle `onError` for debugging** — During development, log all errors to understand what's happening.

7. **Use multiple detection strategies** — Don't rely on a single event. Combine `onMemberJoined`, `getMemberIds()`, `onMemberStreamStart`, and polling for robust member detection.

8. **Preserve room code case** — Room codes are case-sensitive. Never convert to uppercase or lowercase.

---

## Common Gotchas

| Issue | Cause | Solution |
|-------|-------|----------|
| Can't join room | Room code case was changed | Preserve original case |
| Room code shows "------" | Waiting for `onJoined` | Display code immediately after `createChannel()` |
| Existing member not detected | `onMemberJoined` doesn't fire for existing members | Use `pollForHost()` pattern (Section 4b) |
| Member name shows "Guest" | Name not yet synced | Try getting name at multiple event points |
| New member not detected | `onMemberJoined` didn't fire | Add polling as backup |
| Video not showing | Stream not attached | Call `setMemberVideoElement()` in `onMemberStreamStart` |

---

## UI State Patterns

### 1. Waiting vs Connecting Messages

Show different messages based on user role:

```javascript
// Room creator - waiting for guest
await chat.createChannel({ title: 'My Room' });
waitingText.textContent = 'Waiting for guest...';

// Guest joining - connecting to host
await chat.joinByCode(roomCode);
waitingText.textContent = 'Connecting...';
```

This helps users understand their context in the room.

### 2. Avatar Status Indicators

When video is off, show BOTH video and audio status on the avatar fallback:

```javascript
function updateAvatarStatus(videoOn, audioOn) {
    videoStatus.classList.toggle('off', !videoOn);
    videoStatus.textContent = videoOn ? 'Video on' : 'Video off';

    audioStatus.classList.toggle('off', !audioOn);
    audioStatus.textContent = audioOn ? 'Audio on' : 'Audio off';
}

// Call in onLocalMediaChange
chat.onLocalMediaChange = () => {
    const state = chat.localMediaState;
    const videoOn = state.video && !state.videoMuted;
    const audioOn = state.audio && !state.audioMuted;
    updateAvatarStatus(videoOn, audioOn);
};
```

### 3. Member Left Handling

When a member leaves, provide clear feedback:

```javascript
chat.onMemberLeft = (memberId) => {
    if (memberId === remoteMemberId) {
        const leftName = remoteName.textContent;

        // Show notification
        showToast(`${leftName} has left the session`);

        // Update overlay text
        waitingText.textContent = `${leftName} left the call`;
        waitingOverlay.classList.remove('hidden');

        // Update tile
        remoteName.textContent = 'Disconnected';
        remoteAvatar.classList.remove('visible');
        roleBadge.style.display = 'none';

        remoteMemberId = null;
    }
};
```

**Key points:**
- Capture the name before clearing state
- Show toast notification
- Update waiting overlay with context
- Keep tile visible with disconnected state

---

## Next Steps

1. Open `minichat-example.html` in your browser and test the three entry flows
2. Open Developer Tools and watch the Event Log to understand the sequence
3. Copy the patterns into your own app, adapting the UI to your design

The MiniChatCore API is intentionally small. Master these ~15 methods and ~10 events, and you can build any video chat experience.

---

**Version:** 0.30
**Last Updated:** 2026-01-28
**Changes from v0.29:**
- Documented `getMemberIds()` limitation: only returns streaming members
- Added fallback strategy for detecting video-off hosts via `onMemberStreamStart`

**Changes from v0.28:**
- Added Section 4b: Detecting Existing Members (Guest/Joiner) with `pollForHost()` pattern

**Changes from v0.27:**
- Added "UI State Patterns" section with code examples:
  - Waiting vs Connecting messages (role-specific)
  - Avatar status indicators (video + audio)
  - Member left handling with notifications

**Changes from v0.25:**
- Added "Timing & Synchronization Patterns" section
- Emphasized room code case-sensitivity
- Documented existing member detection patterns
- Added polling for guest detection
- Added member name synchronization patterns
- Added "Common Gotchas" troubleshooting table
- Added "Step 5: Handle Screen Share" section with code examples

---

*For questions or to obtain context credentials, contact the Makedo team.*
