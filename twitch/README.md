# 🎮 Twitch Chat Overlay

A stunning glassmorphism Twitch chat overlay built for OBS. Features a dark purple aesthetic matching Twitch's brand colors, smooth spring animations, real-time chat via tmi.js, and a built-in demo mode.

---

## 📁 Files

| File | Purpose |
|------|---------|
| `index.html` | The overlay — fully self-contained, drop into OBS |

---

## ⚙️ Configuration

Open `index.html` in any text editor and change the two constants near the top of the `<script>` block:

```js
const CHANNEL   = 'your_channel_name'; // ← your Twitch username / channel
const DEMO_MODE = true;                // ← set to false for real Twitch chat
```

> **Example:** If your Twitch channel is `twitch.tv/examplestreamer`, set `CHANNEL = 'examplestreamer'`

---

## 🖥️ OBS Setup

### Step 1 — Add a Browser Source

1. In OBS, click **+** under **Sources**
2. Select **Browser**
3. Give it a name, e.g. `Twitch Chat Overlay`

### Step 2 — Configure the Browser Source

| Setting | Value |
|---------|-------|
| **Local file** | ✅ Check this box |
| **File path** | Browse to `index.html` (this file) |
| **Width** | `400` (recommended) |
| **Height** | `800` (recommended, or match your scene height) |
| **Custom CSS** | *(leave blank)* |
| **Shutdown source when not visible** | ✅ Recommended |
| **Refresh browser when scene becomes active** | ✅ Recommended |

### Step 3 — Position in your Scene

- Drag the browser source to wherever you want chat to appear
- Resize handles in the scene preview to fit your layout
- The background is **fully transparent** — it will blend into your stream perfectly

> [!TIP]
> Right-click the source → **Filters** → add a **Color Correction** filter if you need to fine-tune opacity or brightness.

---

## 🔴 Going Live with Real Twitch Chat

1. Open `index.html` in a text editor
2. Set your channel:
   ```js
   const CHANNEL   = 'your_actual_channel';
   const DEMO_MODE = false;
   ```
3. Save the file
4. In OBS, right-click your Browser Source → **Refresh**

The overlay connects **anonymously** (read-only) — no login, no OAuth token required.

> [!NOTE]
> Anonymous connection means the overlay can **read** chat but cannot send messages. This is perfect for an overlay.

---

## 🎨 Features

| Feature | Details |
|---------|---------|
| **Glassmorphism UI** | Frosted glass bubbles, purple/magenta Twitch palette (`#9146FF`, `#BF94E4`) |
| **Spring animations** | Messages slide in with a `cubic-bezier(0.34, 1.56, 0.64, 1)` spring bounce |
| **Auto-fade** | Each message fades out after **30 seconds** |
| **Max messages** | Oldest message removed when **12** are visible |
| **Username colors** | Each user gets a consistent color from a curated 20-color vibrant palette |
| **Badges** | 👑 Broadcaster · ⚔️ Mod · 💎 VIP · ⭐ Subscriber |
| **Emote chips** | Known emote words (PogChamp, LUL, KEKW, etc.) render as styled chips |
| **Timestamps** | Each message shows the time it was received |
| **Demo mode** | 15 fake users, 40+ realistic messages, random intervals (1.6–4.2s) |
| **Live mode** | Real Twitch IRC via [tmi.js](https://tmijs.com/) CDN |

---

## 🛠️ Customization Tips

### Change the fade duration
```js
const FADE_AFTER_MS = 30000; // milliseconds — 30000 = 30 seconds
```

### Change max visible messages
```js
const MAX_MESSAGES = 12; // reduce for less clutter, increase for busier chat
```

### Change demo message speed
```js
const DEMO_INTERVAL_MIN_MS = 1600; // minimum ms between messages
const DEMO_INTERVAL_MAX_MS = 4200; // maximum ms between messages
```

### Resize the overlay
Change `Width` and `Height` in the OBS Browser Source settings. The layout automatically adjusts.

### Add your own emote names
Find the `EMOTE_TOKENS` set near the top of the script and add your channel's custom emotes:
```js
const EMOTE_TOKENS = new Set([
  // ... existing emotes ...
  'YourEmote1', 'YourEmote2',
]);
```

---

## ❓ Troubleshooting

| Problem | Solution |
|---------|----------|
| Chat not appearing | Make sure `DEMO_MODE = false` and `CHANNEL` is correct |
| Blank overlay in OBS | Enable **Local file** in Browser Source settings |
| Chat connected but no messages | Check your channel name — it's case-insensitive but must be exact |
| Overlay not transparent | Make sure you're using it as a **Browser Source**, not an image |
| Messages not fading | OBS Browser Sources throttle JS when the scene isn't active — switch to the scene to test |

---

## 📦 Dependencies (CDN, no download needed)

- [tmi.js 1.4.2](https://cdn.jsdelivr.net/npm/tmi.js@1.4.2/dist/tmi.min.js) — Twitch IRC client
- [Google Fonts: Outfit](https://fonts.google.com/specimen/Outfit) — Typography

> [!IMPORTANT]
> Both CDN links require an **internet connection** on the machine running OBS. The overlay does **not** work fully offline unless you replace these with local copies.

---

*Built with 💜 for streamers.*
