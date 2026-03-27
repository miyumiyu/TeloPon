# ▶️ YouTube Integration Tool (YoutubeLivePlugin.py)

This plugin lets you simply specify your YouTube Live stream URL to **pick up viewer comments in real time and have the AI react to them**.

Even better, it doesn't just get comments — it **automatically fetches the stream's "title", "description", and "thumbnail image" and sends them to the AI's brain**, so the AI fully understands what today's stream is about and co-hosts the show as your perfect partner!

---

## ⚙️ Usage

### 1. Open the Settings Panel
Click the **"⚙️ Settings"** button for **"YouTube Integration Tool"** in the "Extensions (Plugins)" panel on the right side of the TeloPon main screen.

![YouTube Integration settings panel](../../../images/YoutubeLivePlugin.png)

### 2. Enter the URL and "Connect"
Paste the **YouTube Live URL** of your current (or upcoming) stream into the input field at the top, then press the **"Connect"** button on the right.
*(※ Please enter the public viewer URL, not a regular video URL or the YouTube Studio dashboard URL.)*

### 3. Confirm the Information Was Fetched
When the connection succeeds, "✅ Connection successful" will appear on screen, and the stream's **"title", "description", and "thumbnail image"** will be shown in a preview.
At this point, the plugin badge in the main screen also turns green `ON`. Press "Close" to dismiss the panel.

### 4. Start the Live Connection
Return to the TeloPon main screen and press **"🔴 Start Live Connection"** to begin the AI session.

That's it — integration complete!
As soon as the AI session begins, the YouTube stream information is shared with the AI, and it will automatically react to viewer comments.

---

## 🧠 Information Shared with the AI

Using this tool, the following information is automatically sent to the AI.

1. **Basic stream info (at connection time)**
   The fetched "stream title" and "description" are quietly appended to the end of the AI's base system prompt. This allows the AI to start speaking with prior knowledge of "what game are we playing today" or "what kind of event is this".
2. **Thumbnail image visual info (5 seconds after session start)**
   About 5 seconds after TeloPon's live connection starts, the AI's vision is forcibly shown **"today's thumbnail image"** along with a director's cue: "Look at this and share your thoughts!" This naturally leads to opening talk about the thumbnail.
3. **Viewer real-time comments (ongoing)**
   The stream's chat is monitored in the background, and new comments are sent to the AI as they arrive. The AI reads out comments and answers questions.

---

## ⚠️ Notes

* **Order of operations (starting TeloPon live connection)**
  To ensure the AI properly recognizes the stream title and description, it is strongly recommended to **"Connect" to the YouTube URL in this tool before pressing TeloPon's "🔴 Start Live Connection" button**. (If you connect mid-session, the title and description won't be recognized — only comments and the thumbnail will be sent.)
* **When there are too many comments**
  For large streams where comments flow like a waterfall, the AI may try to react to every comment and end up talking non-stop, potentially overloading.
* **Private/unlisted streams**
  This tool fetches information from publicly accessible YouTube pages, so it cannot connect to private streams. (Unlisted streams can be connected to if you have the URL.)

---
[⬅️ Back to Plugin List](../../../README_en.md)
