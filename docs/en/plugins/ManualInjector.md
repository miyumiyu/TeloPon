# 💉 Manual Data Injection Tool (ManualInjector.py)

This plugin is an extremely powerful director's tool that lets you **directly inject "cue cards (text)" or "images (visual information)" into the AI's brain with a single button press during a stream**.

It's invaluable when you want to control the flow of your stream in real time — "Look at this gacha result!", "Guide the stream toward the closing greeting!" and so on.

---

## ⚙️ Basic Usage

This tool is a "TOOL type" plugin, so there is no ON/OFF switch. You can open the control panel whenever you want to use it.

### 1. Start the Live Connection
To send data to the AI with this tool, **TeloPon must be in an active AI session (🔴 Live Connected).**
First, start the live connection from the main screen.

### 2. Open the Control Panel
Click the **"🖥️ Control Panel"** button for **"Manual Data Injection Tool"** in the "Extensions (Plugins)" panel on the right side of the main screen.
(※ The panel stays on top of other windows so it doesn't get hidden behind OBS etc.)

![Manual Data Injection Tool control panel](../../../images/ManualInjector.png)

### 3. Set the Data to Send
Once the panel opens, enter the information you want to send to the AI. (※ You can send text only, image only, or both at the same time.)

* 👤 **Sender / Title (top field)**
  A title that tells the AI "who this instruction is from and what it's about". The default is "[Forced Cue Card from Program Director]", but changing it to something like "Revelation from the Gods" or "System Message" can produce interesting reactions.
* 📝 **Cue Card / Instructions for AI (bottom field)**
  Write the message or instruction you want to deliver directly to the AI.
  (e.g., "Comment on this image in a funny way!", "Remind me it's been an hour since the stream started and suggest taking a break.")
* 📸 **Image to Show the AI (visual information)**
  Press the "📁 Select Image File" button and choose an image (PNG, JPG, etc.) from your PC. The selected image is shown in a preview inside the panel.
  (※ Images are automatically resized and compressed internally for easy transmission — no need to worry.)

### 4. Send (Inject) to the AI!
Press the large blue **"🚀 Inject Data to AI (SEND)!"** button at the bottom.
If successful, "✅ Data injected to AI!" will appear, the AI receives the information, and starts responding immediately! (Text and images are automatically cleared after sending.)

---

## 💡 Creative Ideas for Streaming

Using this tool makes truly interactive AI-powered streaming possible — far beyond simple voice conversation.

* 🎮 **Share gacha results or screenshots**
  When you get a rare item in a game or right after pulling a gacha, select a screenshot, add text like "What do you think of this result?" and send it! The AI will see the image and celebrate or commiserate with you.
* 🤫 **Share photos**
  Share photos of things that happened today — the AI will see the photo and comment on it.
* 🧭 **Redirect the conversation**
  When the stream is getting off track or you want to change topics, send a cue card like "Let's talk about X next" to use the AI as a natural MC.

---

## ⚠️ Notes

* **Timing of sends**
  If you inject data while the AI is already in the middle of delivering a long response, the AI may not be able to process both and the messages may mix or be temporarily skipped. Send when the AI has finished speaking or during a brief pause for the smoothest results.
* **Image size**
  If you send an image with very small text (like a full PC screen), the AI may not be able to read the text accurately. The key is to select an image where the thing you want to show is clearly visible.

---
[⬅️ Back to Plugin List](../../../README.md)
