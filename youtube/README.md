# YouTube Live Chat Overlay

A stunning, glassmorphism-style YouTube Live chat overlay for OBS Studio.  
Self-contained single HTML file — no server required.

---

## ✨ Features

- 🎨 Dark glassmorphism aesthetic with YouTube red accents
- 🌊 Spring slide-in animations for new messages
- ⏱️ Messages auto-fade after 30 seconds
- 🎨 Consistent per-user color from a curated 20-color palette
- 🛡️ Badges: Moderator, Member/Subscriber, Owner, Verified
- 💛 Super Chat support with 7 tier color coding + pulsing glow
- 🤖 Demo mode with realistic fake messages and super chats
- 📺 Max 12 messages visible — oldest auto-removed

---

## 🚀 Quick Start (OBS Setup)

### Step 1 — Add a Browser Source

1. In OBS, click **+** in the Sources panel
2. Select **Browser**
3. Name it `YouTube Chat Overlay`

### Step 2 — Configure the Browser Source

| Setting | Value |
|---------|-------|
| **Local file** | ✅ Checked |
| **File path** | `C:\Users\_k3t_\Desktop\stream ext's\youtube\index.html` |
| **Width** | `400` |
| **Height** | `900` (or your canvas height) |
| **Custom CSS** | *(leave blank)* |
| **Shutdown source when not visible** | ✅ Recommended |
| **Refresh browser when scene becomes active** | ✅ Recommended |

> **Tip:** The background is fully transparent — no chroma key needed. Just position the overlay in the bottom-left of your scene.

### Step 3 — Demo Mode (Default)

By default, `DEMO_MODE = true` in the HTML file. This runs a realistic simulation with random users, messages, and super chats. Perfect for testing your layout.

---

## 🔧 Configuration

Open `index.html` in a text editor and find these lines near the top of the `<script>` tag:

```js
const VIDEO_ID  = 'your_video_id'; // <-- Change this to your YouTube Live video ID
const DEMO_MODE = true;            // Set to false and add VIDEO_ID for real YouTube chat
```

---

## 📡 Real YouTube Chat Integration

### Method 1 — YouTube Live Chat Embed (Simplest, No API Key)

YouTube provides a native chat embed URL:

```
https://www.youtube.com/live_chat?v=VIDEO_ID&embed_domain=localhost
```

**How to use it:**

1. Add a **second Browser Source** in OBS pointing to the URL above  
   (replace `VIDEO_ID` with your actual live video ID)
2. Resize it to a small corner of your scene (or make it invisible)
3. Overlay *this* HTML file on top as your styled layer

> ⚠️ Due to YouTube's cross-origin restrictions, the chat iframe's messages cannot be intercepted via JavaScript from a separate file. The iframe approach works best as a **visual fallback**.

---

### Method 2 — YouTube Data API v3 (Full Integration)

This method lets you poll real chat messages and display them through the beautiful overlay.

#### Step 1 — Get a YouTube Data API Key

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select existing)
3. Navigate to **APIs & Services → Library**
4. Search for **YouTube Data API v3** and click **Enable**
5. Go to **APIs & Services → Credentials**
6. Click **Create Credentials → API Key**
7. Copy the API key

#### Step 2 — Get Your Live Chat ID

Every YouTube Live stream has a `liveChatId`. You need this to poll messages.

**Option A — From Video URL:**
Your video URL looks like: `https://www.youtube.com/watch?v=VIDEO_ID`  
The `VIDEO_ID` is the part after `?v=`.

**Option B — Via API:**
```
GET https://www.googleapis.com/youtube/v3/videos
  ?part=liveStreamingDetails
  &id=VIDEO_ID
  &key=YOUR_API_KEY
```
The `liveChatId` is inside `liveStreamingDetails.activeLiveChatId`.

#### Step 3 — Poll for Messages

