# 💬 Discord Real-time Integration (discord_integration.py)

With this plugin, TeloPon's AI will read out and react to comments from a specified channel on your Discord server in real time!

Setup consists of three main steps: **"① Installing the plugin"**, **"② Creating a Bot on Discord (getting the token)"**, and **"③ Configuring in TeloPon"**. Even beginners can complete it in about 5 minutes by following the steps in order!

---

## 📥 Step 1: Install the Plugin

This extension is not included in the TeloPon base package, so first add it to your TeloPon installation.

1. Download the file **`discord_integration.py`** from
   🔗 [TeloPon Official Extension Plugin Pack v1.0 (Discord & Slack)](https://github.com/miyumiyu/TeloPon/releases/tag/plugins-v1.0)
2. Open your `TeloPon-XXX` folder (the main application folder) on your PC.
3. Place the downloaded `discord_integration.py` file directly into the **`plugins`** folder inside it.
4. Launch (or restart) TeloPon. If **"💬 Discord Real-time Integration"** appears in the "🔌 Extensions" list on the main screen, the installation was successful!

---

## 🛠️ Step 2: Create a Discord Bot and Get the "Token"

For the AI to read Discord comments, you need to create a dedicated "Bot (robot)" for yourself.

### 1. Access the Developer Portal
In a PC browser, go to [Discord Developer Portal](https://discord.com/developers/applications) and log in with your usual Discord account.

### 2. Create an Application (Bot)
1. Click the **"New Application"** button in the upper-right.
2. Enter a name you like in `Name` (e.g., `TeloPon Bot`), check the terms checkbox, and press **"Create"**.

### 3. [CRITICAL] Turn on Permission to Read Messages
**※ Without this, the bot cannot read comments! This is mandatory.**
1. Click **"Bot"** in the left menu.
2. Scroll down a bit and find the **"Privileged Gateway Intents"** section.
3. Turn the **"Message Content Intent"** switch **ON (green)**, and if a popup appears, press "Save Changes".

### 4. Copy the Token (Key)
1. Return to the top of the same **"Bot"** screen.
2. Press the **"Reset Token"** button next to the "Token" field and select "Yes, do it!".
3. A long string (this is your token) will appear — press **"Copy"** to copy it.
*(※ This will not be shown again once you close the screen, so paste it somewhere like Notepad.)*

---

## 🖥️ Step 3: Configure and Connect in TeloPon

Open "🔌 Extensions" from the TeloPon main screen and press the Settings (⚙️) button for **"💬 Discord Real-time Integration"** to open the panel.

![Discord integration settings panel](../../../images/slack_integration.png)

### 1. Enter the Token
* Paste the long string (token) you copied in Step 2 into the "Bot Token" input field.

### 2. Invite the Bot to Your Server
* With the token entered, click the **"🌐 Invite this Bot to a server"** button just below.
* A browser will automatically open showing the Discord invitation screen.
* Select the server you want to add the bot to and proceed through "Yes" → "Authorize".
*(※ Success is confirmed when Discord shows "○○ added TeloPon Bot" in the channel.)*

### 3. Fetch the Channel List
* Return to the TeloPon screen and press the **"🔄 Verify Token & Fetch Channel List"** button.
* On success, "✅ Success: Fetched ○ channels!" will appear.

### 4. Select the Channel to Monitor
* Open the "Channel to monitor" dropdown (▼) below — your server's channels (💬 text, 🔊 voice, etc.) will be listed.
* Select the channel you want the AI to read from.

### 5. Connect!
* Finally, press the blue **"Start (Connect)"** button!
* You're ready when the status changes to the green "⚡ Status: Connected".

---

## 💡 Tips and Usage Notes

Once configured, just press "Start Live Connection (Launch AI)" from the TeloPon main screen as usual!
When someone posts in the specified Discord channel, the comment is sent to the AI with about 0.1 seconds of lag, and the AI reacts.

* **Customizing the prompt**: By rewriting the "Instruction text passed to AI" at the bottom of the settings panel, you can change how the AI responds. (e.g., "These are comments from Discord listeners. Please reply in a friendly way!")
* **Bot going offline?**: The Discord bot shows as "Online (green dot)" only while TeloPon's "Connect" button is active. When TeloPon closes, it automatically goes offline.
