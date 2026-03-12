# 🎮 Twitch Live Plugin (TwitchPlugin.py)

This plugin lets you simply specify a Twitch channel name or URL to **pick up viewer chat comments in real time and have the AI react to them**.

Additionally, if you set a **Twitch Client ID and Client Secret** (optional), the stream's "title", "category", and "thumbnail image" are also automatically fetched and sent to the AI. **The plugin works for chat reading even without a Client ID.**

---

## ⚙️ Usage

### 1. Open the Settings Panel

Click the **"⚙️ Settings"** button for **"Twitch Live Plugin"** in the "Extensions (Plugins)" panel on the right side of the TeloPon main screen.

### 2. Enter the Channel Name

Enter the Twitch channel name (e.g., `mychannel`) or URL (e.g., `https://twitch.tv/mychannel`) in the input field at the top.

### 3. Set Client ID / Client Secret (Optional)

If you also want to share the stream title, category, and thumbnail with the AI, enter your **"🔑 Client ID"** and **"🔒 Client Secret"**.

> ⚠️ **Chat reading works without these credentials.** You can skip this step if you only need chat integration.

**How to get your Client ID / Client Secret:**

1. Log in to [https://dev.twitch.tv/console](https://dev.twitch.tv/console) with your Twitch account.
2. Register a new application under "Register Your Application" (name and category are your choice).
3. Copy the "Client ID" and generate a "New Secret", then paste them into the fields.

### 4. Press the "Connect" Button

Once your input is ready, press the **"Connect"** button.

* **Without Client ID**: "✅ Chat connected" will appear, and chat reading becomes active.
* **With Client ID**: Stream info and thumbnail will be fetched and displayed, showing "✅ Connected".

Confirm the plugin badge turns green `ON`, then close the panel.

### 5. Start the Live Connection

Return to the TeloPon main screen and press **"🔴 Start Live Connection"** to begin the AI session.

That's it — integration complete! As soon as the AI session begins, Twitch information is shared with the AI, and it will automatically react to viewer comments.

---

## 🧠 Information Shared with the AI

| Information | Requirement | Timing |
|---|---|---|
| Stream title & category | Client ID required | At live session start |
| Thumbnail image | Client ID required | ~5 seconds after live session start |
| Viewer chat comments | Always (anonymous IRC) | Real-time (batched every 5 seconds) |

### Comment Injection Behavior

* Chat comments are sent to the AI in batches every **5 seconds**, or when **10 comments** accumulate.
* Each comment is delivered to the AI in the format: `[COMMENT] username: comment text`

---

## ⚠️ Notes

* **Order of operations**
  To ensure the AI properly recognizes the stream title and category, it is strongly recommended to **"Connect" in this tool before pressing TeloPon's "🔴 Start Live Connection" button**. (If you connect mid-session, the title and category won't be recognized.)

* **Offline channels**
  When using Client ID mode, if the target channel is offline (not currently streaming), stream info cannot be fetched. Chat connection will still work.

* **When there are too many comments**
  For large streams where comments flow rapidly, the AI may try to react to every comment and end up talking non-stop.

---

[⬅️ Back to Plugin List](../../../README.md)
