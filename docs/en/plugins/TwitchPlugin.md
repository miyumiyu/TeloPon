# 🎮 Twitch Live Plugin (TwitchPlugin.py)

A Twitch integration plugin.  
The AI automatically performs various Twitch operations based on the streamer's voice commands.

<img src="../../../images/TwitchPlugin.png" width="600">

---

## 🌟 Key Features

| Feature | Description | Account Restriction |
|---|---|---|
| Chat comment reading | Injects viewer comments to AI in real time | None |
| Chat comment writing | AI posts to Twitch chat | None |
| Poll creation | Create and tally viewer polls | **Affiliate/Partner only** |
| Prediction | Viewers bet channel points | **Affiliate/Partner only** |
| Title change | Change stream title by voice | None |
| Clip creation | Clip the last ~30 seconds | None |
| Viewer count | Get concurrent viewer count | None |
| Bot exclusion | Exclude specific users from AI injection | - |

---

## 📋 Features by Account Type

| Feature | Regular | Affiliate | Partner |
|---|---|---|---|
| Read comments | ✅ | ✅ | ✅ |
| Write comments | ✅ | ✅ | ✅ |
| Title change | ✅ | ✅ | ✅ |
| Clip creation | ✅ | ✅ | ✅ |
| Viewer count | ✅ | ✅ | ✅ |
| Polls | ❌ | ✅ | ✅ |
| Predictions | ❌ | ✅ | ✅ |

> Account type is automatically detected during authentication and displayed as `[Standard]`, `[Affiliate]`, or `[Partner]`.  
> Unavailable features are automatically unchecked and grayed out.

---

## ⚙️ Setup Guide

### Step 1: Choose Connection Mode

#### IRC Mode (any stream, chat read only)

**No authentication required.** Reads comments from any Twitch channel.

> Skip Steps 2-3 and go directly to Step 4.

#### Auth Mode (your stream, all features)

**OAuth2 authentication required.** Connects to your own channel with full features.  
Follow Steps 2-3 below.

---

### Step 2: Register App on Twitch Developer Console

1. Log in to [Twitch Developer Console](https://dev.twitch.tv/console/apps)
2. Click "Register Your Application"
3. Fill in:

| Field | Example | Description |
|---|---|---|
| Name | `TeloPon` | Any name |
| OAuth Redirect URL | `https://localhost` | Not used but required |
| Category | `Application Integration` | Any |

4. After creation, copy the **Client ID**

5. **Client Secret (recommended):**  
   Click "New Secret" to generate and copy it.

   > ⚠️ **Client Secret is not required**, but without it the access token expires after ~4 hours and you'll need to re-authenticate via browser each time.

### Step 3: Authenticate in TeloPon

1. Open TeloPon → Extensions → "Twitch Live Plugin" gear icon
2. Enter **Client ID** and **Client Secret** (optional)
3. Click **"Sign in with Twitch"**
4. A browser opens with the authentication URL (copy button available)
5. Log in and authorize on Twitch
6. On success, your username and account type are displayed

> To cancel during authentication, click "Cancel auth".

---

### Step 4: Connect and Start Live

1. **Auth mode:** Click "Connect". Stream info (title/category/thumbnail) will be displayed.
2. **IRC mode:** Enter channel name and click "Connect".
3. Press **"🔴 Start Live"** on TeloPon's main screen.

> **Important:** Press "Connect" **before** starting the live broadcast.

---

## 🎙️ Voice Command Examples

| Streamer says | AI action |
|---|---|
| "Write 'hello' in chat" | Posts "hello" to Twitch chat |
| "Create a poll about favorite colors" | Creates a poll with color options for 60 seconds |
| "Tally the results" | Closes the poll and announces results |
| "Let viewers bet on whether I can beat this boss" | Creates a prediction with options |
| "The answer is number 1!" | Resolves the prediction |
| "Change the title to 'Chill stream'" | Changes the stream title |
| "Clip that!" | Creates a clip of the last ~30 seconds |
| "How many viewers?" | Gets and announces the viewer count |

---

## 🛡️ Viewer Permission Settings

Each feature has a "Viewer" checkbox to control whether viewer comments can trigger commands.

---

## 👤 User Management (Bot Exclusion)

The settings panel includes a user management section.  
**Double-click** recent users to add to the exclusion list.  
**Double-click** excluded users to remove from the list.

- The authenticated user is added to the exclusion list by default (can be removed)
- Useful for excluding bots like `nightbot`, `streamelements`

---

[⬅️ Back to plugin list](../../../README_en.md#-official-extension-plugins-individual-download)