```js
async function pollChat(liveChatId, apiKey, pageToken) {
  const url = new URL('https://www.googleapis.com/youtube/v3/liveChat/messages');
  url.searchParams.set('liveChatId', liveChatId);
  url.searchParams.set('part', 'snippet,authorDetails');
  url.searchParams.set('key', apiKey);
  if (pageToken) url.searchParams.set('pageToken', pageToken);

  const res  = await fetch(url);
  const data = await res.json();

  for (const item of data.items) {
    const snippet = item.snippet;
    const author  = item.authorDetails;

    if (snippet.type === 'superChatEvent') {
      window.ytChat.addMessage({
        username:    author.displayName,
        badges:      getBadges(author),
        text:        snippet.superChatDetails.userComment || '',
        isSuperchat: true,
        scAmount:    snippet.superChatDetails.amountDisplayString
                       .replace(/[^0-9.]/g, '') * 1,
        timestamp:   new Date(snippet.publishedAt),
      });
    } else if (snippet.type === 'textMessageEvent') {
      window.ytChat.addMessage({
        username:  author.displayName,
        badges:    getBadges(author),
        text:      snippet.textMessageDetails.messageText,
        timestamp: new Date(snippet.publishedAt),
      });
    }
  }

  // Schedule next poll (YouTube recommends waiting pollingIntervalMillis)
  const interval = data.pollingIntervalMillis || 5000;
  setTimeout(() => pollChat(liveChatId, apiKey, data.nextPageToken), interval);
}

function getBadges(author) {
  const badges = [];
  if (author.isChatOwner)      badges.push('owner');
  if (author.isChatModerator)  badges.push('mod');
  if (author.isChatSponsor)    badges.push('member');
  if (author.isVerified)       badges.push('verified');
  return badges;
}
```

Add this script to `index.html` (before `</body>`) and call:

```js
// After page loads, when your stream is live:
const API_KEY     = 'YOUR_API_KEY_HERE';
const LIVE_CHAT_ID = 'YOUR_LIVE_CHAT_ID_HERE';

pollChat(LIVE_CHAT_ID, API_KEY);
```

> **API Quota Note:** YouTube Data API v3 has a daily quota of 10,000 units. Each `liveChat.messages.list` call costs 5 units. At the minimum polling interval (~5s), that's ~1,728 calls/day × 5 = **8,640 units/day** — just within the free tier for a single stream.

---

## 🎛️ Public JavaScript API

You can push custom messages from the OBS browser source console:

```js
// Simple message
window.ytChat.addMessage({
  username: 'CoolViewer',
  badges:   ['mod'],
  text:     'Hello from the console!',
});

// Super Chat
window.ytChat.addMessage({
  username:    'BigFan',
  badges:      ['member'],
  text:        'You are the GOAT!',
  isSuperchat: true,
  scAmount:    50.00,
});
```

---

## 🎨 Customization

All visual tokens are CSS custom properties. Edit inside `<style>` in `index.html`:

| Variable | Default | Purpose |
|----------|---------|---------|
| `--glass-bg` | `rgba(14,14,20,0.72)` | Chat bubble background |
| `--glass-blur` | `14px` | Frosted glass blur intensity |
| `--glass-border` | `rgba(255,255,255,0.07)` | Bubble border color |
| `--radius-msg` | `14px` | Message bubble corner radius |
| `MAX_MESSAGES` | `12` | Max visible messages (JS constant) |
| `FADE_AFTER_MS` | `30000` | Milliseconds before message fades |
| `SHOW_BRAND` | `true` | Toggle YouTube brand pill |

Change `#chat-overlay` width (default `380px`) to resize the chat panel.

---

## 🛡️ Badge Reference

| Badge | Emoji | Trigger |
|-------|-------|---------|
| Owner | 👑 | `badges: ['owner']` |
| Moderator | 🛡️ | `badges: ['mod']` |
| Member/Sub | ⭐ | `badges: ['member']` |
| Verified | ✔️ | `badges: ['verified']` |

---

## 💛 Super Chat Tiers

| Amount | Color | Emoji |
|--------|-------|-------|
| $500+ | Dark Red | 🔴 |
| $100+ | Purple | 💜 |
| $50+ | Orange | 🟠 |
| $20+ | Yellow | 🟡 |
| $10+ | Teal | 🟢 |
| $5+ | Blue | 🔵 |
| $1+ | Light Blue | 🔵 |

---

## 📁 File Structure

```
stream ext's/
└── youtube/
    ├── index.html   ← The overlay (add as OBS Browser Source)
    └── README.md    ← This file
```

---

*Built with ❤️ for streamers — no dependencies, fully offline-capable.*
