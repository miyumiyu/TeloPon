# 🔊 VOICEVOX TTS (voicevox_plugin.py)

This plugin reads aloud the text of AI-generated telops using the **VOICEVOX** speech synthesis engine.
With a wide variety of character voices and fine-tuned audio parameters, it brings your stream to life.

---

## 🛠️ Preparation: Installing and Starting VOICEVOX

You need to install and run VOICEVOX separately from TeloPon.

1. Download and install VOICEVOX from the **[VOICEVOX official website](https://voicevox.hiroshiba.jp/)**.
2. **Launch VOICEVOX**. An internal HTTP server starts automatically (default: `http://localhost:50021`).
3. Keep VOICEVOX **running alongside TeloPon**.

> 💡 You can minimize the VOICEVOX window. It runs in the background.

---

## ⚙️ TeloPon Configuration and Usage

### 1. Open the Control Panel

Click the **"Control Panel"** button next to **"VOICEVOX TTS"** in the Extensions panel on the right side of TeloPon.

![VOICEVOX TTS Settings](../../../images/voicevox_plugin.png)

---

### 2. Connection Test

1. Enter the VOICEVOX address in **URL** (default: `http://localhost:50021`)
2. Press the **"Test Connection"** button
3. If "✅ Connection successful" appears, you're ready

> ⚠️ If the connection fails, make sure VOICEVOX is running. Voice settings will be grayed out until a connection is established.

### 3. Enable TTS

Check **"TTS ON"** to automatically read aloud each telop as it appears.

### 4. Character and Style Selection

After a successful connection test, the character list from VOICEVOX appears in the dropdown.

* **Character**: Choose the voice character (e.g., Shikoku Metan, Zundamon, etc.)
* **Style**: Each character has multiple styles (e.g., Normal, Sweet, Tsun-tsun, etc.)

An icon image of the selected character/style is displayed on the left.

### 5. Audio Parameter Adjustment

| Parameter | Range | Default | Description |
|---|---|---|---|
| **Speed** | 0.5 – 2.0 | 1.0 | Reading speed. Higher = faster |
| **Volume** | 0.1 – 2.0 | 1.0 | Reading volume |
| **Pitch** | -0.15 – 0.15 | 0.0 | Voice pitch. + = higher, - = lower |
| **Intonation** | 0.0 – 2.0 | 1.0 | Emotional intensity. 0 = monotone, higher = more expressive |
| **Pre-silence** | 0.0 – 1.5s | 0.1 | Silence before speech starts (seconds) |

> 💡 Parameters take effect immediately on the next telop readout.

### 6. Telop Settings

* **📌 Read TOPIC aloud**: When ON, reads "TOPIC. MAIN" together. When OFF, reads MAIN only.
* **📺 Show Telop**: When ON, telop is displayed in OBS (delay can be configured). When OFF, telop display is suppressed.
* **Telop Delay (sec)**: Delays telop display in OBS (0 = instant). Useful for syncing readout with telop appearance.

### 7. Close

Press the **"Close"** button or the window **×** button to close the settings panel. Settings are automatically saved to `plugins/voicevox.json` and restored on next launch.

---

## ❓ Troubleshooting

### Q. Connection test shows "❌ Connection failed"
- Make sure VOICEVOX is running
- Check that the URL is `http://localhost:50021`
- Check if security software is blocking port 50021

### Q. Readout suddenly stopped
- VOICEVOX may have been closed. If the connection is lost during readout, TTS is automatically turned OFF
- Restart VOICEVOX and use "Test Connection" in the control panel to reconnect

### Q. No sound
- Check that your PC volume is not muted
- Make sure "TTS ON" checkbox is checked
- Make sure the volume parameter is 0.1 or higher

---
