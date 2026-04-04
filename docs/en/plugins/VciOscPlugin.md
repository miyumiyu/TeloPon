# 🎮 VCI OSC Telop (VciOscPlugin.py)

> 📥 **[Download Plugin](https://github.com/miyumiyu/TeloPon/releases/download/plugins-v1.3/VciOscPlugin.py)**

This plugin sends AI-generated telop text to **VirtualCast VCI** in real time via **OSC (OpenSound Control)** protocol.
It enables telop display inside VirtualCast's virtual space.

---

## 🛠️ Preparation

### Requirements

1. **VirtualCast** (PC version) installed and running
2. VirtualCast **OSC Receive Function** enabled (port: 19100)
3. A telop display **VCI** placed in VirtualCast

### VirtualCast OSC Setup

1. Open VirtualCast title screen → **"VCI"** settings
2. Set **"OSC Receive Function"** to **"Enabled"** or **"Creator Only"**
3. Confirm receive port is **19100** (default)

### VCI Receive Script (Lua)

Set up a Lua script in your telop display VCI:

```lua
-- Receive telop from TeloPon
vci.osc.RegisterMethod("/telopon/telop/text", function(sender, name, args)
    local text = args[1]
    vci.assets.SetText("TextBoard", text)
end, {ExportOscType.String})

-- Receive badge (optional)
vci.osc.RegisterMethod("/telopon/telop/text/badge", function(sender, name, args)
    local badge = args[1]
end, {ExportOscType.String})

-- Receive window type (optional)
vci.osc.RegisterMethod("/telopon/telop/text/window", function(sender, name, args)
    local window = args[1]
end, {ExportOscType.String})
```

---

## ⚙️ TeloPon Configuration

### 1. Open the Control Panel

Click **"Control Panel"** next to **"VCI OSC Telop"** in the Extensions panel.

### 2. Enable OSC Send

Check **"OSC Send ON"** to automatically send telop data to VCI.

### 3. Connection Settings

| Setting | Default | Description |
|---|---|---|
| **Target IP** | `127.0.0.1` | IP of the PC running VirtualCast |
| **Target Port** | `19100` | VirtualCast OSC receive port |
| **OSC Address** | `/telopon/telop/text` | Must match VCI's `RegisterMethod` |

### 4. Test Send

Press **"Test Send"** to verify the connection.

### 5. Content Settings

| Setting | Default | Description |
|---|---|---|
| **📌 Send TOPIC** | ON | Sends "TOPIC \| MAIN" format. OFF sends MAIN only |
| **🏷️ Send Badge** | OFF | Sends badge name to `/address/badge` |

### 6. Close

Settings are saved to `plugins/vci_osc.json` automatically.

---

## 📡 OSC Messages Sent

| OSC Address | Type | Content | Example |
|---|---|---|---|
| `/telopon/telop/text` | String | Telop text | `Stream Start \| Here comes TeloPon!` |
| `/telopon/telop/text/badge` | String | Badge name (optional) | `Surprised` |
| `/telopon/telop/text/window` | String | Window type (always) | `window-simple` |

---

## ❓ Troubleshooting

### Q. Test send shows "❌ Send failed"
- Check that the IP and port are correct
- Check firewall settings for UDP port 19100

### Q. No text in VCI
- Ensure VirtualCast OSC receive is enabled
- Ensure OSC addresses match between TeloPon and VCI
- Ensure the VCI is placed in VirtualCast

---
