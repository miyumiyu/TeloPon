**English** | [日本語](https://github.com/miyumiyu/TeloPon/blob/main/docs/ja/extend_plugins/ex-plugins-pack_v1.4.md) | [한국어](https://github.com/miyumiyu/TeloPon/blob/main/docs/ko/extend_plugins/ex-plugins-pack_v1.4.md) | [Русский](https://github.com/miyumiyu/TeloPon/blob/main/docs/ru/extend_plugins/ex-plugins-pack_v1.4.md)

The latest official extension plugin pack to make TeloPon even more powerful!
Download only the feature files (`.py`) you need and add them to your TeloPon.
**※ Available from version V2.15b onwards.**

## 📦 Included Plugins

### 🆕 🎮 VCI OSC Telop (`VciOscPlugin.py`)

Sends AI-generated telop text to **VirtualCast VCI** in real time via **OSC (OpenSound Control)** protocol.
* **VirtualCast integration**: Display telops inside the virtual space
* **Configurable target**: IP, port, and OSC address are customizable
* **TOPIC & Badge**: Sends topic and badge (emotion tag) on separate OSC addresses
* **Test send**: One-click connection verification

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/VciOscPlugin.py)
👉 [Detailed usage guide](../plugins/VciOscPlugin.md)

> 💡 **Requirements**: Enable OSC receive in [VirtualCast](https://virtualcast.jp/) (PC version).

### 🔊 VOICEVOX TTS (`voicevox_plugin.py`)

<img width="400" alt="VOICEVOX TTS Settings" src="https://github.com/miyumiyu/TeloPon/blob/main/images/voicevox_plugin.png?raw=true" />

Reads aloud AI-generated telop text in real time using the **VOICEVOX** speech synthesis engine.
* **Wide variety of character voices**: All VOICEVOX characters + styles. Easy dropdown selection.
* **Icon display**: Selected character's icon shown in settings panel.
* **Fine-tuned audio**: 5 parameters with real-time adjustment.
* **TOPIC readout**: Optionally reads TOPIC along with main text.
* **Telop display delay**: Sync readout with OBS telop appearance.
* **Auto connection check**: Automatically turns OFF if VOICEVOX is unavailable.

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/voicevox_plugin.py)
👉 [Detailed usage guide](../plugins/voicevox_plugin.md)

> 💡 **Requirements**: Install [VOICEVOX](https://voicevox.hiroshiba.jp/) and run alongside TeloPon.

### 💬 Discord Real-time Integration (`discord_integration.py`)
Fetches comments from a specified Discord server channel in real time and delivers them to the AI.
* **Super easy setup**: Just enter a Bot token and press a button — invite URL is auto-generated.
* **Fully real-time**: Picks up comments instantly with zero lag.
* **Avatar support**: Commenter icons displayed on reply telops.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/discord_integration.py)
👉 [Detailed usage guide](../plugins/discord_integration.md)

### 🏢 Slack Comment Integration (`slack_integration.py`)
Fetches comments from a specified Slack workspace channel in real time and delivers them to the AI.
* **Socket Mode support**: Zero-delay, fully real-time reception.
* **Automatic name conversion**: Converts Slack user IDs to display names.
* **Avatar support**: Commenter icons displayed on reply telops.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.4/slack_integration.py)
👉 [Detailed usage guide](../plugins/slack_integration.md)

---

## 🛠️ How to Install Plugins

1. Click the feature file (`.py`) you want from the **Assets** section at the bottom of this page.
2. Open the `TeloPon-XXX` folder on your computer.
3. Drop the downloaded `.py` file into the **`plugins`** folder.
4. Launch TeloPon — the new feature will appear in the "🔌 Extensions" list!

---

## 📋 Release Title (for GitHub Releases)

```
🔌 TeloPon Official Extension Plugin Pack v1.4 (VCI OSC, VOICEVOX, Discord & Slack)
```
