**English** | [日本語](README_ja.md) | [한국어](README_ko.md) | [Русский](README_ru.md)

# TeloPon 🎙️✨ - Early Access

> **⚠️ Important Notice for Downloaders** > This repository is the **distribution-only page** for "TeloPon Early Access (executable)". It is not a source code release or development repository.
>
> **Always download from the [Releases (Latest)] link below.** > 👉 **[Download TeloPon Latest Version](https://github.com/miyumiyu/TeloPon/releases/latest)**
> （※ Note: Downloading via the green "Code" button → "Download ZIP" will NOT work correctly!）

**Ultra-low-latency, feature-rich AI Streaming Assistant**

TeloPon leverages Google's latest AI "Gemini Live API" to listen to the streamer's voice and viewer comments, then **display real-time reactions, quips, and summaries as on-screen captions (telops)** — the next-generation streaming assistant tool.

This is not just "transcription software". It summons a talented, unique **AI co-host** for your stream!

![TeloPon Demo](images/demo.gif)

---

## 🌟 What Makes TeloPon Amazing!

![TeloPon Features](images/telopon1.png)

### 1. Overwhelming "Ultra-Low Latency" and "Context Understanding"
By directly calling Gemini's native audio API, TeloPon completely eliminates the time-consuming steps of "speech recognition → text conversion → sending to AI". It instantly reads the nuances of the streamer's words, sighs, and even laughter, returning **snappy responses that never break the flow of conversation**.

### 2. "Prompt Settings" to Create Your Own AI
Freely create "prompts (AI scripts / personality sheets)" as text files!
Simply write instructions like "You are an energetic assistant who speaks in Osaka dialect" or "When I'm at a loss for words, roast me without mercy" — you can **easily create any character AI** you can imagine.

### 3. Image Support! Infinite Extensibility with "Plugin Features"
Beyond the streamer's microphone audio, TeloPon can "secretly whisper" external information to the AI in real time.
* **YouTube Integration**: Have the AI read stream comments and co-host the show!
* **Manual Data Injection Panel**: During a stream, send "cue cards" with one button, or even **show the AI "images" and have it react**!
Simply write a simple Python script to add your own custom integrations (game linking, etc.).

### 4. Completely Free "Telop Design" with CSS
All telops displayed in OBS are rendered in HTML/CSS.
Add unlimited custom designs (themes) with just a bit of CSS — news broadcast style, variety show style, cyberpunk style, and more. **Sound effects (SE) for telop appearances are also freely configurable**.

### 5. Super-Easy OBS Integration & Silence Prevention System
Just launching TeloPon starts the built-in server — simply add the URL as an OBS "Browser Source" and you're ready to go.
Additionally, the **"Auto Talk (silence prevention)" feature** is built in, so when the streamer goes silent, the AI reads the atmosphere and proactively introduces a topic to prevent dead air.

---

## 🗺️ TeloPon Architecture

TeloPon uses an architecture where the central system called **"CORE (TeloPon Core)"** acts as a hub connecting the microphone, AI (Google Gemini API), OBS, and user configuration files.

The following diagram illustrates the data flow and the role of each component.

![TeloPon Architecture Diagram](images/architecture.png)

### 🔄 Data Flow and Component Roles

1. **🎤 Microphone Audio & Instructions (Voice & Instruction)**
   * Audio input from the streamer's microphone is sent through **CORE** to the **Google Gemini API** in real time.
   * Simultaneously, the contents of **prompt (.txt)** files in the user settings area are sent as instructions to the AI, determining the AI's character and behavior.

2. **🤖 Telop Information**
   * The AI thinks based on the audio and instructions, then returns the results (content, emotion, display style, etc.) to CORE as **telop information**.

3. **📺 Drawing Data & CSS Loading**
   * CORE converts the received telop information into **drawing data** (HTML) that can be displayed in a browser.
   * **OBS Studio (Browser Source)** accesses CORE and displays this drawing data.
   * At this point, the **theme (.css)** files in the user settings area are loaded, applying telop designs and animations.

---

## 🔑 Setup: Get Your Free API Key!

To run TeloPon, you need an AI key (API key) provided by Google.
A **completely free tier is available with no credit card required**, and it's more than enough for personal streaming and hobby use!

👉 **[Detailed instructions with images (beginner-friendly)](docs/en/04_get_apikey.md)**

1. Visit **[Google AI Studio](https://aistudio.google.com/)** and sign in with your Google account.
2. Click **"Get API key"** in the lower left.
3. Click the **"Create API key"** button in the upper right to create your key.
4. Copy the string starting with "AIza..." that appears and store it safely (never share it with others).

---

## 🛠️ Download and Launch (Windows Only)

### Step 1: Download the File
Download the latest `TeloPon-xxx.zip` from the **[Releases (Latest)](https://github.com/miyumiyu/TeloPon/releases/latest)** page on GitHub.

### Step 2: Extract
Right-click the downloaded ZIP file and select **"Extract All"** to unzip it.
> ⚠️ **Important:** If you double-click the contents directly inside the ZIP without extracting, settings will not be saved and it will not work properly. Always extract to a folder first.

### Step 3: Launch
Double-click **`TeloPon.exe`** inside the extracted folder to start.

---

## 📁 Folder Structure and Customization

After extracting the ZIP file, the structure looks like this. You can freely extend functionality by modifying the contents of each folder.

```text
TeloPon_Release/
 ├── TeloPon.exe         # Main application
 ├── icon/               # App icon images
 ├── plugins/            # 📦 Standard bundled plugins
 ├── prompts/            # 🧠 AI personality / scripts (text files)
 ├── sounds/             # 🎵 Sound effects for telop appearances
 └── themes/             # 🎨 Telop appearance / design (CSS)
```

---

## 🎛️ TeloPon UI — Detailed Usage Guide

When you launch the app, the main settings window appears.

![UI Screen](images/ui.png)

### ⚙️ 1. AI Settings (Basic Settings)
* **🔑 API Key**: Paste the key starting with `AIza...` and press the "Authenticate" button. When you see "✅ Authentication successful", you're ready to go.
* **🧠 Model**: Leave it as `Auto` by default.
* **🧠 AI Personality (Prompt)**: Select the AI script from the `prompts/` folder.
* **🎥 Streamer Name (※ Required)**: Enter your name. The AI uses this to address you.

### 🎤 2. Microphone Selection and Audio Level (★ Super Important!)
Select the microphone you speak into. The black bar below it is the **"Level Meter"** — when the microphone picks up sound, the bar moves and changes color.

**【Meter Color Meanings】**
* **No color (black)**: Not picking up sound.
* **Light yellow**: The AI has recognized "a voice is coming in" and has started listening.
* **Yellow**: **Best volume!** Adjust your microphone volume (on the PC side) so it stays yellow when you speak normally.
* **Red**: The sound is too loud (clipping). The AI won't be able to hear your words clearly, so lower the volume to avoid red.

**【✂️ Threshold Adjustment】**
The **"Threshold Slider"** below the meter is the boundary for "from what volume level up to treat sound as your voice and send to the AI".
* **Tip**: If your stream's BGM or game audio is coming into the microphone, move this slider to the right to increase the value. The key to making the AI work smartly is finding the point where **"just BGM playing doesn't trigger the meter, but it goes light yellow to yellow only when you speak"**!

### ⏱️ 3. Time Settings (Seconds)
* **Normal / Special**: The number of seconds that "normal telops" (upper right) and "explanation telops" (bottom) remain visible on OBS.
* **Auto Talk Enabled**: Check this box and set the "intervention time (seconds)" — **when you're silent for that many seconds, the AI reads the atmosphere and proactively introduces a topic to prevent dead air**.

### 🔌 4. Extensions (Plugins)
Useful tools are lined up in the right panel.
* **Settings Button**: Configure detailed settings such as integration URLs.
* **Control Panel Button**: Opens a dedicated window for tools you operate by hand during the stream, such as the manual data injection tool.

### 🎨 5. Theme (CSS)
Select the design (skin) of the telops displayed in OBS.

### 🎵 6. Telop Sound Effects (SE)
Toggle sound effects on/off for telop appearances and adjust volume (0–100%).
Adjust the slider to the optimal volume for your streaming environment (BGM volume, etc.).
※ Unchecking "Play sound effects" grays out and disables the slider.

### 🔗 7. Copy OBS URL
Copies the URL for use in OBS Browser Source to the clipboard.
#### 📺 How to Add to OBS

1. Press **"🔗 Copy OBS URL"** on the right side of the screen (click inside the dotted box).
2. In OBS Studio, add a source and select **"Browser"**, then paste the copied URL.
3. Set the size to a 16:9 ratio **(recommended: width 1664 / height 936)** and you're done!

**👉 Once all settings are complete, press the "🔴 Start Live Connection" button to begin real-time conversation with the AI!**

---

## 🔌 Plugin (Extension) Details

TeloPon has two types of plugins: "Standard Bundled Plugins" that come pre-installed, and "Official Extension Plugins" that individual users download and add as needed.

### 📦 Standard Bundled Plugins (Pre-installed)

* 💬 **Comment Generator File Reader** (`CommentGenerator_read.py`)
  Periodically reads text files output by external comment generators, and relays new comments to the AI.
  👉 [Detailed usage guide](docs/en/plugins/CommentGenerator_read.md)

* 📝 **Custom Instruction Addon Tool** (`custom_prompt.py`)
  At stream start, appends additional instructions to the base prompt (AI personality) — e.g., "Today I'll be playing ○○ game".
  👉 [Detailed usage guide](docs/en/plugins/custom_prompt.md)

* 💉 **Manual Data Injection Tool** (`ManualInjector.py`)
  During the stream, directly sends arbitrary text (cue cards) or images into the AI's brain from the dedicated control panel with one button.
  👉 [Detailed usage guide](docs/en/plugins/ManualInjector.md)

* 🎮 **OBS Screen AI Commentary** (`obs_capture.py`)
  Uses OBS-WebSocket to periodically capture the OBS preview screen and show it to the AI, enabling it to comment on and react to game screens or stream content.
  👉 [Detailed usage guide](docs/en/plugins/obs_capture.md)

* ▶️ **YouTube Integration Tool** (`YoutubeLivePlugin.py`)
  Simply specify a YouTube Live stream URL to have the AI pick up and react to viewer comments. The stream's "title", "description", and "thumbnail image" are also automatically sent to the AI, so it can deeply understand the stream content and co-host the show with you.
  👉 [Detailed usage guide](docs/en/plugins/YoutubeLivePlugin.md)

### 🌟 Official Extension Plugins (Individual Download)

To keep the main app light and simple, the following integrations are available as separate downloads for those who need them.
🔗 [TeloPon Official Extension Plugin Pack v1.0 (Discord & Slack)](https://github.com/miyumiyu/TeloPon/releases/tag/plugins-v1.0)

* 💬 **Discord Real-time Integration** (`discord_integration.py`)
  Fetches comments from a specified Discord server channel in real time and has the AI read them aloud. Features fully automatic invite URL generation, so complex Bot setup is completed with just one button.
  📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.0/discord_integration.py)
  👉 [Detailed usage guide](docs/en/plugins/discord_integration.md)

* 🏢 **Slack Comment Integration** (`slack_integration.py`)
  Fetches comments from a specified Slack workspace channel with zero delay via "Socket Mode". Alphanumeric Slack user IDs are automatically converted to names before being delivered to the AI.
  📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.0/slack_integration.py)
  👉 [Detailed usage guide](docs/en/plugins/slack_integration.md)

*(💡 Installation: Just place the downloaded `.py` file into TeloPon's `plugins` folder — that's all it takes!)*

---

## 🚥 Status Display Guide (AI State Visualization)

The "Status" display below the microphone settings shows the AI's current internal state and system communication status in real time.

### 🟢 Basic Statuses
* **⬛ Standby**: Before live connection. The AI is not active yet.
* **⏳ Preparing greeting...**: Immediately after connection. The AI is thinking of the "opening greeting".
* **🟢 On Air!**: Standby OK. Ready to pick up your voice at any time.
* **🎧 Listening...** (cyan): Detected your voice and is paying attention.
* **🧠 Thinking...** (orange): Detected that you finished speaking and is formulating a response.
* **🗣️ Outputting...** (green): Compiling thoughts and outputting as a telop on screen.
* **✨ Response complete** (green): Sign that the AI's turn ended normally.

### 👻 "Empty Turn (No Response)" Statuses
Displays why the AI finished a turn without saying anything.
* **👻 Speech skipped**: Your words were heard, but the AI read the atmosphere and decided "this is a moment to just listen quietly" (this is normal behavior).
* **👻 Noise skipped**: A cough, ambient noise, or AI processing lag prevented it from being recognized as speech.
* **👻 Comment skipped / Image skipped**: Information sent from a plugin was acknowledged, but the AI determined there was nothing particular to comment on.

### ⚠️ Warning / Error Statuses
* **⚠️ Interruption detected (Barge-in)**: You started speaking while the AI was talking, so the AI read the atmosphere and canceled its own speech midway.
* **🚫 Safety restriction**: The AI's safety filter (blocking violent/sexual content) triggered and the output was forcibly stopped.
* **⚠️ AI processing overload**: An error or processing timeout occurred on Google's servers.

---

## 💡 One-Point Tips for Stable Streaming

### 🔄 "Manual Refresh" for Long Streaming Sessions
After about 30 minutes from stream start, the AI's memory fills up, processing becomes heavy, and **"👻 Noise skipped" and "⚠️ AI processing overload" start occurring frequently**.
In that case, at a natural break in the stream (when topics change, etc.), **press the "⬛ Disconnect" button once, then immediately press "🔴 Start Live Connection" again**.
The AI's memory is completely cleared and it instantly returns to a sharp, responsive state!

---

## 🚀 Launch Options (Arguments)

You can enable advanced launch modes by adding a space and the following flags to the end of the "Target" field in your shortcut's properties.

| Option | Example | Description |
| :--- | :--- | :--- |
| `-d`<br>`--debug` | `TeloPon.exe -d` | **Debug mode**. Detailed communication logs and AI internal processing are displayed in the console window, and a manual control panel is added to the UI. |
| `-p`<br>`--port` | `TeloPon.exe -p 8080` | **Change port number** (default: `8000`). Change this if the local server port conflicts with another application and the display doesn't appear. |
| `-t`<br>`--temperature` | `TeloPon.exe -t 1.0` | **AI creativity (Temperature)** (default: `0.7`). Sets the randomness of AI responses. Higher = more unpredictable; lower = more straightforward. |
| `-tp`<br>`--top_p` | `TeloPon.exe -tp 0.8` | **AI diversity (Top-P)** (default: `0.8`). Sets the candidate selection range for AI responses. Lower values produce more stable responses. |
| `-th`<br>`--thought` | `TeloPon.exe -th` | **Thinking mode**. Outputs the AI's internal thought process (Thoughts) — "what it's thinking and why it gave that response" — to the console. |
| `-tb`<br>`--thinking_budget`| `TeloPon.exe -tb 1024` | **Thinking budget (token specification)** (default: `2048`). Specifies the token budget for thinking mode. `0` disables thinking. |
| `-gs`<br>`--google_search` | `TeloPon.exe -gs` | **Google Search integration**. Enables the AI to reference up-to-date information using Google Search when responding. |
| `-at`<br>`--auto_turn` | `TeloPon.exe -at 10` | **Auto-turn mode**. Locks the microphone when speech ends and unlocks it after the AI responds. Specify the timeout in seconds (default: 10). `0` = wait indefinitely. |
| `-seg`<br>`--segment` | `TeloPon.exe -seg` | **Periodic segment mode**. Uses the UI's Auto Talk timer as a cycle to periodically send summary/topic requests to the AI. |
| `-as`<br>`--audio_save` | `TeloPon.exe -as` | **Auto-save speech WAV**. Automatically saves speech to the `debug_audio/` folder (debug mode not required). |

---

## 📖 Development & Customization Documentation

Guides for those who want to hack TeloPon to their heart's content!

* 🧠 **[AI Prompt Creation Guide](docs/en/01_prompt_guide.md)**: How to create your own AI and the absolute rules to follow.
* 🎨 **[Theme / CSS Customization Guide](docs/en/02_theme_css.md)**: How to create custom designs and configure sound effects.
* 🧩 **[Plugin Development Guide](docs/en/03_plugin_dev.md)**: How to create custom extensions using the libraries bundled in the exe version (`requests`, `pytchat`, `obsws_python`, `Pillow`, etc.).

---

## ❓ Troubleshooting

### Q. Pressing the authenticate button shows "🔴 Authentication failed"
- Check that no extra spaces are included before or after the copied API key.
- Check the message displayed in the error popup. If it says "API key not valid" or similar, the key is incorrect.

### Q. The AI keeps responding with "skip" or "Are you silent?" even though I'm speaking
- **Microphone setting error**: The "Microphone Selection" in TeloPon may have selected a device that picks up game audio or BGM. Please reselect your headset or microphone.
- **Noise threshold adjustment**: The "Threshold" slider may be set too high. Lower the slider to the left so the green bar responds to your voice.

### Q. Nothing is displayed in OBS
- Check that the "Local File" checkbox is unchecked in the OBS browser source properties.
- Try copying the URL again using the "🔗 Copy OBS URL" button and re-pasting it.

### Q. No sound effect (SE) plays when telops appear
- Check in the OBS browser source properties that **"Control audio via OBS" is UNCHECKED**. If checked, the sound plays in the stream but you (the streamer) won't hear it.
- Also check that "Play sound effects" is ON in the app and that the volume slider is not at 0.

---

## ⚠️ Notes on Early Access Version (Terms of Use)

This version is a **beta version under development (early access)**. Please acknowledge the following before using.

1. **Launch restrictions**: This software performs an online version/configuration check at startup. Older versions may be discontinued without notice at the end of the beta test period or with major updates, and a message prompting an update to the latest version may be displayed.
2. **No warranty**: This tool is provided "AS IS". The developer assumes no responsibility for any damages arising from the use of this tool (API usage charges, streaming issues, PC problems, etc.). Use at your own risk.
3. **Prohibition of analysis and redistribution**: To protect the stability and security of the software, reverse engineering, modification, and unauthorized redistribution of the executable (exe) are strictly prohibited. (※ Self-made / modified plugins in the `plugins` folder and custom prompts may be freely created, modified, and shared.)
4. **Nature of AI-generated code**: This tool was entirely developed and code-generated by AI, so unexpected behavior may occur.

## 🤝 Special Thanks & Credits
- **Development Partner:** Google Gemini (Code Generation & Thought Partner)
- **Concept & Vibe:** [![miyumiyuna5](https://img.shields.io/badge/miyumiyuna5-%23000000.svg?style=for-the-badge&logo=X&logoColor=white)](https://x.com/miyumiyuna5)
- **Documentation:** This README was also generated by Gemini.

---
© 2026 TeloPon Project All Rights Reserved.
