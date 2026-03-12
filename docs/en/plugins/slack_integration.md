# 🏢 Slack Comment Integration (slack_integration.py)

With this plugin, TeloPon's AI reads out comments posted to your company Slack workspace or team channels with zero latency!
It uses the latest "Socket Mode" technology, enabling remarkably real-time reactions.

Setup consists of three main steps: **"① Installing the plugin"**, **"② Creating a Slack Bot (getting two tokens)"**, and **"③ Configuring in TeloPon"**. There are quite a few steps, but if you follow them in order you can definitely get it set up!

---

## 📥 Step 1: Install the Plugin and Prepare

First, add the extension to your TeloPon installation.

1. Download **`slack_integration.py`** from
   🔗 [TeloPon Official Extension Plugin Pack v1.0 (Discord & Slack)](https://github.com/miyumiyu/TeloPon/releases/tag/plugins-v1.0)
2. Place the downloaded `slack_integration.py` file into the **`plugins`** folder inside your TeloPon main folder.
3. Launch TeloPon — if **"🏢 Slack Comment Integration"** appears in the "🔌 Extensions" list on the main screen, you're done!

---

## 🛠️ Step 2: Create a Slack Bot and Get "Two Tokens"

To allow the AI to read Slack comments, create a dedicated "Bot (app)". Here you'll obtain **2 types of tokens (keys)**.

### 1. Create the App and Give It a Name
1. In a PC browser, go to [Slack API (Your Apps)](https://api.slack.com/apps) and press **"Create New App"** in the upper-right.
2. Select **"From scratch"**, enter a name you like (e.g., `TeloPon Bot`), select the workspace to install it in, and press **"Create App"**.
3. **[Important]** Open **"App Home"** in the left menu, then scroll down to **"Your App's Presence in Slack"** and press the "Edit" button to set a `Display Name` (shown name) and `Default Username` (internal ID, lowercase letters and numbers only), then save. Without this step, you won't be able to invite the bot!

### 2. Enable Socket Mode and Get the "App-Level Token"!
1. Open **"Socket Mode"** from the left menu.
2. Turn the **"Enable Socket Mode"** switch **ON (green)**.
3. When asked for a token name, enter anything (e.g., `telopon-socket`) and press "Generate".
4. A string starting with **`xapp-`** will appear. This is the **[App-Level Token]**. Copy and note it down.

### 3. Grant Permissions to Read Messages (Scopes)
1. Open **"OAuth & Permissions"** from the left menu.
2. Scroll down, and under **"Bot Token Scopes"** in the **"Scopes"** section, press "Add an OAuth Scope" and add the following 5 items:
   * `channels:history` / `groups:history` / `channels:read` / `groups:read` / `users:read`

### 4. Configure Real-time Receiving (Event Subscriptions)
1. Open **"Event Subscriptions"** from the left menu.
2. Turn **"Enable Events"** at the top **ON (green)**.
3. Open **"Subscribe to bot events"** just below, add the following 2 events via "Add Bot User Event", and press **"Save Changes"** in the lower-right:
   * `message.channels` / `message.groups`

### 5. Install to the Workspace and Get the "Bot Token"!
1. Open **"Install App"** from the left menu and press **"Install to Workspace"**. (※ If permissions have changed, it will say "Reinstall".)
2. An authorization screen will appear — press "Allow".
3. A string starting with **`xoxb-`** will appear. This is the **[Bot Token]**. Copy and note it down.

### 6. [CRITICAL] "Invite" the Bot to a Slack Channel
In Slack, **mention the bot in the channel you want to monitor** (e.g., type `@TeloPon Bot` in `#general`). When asked "Add to channel?", make sure to add it. Without this step, it cannot read comments!

---

## 🖥️ Step 3: Configure and Connect in TeloPon

Return to the TeloPon screen and open the Settings (⚙️) panel for "🏢 Slack Comment Integration".

![Slack integration settings panel](../../../images/discord_integration.png)

1. **Enter the Tokens**
   * Paste the **`xapp-`** string obtained in Step 2 into the "App-Level Token" field.
   * Paste the **`xoxb-`** string obtained in Step 2 into the "Bot Token" field.
2. **Fetch the Channel List**
   * Press the **"🔄 Verify Tokens & Fetch Channel List"** button.
   * On success, the channels in your Slack workspace will be listed.
3. **Select the Channel to Monitor**
   * Choose the channel to monitor from the dropdown. (Public channels show 💬, private channels show 🔒.)
4. **Connect!**
   * Finally, press the blue **"Connect"** button — you're ready when the status turns green: "⚡ Status: Connected (Socket Mode active)"!

---

## 💡 Tips and Usage Notes

Once configured, just press "Start Live Connection (Launch AI)" from the TeloPon main screen! The AI will quickly react to messages in the specified Slack channel.

* **Automatic name conversion**: Slack's unique user IDs (U1234..., etc.) are automatically converted to the person's display name (e.g., John Smith) on TeloPon's backend before being sent to the AI. The AI will properly address people by name in responses!
* **Customizing the prompt**: Changing the instruction text at the bottom of the settings panel to something like "These are chats from internal team members! Brighten everyone's day!" can bring out even more of the AI's personality.
