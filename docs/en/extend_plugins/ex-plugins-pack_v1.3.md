**English** | [日本語](https://github.com/miyumiyu/TeloPon/blob/main/docs/ja/extend_plugins/ex-plugins-pack_v1.3.md) | [한국어](https://github.com/miyumiyu/TeloPon/blob/main/docs/ko/extend_plugins/ex-plugins-pack_v1.3.md) | [Русский](https://github.com/miyumiyu/TeloPon/blob/main/docs/ru/extend_plugins/ex-plugins-pack_v1.3.md)

The latest official extension plugin pack to make TeloPon even more powerful!
Download only the feature files (`.py`) you need and add them to your TeloPon.
**※ Available from version V2.14b onwards.**

## 📦 Included Plugins

### 🆕 🔊 VOICEVOX TTS (`voicevox_plugin.py`)

![VOICEVOX TTS Settings](../../images/voicevox_plugin.png)

Reads aloud AI-generated telop text in real time using the **VOICEVOX** speech synthesis engine.
* **Wide variety of character voices**: Supports all VOICEVOX characters + styles (personalities). Easy dropdown selection.
* **Icon display**: The selected character's icon is shown in the settings panel.
* **Fine-tuned audio**: 5 parameters — speed, volume, pitch, intonation, and pre-silence — with real-time adjustment.
* **TOPIC readout**: When ON, reads the top line text (TOPIC) along with the main text.
* **Telop display delay**: Sync readout timing with OBS telop appearance.
* **Auto connection check**: Verifies VOICEVOX connection on startup. Automatically turns OFF if unavailable.

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/voicevox_plugin.py)
👉 [Detailed usage guide](../plugins/voicevox_plugin.md)

> 💡 **Requirements**: Install [VOICEVOX](https://voicevox.hiroshiba.jp/) and run it alongside TeloPon.

### 💬 Discord Real-time Integration (`discord_integration.py`)
Fetches comments from a specified Discord server channel in real time and delivers them to the AI.
* **Super easy setup**: Just enter a Bot token and press a button — invite URL is auto-generated.
* **Fully real-time**: Picks up comments instantly with zero lag.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c8a8585c-52ca-43eb-9cdb-fede4373262a" />

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/discord_integration.py)
👉 [Detailed usage guide](../plugins/discord_integration.md)

### 🏢 Slack Comment Integration (`slack_integration.py`)
Fetches comments from a specified Slack workspace channel in real time and delivers them to the AI.
* **Socket Mode support**: Uses the latest communication method trusted by enterprises for zero-delay, fully real-time reception.
* **Automatic name conversion**: Automatically converts Slack's alphanumeric user IDs to display names before delivering to the AI.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/1f228c32-3552-453e-aa29-c824da5b6795" />

📥 [Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/slack_integration.py)
👉 [Detailed usage guide](../plugins/slack_integration.md)

---

## 🛠️ How to Install Plugins

1. Click the feature file (`.py`) you want from the **Assets** section at the bottom of this page to download it.
2. Open the `TeloPon-XXX` folder on your computer.
3. Drop the downloaded `.py` file into the **`plugins`** folder inside it.
4. Launch TeloPon — the new feature will appear in the "🔌 Extensions" list on the right!

※ For detailed settings, refer to each plugin's settings screen (⚙️) or the official manual.

---

## 📋 Release Title (for GitHub Releases)

```
🔌 TeloPon Official Extension Plugin Pack v1.3 (VOICEVOX, Discord & Slack)
```
