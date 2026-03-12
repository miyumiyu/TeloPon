# 💬 Comment Generator File Reader (CommentGenerator_read.py)

This plugin monitors **comment data (`comment.xml`) output on the local PC and feeds viewer comments to the AI** in real time.

The greatest strength of this feature is that it works not just with YouTube, but **with any streaming platform — Twitch, Niconico Live, TwitCasting, Mirrativ, and more** — allowing the AI to pick up comments from anywhere!

---

## 🛠️ Setup: Creating a System to Output comment.xml

TeloPon alone cannot directly fetch comments from platforms other than YouTube. You need to combine two free external tools to output a `comment.xml` file on your PC.

### Step 1: Download Two Required Applications

1. **Multi Comment Viewer**
   Software that collects comments from multiple streaming platforms in one place.
   👉 Download the "stable version" from the [Multi Comment Viewer official site](https://ryu-s.github.io/app/multicommentviewer) and extract it.
2. **HTML5 Comment Generator**
   A tool for outputting received comments as a file (XML).
   👉 Download the latest version (e.g., `hcg_0_0_8b.zip`) from [KILINBOX (official site)](https://lib.kilinbox.net/comegene/index.cgi) and extract it.

### Step 2: Link the Two Applications

1. Double-click `MultiCommentViewer.exe` inside the extracted "Multi Comment Viewer" folder to launch it.
2. From the top menu, click **"Plugins"** then **"CommentGenerator Integration"**.
3. In the settings window that opens, check **"Enable CommentGenerator Integration"**.
4. Click the **"Browse"** button next to "Settings file location" and select the **`setting.xml`** file inside the "HTML5 Comment Generator" folder you extracted in Step 1.

### Step 3: Confirm comment.xml Output

1. Add your stream URL via "Add Connection" in Multi Comment Viewer and start fetching comments.
2. When comments are received, a **`comment.xml`** file will be automatically created (or updated) inside the specified HTML5 Comment Generator folder.

Setup is complete! Now have TeloPon monitor this `comment.xml`.

---

## ⚙️ TeloPon Settings and Usage

### 1. Open the Settings Panel
Click the **"⚙️ Settings"** button for **"Comment Generator File Reader"** in the "Extensions (Plugins)" panel on the right side of the TeloPon main screen.

![Comment Generator File Reader settings](../../../images/CommentGenerator_read.png)

### 2. Specify the File Path
Click the **"Browse..."** button next to **"Comment file to monitor (XML/TXT)"**.
Select the **`comment.xml`** file inside the HTML5 Comment Generator folder that you confirmed in Step 3.

### 3. Set the Send Interval (Cooldown)
Set the **"Send interval to AI (seconds)"** (default: `5.0` seconds recommended).
* Sending to the AI on every single comment would overwhelm it and cause malfunctions, so the plugin "banks" comments for the configured number of seconds before sending them all at once.
* For high-traffic streams, it's recommended to set a slightly longer value like `10`.

### 4. Enable and Save
* Check the **"Enable comment integration"** checkbox at the top of the screen.
* Press **"Save"** to close the window.
* The plugin badge in the main screen will change from grey `OFF` to green `ON`.

### 5. Start the Live Connection
Press the **"🔴 Start Live Connection"** button on the TeloPon main screen to begin the AI session.
After that, whenever a viewer comments and `comment.xml` is updated, comments will be forwarded to the AI at the configured interval, and the AI will react!

---

## ⚠️ Notes

* **File write conflicts**
  Very rarely, TeloPon may attempt to read the file while the comment generator is writing to it, causing a temporary read error. The system automatically retries, so there is no impact on streaming.
* **Using alongside the YouTube Integration Plugin**
  If you enable both the "YouTube Integration Tool" and this "Comment Generator File Reader" **at the same time, the same comments will be sent to the AI twice.** When streaming on YouTube, please enable only one of them.

---
[⬅️ Back to Plugin List](../../../README.md)
