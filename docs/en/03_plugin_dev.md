# TeloPon: Complete Plugin Development Guide 🧩

TeloPon plugins come in **2 main types**.

| Type | Description | Registration |
|------|------|----------|
| **UI Panel Plugin** | Displayed in TeloPon's "Extensions" screen. Provides AI data injection and settings panels. | Auto-registered by placing the file in the `plugins/` folder |
| **CMD Command Plugin** | Handles processing called when the AI says `[CMD:XXX]` in its output. | Auto-registered by placing the file in the `plugins/` folder |

Both types are created by inheriting from the same `BasePlugin` class.
You can also **combine both functions into a single class**.

---

## 1. The Common Foundation: BasePlugin Class

All plugins are created by inheriting from `BasePlugin` in `plugin_manager.py`.

```python
from plugin_manager import BasePlugin
```

`BasePlugin` has **two sets of fields and methods** — one for UI panels and one for CMD commands.
You only need to configure and implement the ones you want to use.

### Class Variables

```python
class BasePlugin:
    # ===== For UI Panel Plugins =====
    PLUGIN_ID   = ""           # ← If left empty, not registered as a UI panel
    PLUGIN_NAME = "Base Plugin"
    PLUGIN_TYPE = "BACKGROUND" # "BACKGROUND" / "TOOL"

    # ===== For CMD Command Plugins =====
    IDENTIFIER      = ""       # ← The XXX in [CMD:XXX]. If empty, not registered as CMD
    REQUIRED_CONFIG: list = [] # List of config key names to pre-check in setup()
```

Setting `PLUGIN_ID` → functions as a UI Panel Plugin
Setting `IDENTIFIER` → functions as a CMD Command Plugin
**Setting both → becomes a hybrid plugin with both functions**

---

## 2. The Two PLUGIN_TYPEs

When creating a UI Panel Plugin, set `PLUGIN_TYPE` to one of the following:

| PLUGIN_TYPE | Description |
|------------|------|
| `"BACKGROUND"` | Settings screen is locked during streaming. For systems that run automatically in the background. |
| `"TOOL"` | The control panel can be opened and operated at any time, even during streaming. For manual tools. |

---

## 3. Badge Display Mechanism

Each plugin in the Extensions panel displays a **TOOL / ON / OFF** badge.
The badge color (active = blue/green, or gray) is determined by the following priority order:

```
Priority 1: is_connected attribute is True  → Active (blue/green)
Priority 2: is_running   attribute is True  → Active (blue/green)
Fallback:   Neither attribute exists        → Uses the enabled setting value
```

### All State Combinations

| `is_connected` | `is_running` | `enabled` | Badge |
|---|---|---|---|
| True (defined) | — | — | Active (blue/green) |
| False (defined) | — | — | Gray |
| Undefined | True | — | Active (blue/green) |
| Undefined | False | — | Gray |
| Undefined | Undefined | True | Active (blue/green) |
| Undefined | Undefined | False | Gray |

> **`is_running` responds to live session events.**
> When TeloPon connects or disconnects from Gemini Live, it calls `start()` / `stop()` on all plugins.
> Setting `self.is_running = True` inside `start()` and `False` inside `stop()` is the typical usage.

### Behavior by Pattern (quick reference)

| Plugin Implementation | When Badge Becomes Active | Typical Use Case |
|---|---|---|
| `is_connected` defined, fixed True | Always active | TOOL plugins with no external connection but always-on badge |
| `is_connected` defined, variable | When `is_connected = True` | External service connections like YouTube, Twitch |
| `is_running` defined (no `is_connected`) | When `is_running = True` | Background processing plugins |
| Neither attribute defined | When `enabled = True` | Simple custom plugins |

### Important: Whether to Define `is_connected`

**Plugins that connect to external services (YouTube, Twitch, etc.)** should define `is_connected` and
set it to `True` only when the connection is established. It is incorrect for the badge to light up when not connected.

```python
# External service connection type: controlled by is_connected
class YouTubePlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self.is_connected = False  # Set to True when connection is established
        self.is_running   = False  # Set to True when live starts
```

**Simple custom plugins** should avoid defining `is_connected` / `is_running` entirely,
so that the badge becomes active with `enabled = True` alone.

```python
# Simple custom plugin: controlled by enabled (do not define is_connected)
class MySimplePlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"
    PLUGIN_TYPE = "TOOL"

    def get_default_settings(self):
        return {"enabled": True}  # Badge lights up before the stream starts

    # Do not define is_connected / is_running  ← Key point
```

---

## 4. How to Create a UI Panel Plugin

### Where to Place the File

```
TeloPon/
 └── plugins/
      └── my_plugin.py   ← Just placing it here automatically displays it in the UI
```

If `PLUGIN_ID` is set, the `plugins/` folder is automatically scanned and registered at app startup.

### Minimal Example

```python
import tkinter as tk
from tkinter import ttk
from plugin_manager import BasePlugin
import logger

class MyPlugin(BasePlugin):
    PLUGIN_ID   = "my_plugin"      # ← Unique ID (matching the filename is a good practice)
    PLUGIN_NAME = "🔧 Sample Plugin"
    PLUGIN_TYPE = "BACKGROUND"

    def get_default_settings(self):
        """Returns the default values for settings"""
        return {"enabled": False, "message": "Hello"}

    def open_settings_ui(self, parent_window):
        """Window that opens when the ⚙️ Settings button is pressed"""
        panel = tk.Toplevel(parent_window)
        panel.title(self.PLUGIN_NAME)
        panel.geometry("300x150")
        panel.attributes("-topmost", True)

        settings = self.get_settings()

        ttk.Label(panel, text="Message:").pack(pady=5)
        ent = ttk.Entry(panel, width=30)
        ent.insert(0, settings.get("message", ""))
        ent.pack()

        def save():
            s = self.get_settings()
            s["message"] = ent.get()
            self.save_settings(s)
            panel.destroy()

        ttk.Button(panel, text="Save", command=save).pack(pady=10)

    def start(self, prompt_config, plugin_queue):
        """▶️ Called when the live stream starts"""
        logger.info(f"[{self.PLUGIN_NAME}] Started")

    def stop(self):
        """⏹️ Called when the live stream stops"""
        logger.info(f"[{self.PLUGIN_NAME}] Stopped")
```

### About Settings Files

Settings are automatically saved to `plugins/my_plugin.json`. You do not need to manipulate files manually.

```python
# Read (values from get_default_settings are used as the base)
settings = self.get_settings()
value = settings.get("my_key", "default value")

# Save
settings = self.get_settings()
settings["my_key"] = "new value"
self.save_settings(settings)
```

---

## 5. Injecting Information into the AI (The Main Role of UI Panel Plugins)

### Method ① Appending to the Prompt Before the Stream Starts

```python
def get_prompt_addon(self):
    """Called just before the stream starts. Returns a string to append to the AI's system prompt."""
    settings = self.get_settings()
    if not settings.get("enabled"):
        return ""
    return """
    [Advance Notice]
    Today's stream is a celebration of reaching 100 viewers.
    The AI should include congratulatory comments at the beginning, taking this into account.
    """
```

### Method ② Sending Text to the AI in Real Time During the Stream

```python
def start(self, prompt_config, plugin_queue):
    self.plugin_queue = plugin_queue  # Store the queue
    # ... start threads etc.

def _some_event_happened(self):
    # Inject text to the AI (using the dedicated method is convenient)
    self.send_text(self.plugin_queue, "[Notification] Viewer count increased! Please acknowledge this.")
    # Or put directly — same result
    # self.plugin_queue.put({"type": "text", "content": "..."})
```

### Method ③ Sending Images to the AI in Real Time During the Stream

```python
import io
from PIL import Image

def _send_screenshot(self):
    img = Image.open("screenshot.png")
    if img.mode != "RGB":
        img = img.convert("RGB")
    img.thumbnail((1024, 1024))

    buf = io.BytesIO()
    img.save(buf, format="JPEG", quality=85)

    # Using the dedicated method is convenient (mime_type defaults to "image/jpeg" if omitted)
    self.send_image(self.plugin_queue, buf.getvalue())
    # Then send instructions about what the image is — this is the key!
    self.send_text(self.plugin_queue, "Please comment on the image you just saw.")
```

---

## 6. Telop Output Hook (Receiving AI Output in Plugins)

Override the `on_telop_output` method to receive data every time the AI outputs a telop.
Telop output is delivered via a **fire-and-forget** mechanism -- Core does not wait for plugin processing to complete.

### Architecture (Decoupled Design)

```
Core (on telop output):
  ① on_telop_output() dispatched to all plugins asynchronously (fire-and-forget)
  ② Reads the delay value previously set via set_telop_delay()
  ③ Controls OBS telop display (instant / delay / suppress)
```

**Important**: Core never waits for `on_telop_output()` to complete. Whatever happens inside a plugin (errors, slowness, freezes, etc.) has no impact on Core.

### Two Functions Available to Plugins

| Method | Purpose | When to Call |
|---|---|---|
| `on_telop_output(topic, main, window, layout, badge)` | Receive telop data | Called automatically by Core (no return value needed) |
| `set_telop_delay(n)` | Set OBS display delay | Called by the plugin at any time |

### Delay Control (set_telop_delay)

```python
self.set_telop_delay(0)   # Instant display (default)
self.set_telop_delay(5)   # Display after 5 seconds
self.set_telop_delay(-1)  # Suppress OBS display
```

Core reads `get_telop_delay()` from all plugins before enqueuing and aggregates the values.
**Multiple plugin conflict rule**: `-1` (suppress) has highest priority. Otherwise `max(delay)` is used.

### Data Received

```python
def on_telop_output(self, topic, main, window, layout, badge):
    # Process the data (no return value needed)
    pass
```

| Argument | Content | Example |
|---|---|---|
| `topic` | Top line text | `To Sheep` |
| `main` | Body text | `What an <b1>observant</b1> eye!` |
| `window` | Window type | `window-reply`, `window-simple`, `window-explain` |
| `layout` | Layout | `layout-flat`, `layout-big-top` |
| `badge` | Badge (author name etc.) | `@sheep-x9y`, `NONE` |

**Tags in `main` / `topic`** (raw data before `drawing.apply_semantic_classes`):

| Tag | Meaning | Example |
|---|---|---|
| `<b1>...</b1>` | Highlight color 1 (theme h1 color) | `<b1>important</b1>` |
| `<b2>...</b2>` | Highlight color 2 (theme h2 color) | `<b2>notice</b2>` |
| `<P1>` `<M5>` etc. | Mahjong tiles | `<P1><P1><P1>` |

### Example: Monitor Plugin (No Display Control)

```python
def on_telop_output(self, topic, main, window, layout, badge):
    # Simply log the telop content
    logger.info(f"[Monitor] {window} | {topic} | {main}")
```

### Example: Setting a Delay

```python
def __init__(self):
    super().__init__()
    self.set_telop_delay(5)  # Initial value: 5-second delay

def on_telop_output(self, topic, main, window, layout, badge):
    # Receive data (delay was already configured via set_telop_delay)
    self._log_telop(topic, main)

def _on_delay_changed(self, new_value):
    # Called when the delay value is changed from the UI
    self.set_telop_delay(new_value)
```

### Example: Conditional Suppression

```python
def __init__(self):
    super().__init__()
    self._suppress_reply = False

def on_telop_output(self, topic, main, window, layout, badge):
    # Always receive the data
    self._process(topic, main)

def toggle_suppress(self):
    # Called from a UI checkbox
    self._suppress_reply = not self._suppress_reply
    self.set_telop_delay(-1 if self._suppress_reply else 0)
```

### ⚠️ Important: Thread Safety

`on_telop_output` is called from a **background thread** (executed asynchronously in a daemon thread).
The following restrictions apply.

**🚫 Forbidden: Tkinter operations**

Calling Tkinter methods directly inside `on_telop_output` causes **deadlocks**:

```python
# ❌ Do NOT call these inside on_telop_output
self._panel.after(0, ...)        # Deadlock
self._panel.winfo_exists()       # Deadlock
self._text_widget.insert(...)    # Deadlock
```

**✅ Correct pattern: deque buffer + polling**

Push data to a thread-safe `deque`, then consume it from a main-thread timer.

```python
from collections import deque

class MyPlugin(BasePlugin):
    def __init__(self):
        super().__init__()
        self._log_buffer = deque(maxlen=500)  # Thread-safe

    def on_telop_output(self, topic, main, window, layout, badge):
        # ✅ Only deque operations (no Tkinter)
        self._log_buffer.append((topic, main, window))

    def open_settings_ui(self, parent_window):
        self._panel = tk.Toplevel(parent_window)
        # ... build UI ...
        self._poll()  # Start polling

    def _poll(self):
        # ✅ Consume deque and update UI on main thread
        while self._log_buffer:
            topic, main, window = self._log_buffer.popleft()
            self._text_widget.insert("end", f"{topic} | {main}\n")
        if self._panel and self._panel.winfo_exists():
            self._panel.after(100, self._poll)
```

**Safe operations inside `on_telop_output`:**
- Read/write variables (`self._delay_value`, etc.)
- `deque.append()` / `list.append()` / `queue.put()`
- File I/O
- Network calls
- Heavy/long-running operations (they do not affect Core)

### ALWAYS_ACTIVE Attribute

Set `ALWAYS_ACTIVE = True` to display the plugin in active color (black text, bold) in the plugin list at all times. The `on_telop_output` hook is called for all plugins regardless of `enabled` state, making this ideal for monitor-type plugins that work just by opening the panel.

```python
class MyViewerPlugin(BasePlugin):
    PLUGIN_ID   = "my_viewer"
    PLUGIN_TYPE = "TOOL"
    ALWAYS_ACTIVE = True  # Always shown as active
```

---

## 7. Full Template for a BACKGROUND Plugin

A complete template for a plugin that runs continuously in the background and periodically sends information to the AI.

```python
import time
import threading
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class AutoTimerPlugin(BasePlugin):
    PLUGIN_ID   = "auto_timer_plugin"
    PLUGIN_NAME = "⌚ Auto Timer Plugin"
    PLUGIN_TYPE = "BACKGROUND"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_running   = False
        self.is_connected = False  # Connection state flag synced with UI
        self.thread       = None

    def get_default_settings(self):
        return {"enabled": False, "interval_sec": 60}

    # ==========================================
    # Settings UI
    # ==========================================
    def open_settings_ui(self, parent_window):
        # If already open, bring to front
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(f"{self.PLUGIN_NAME} Settings")
        self.panel.geometry("350x220")
        self.panel.attributes("-topmost", True)

        settings = self.get_settings()
        self.is_connected = settings.get("enabled", False)

        main_f = ttk.Frame(self.panel, padding=15)
        main_f.pack(fill=tk.BOTH, expand=True)

        # Status label
        self.lbl_status = ttk.Label(main_f, text="", font=("", 11, "bold"))
        self.lbl_status.pack(anchor="w", pady=(0, 10))

        # Settings input
        row = ttk.Frame(main_f)
        row.pack(fill=tk.X, pady=5)
        ttk.Label(row, text="Notification interval (seconds):").pack(side="left")
        self.ent_interval = ttk.Entry(row, width=8)
        self.ent_interval.pack(side="left", padx=5)
        self.ent_interval.insert(0, str(settings.get("interval_sec", 60)))

        # Connect/Disconnect button
        self.btn_connect = tk.Button(
            main_f, font=("", 10, "bold"),
            command=self._toggle_connection
        )
        self.btn_connect.pack(fill=tk.X, pady=8)

        # Save and close button
        tk.Button(
            main_f, text="Save and Close",
            bg="#6c757d", fg="white",
            command=self._save_and_close
        ).pack(fill=tk.X)

        self._update_ui()  # Apply initial UI state

    def _update_ui(self):
        """Switch UI appearance based on connection state"""
        if not hasattr(self, "btn_connect") or not self.btn_connect.winfo_exists():
            return
        if self.is_connected:
            self.lbl_status.config(text="⚡ Running", foreground="#28a745")
            self.btn_connect.config(text="Stop", bg="#dc3545", fg="white")
            self.ent_interval.config(state="disabled")  # Lock while running
        else:
            self.lbl_status.config(text="🔌 Stopped", foreground="gray")
            self.btn_connect.config(text="Start", bg="#007bff", fg="white")
            self.ent_interval.config(state="normal")

    def _toggle_connection(self):
        if self.is_connected:
            self.is_connected = False
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] Stopped")
        else:
            try:
                int(self.ent_interval.get())
            except ValueError:
                messagebox.showwarning("Error", "Please enter a number for the seconds value")
                return
            self.is_connected = True
            self._update_ui()
            self._save_from_ui()
            logger.info(f"[{self.PLUGIN_NAME}] Enabled")

    def _save_from_ui(self):
        s = self.get_settings()       # ← Load current settings first
        s["enabled"]      = self.is_connected
        s["interval_sec"] = int(self.ent_interval.get())
        self.save_settings(s)

    def _save_and_close(self):
        self._save_from_ui()
        self.panel.destroy()

    # ==========================================
    # Lifecycle (live stream start/stop)
    # ==========================================
    def start(self, prompt_config, plugin_queue):
        settings = self.get_settings()
        if not settings.get("enabled"):
            return
        if self.is_running:
            return

        self.is_running   = True
        self.plugin_queue = plugin_queue
        self.thread = threading.Thread(
            target=self._loop,
            args=(prompt_config,),
            daemon=True
        )
        self.thread.start()
        logger.info(f"[{self.PLUGIN_NAME}] Background loop started")

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None

    # ==========================================
    # Background processing (runs in a separate thread)
    # ==========================================
    def _loop(self, prompt_config):
        settings     = self.get_settings()
        interval     = int(settings.get("interval_sec", 60))
        ai_name      = prompt_config.get("ai_name", "AI")
        elapsed      = 0

        while self.is_running:
            time.sleep(1.0)  # ← Required! Without this, CPU usage will hit 100%
            elapsed += 1

            if elapsed >= interval:
                current_time = time.strftime("%I:%M %p")
                self.send_text(
                    self.plugin_queue,
                    f"[System Notification] The current time is {current_time}. {ai_name}, please announce the time."
                )
                logger.info(f"[{self.PLUGIN_NAME}] Time sent to AI: {current_time}")
                elapsed = 0
```

---

## 8. Template for a TOOL Plugin

A manual tool that can be operated even during streaming. Simply setting `PLUGIN_TYPE = "TOOL"` makes the panel operable during streams.

### Simple TOOL type (badge controlled by enabled only)

A simple configuration without `is_connected` / `is_running` defined.
The badge stays active as long as `enabled = True`, so it lights up blue even before the stream starts.

```python
import tkinter as tk
from tkinter import ttk, messagebox
from plugin_manager import BasePlugin
import logger


class ManualCuePlugin(BasePlugin):
    PLUGIN_ID   = "manual_cue_plugin"
    PLUGIN_NAME = "🎬 Manual Cue Sender"
    PLUGIN_TYPE = "TOOL"  # ← This is the key

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        # ★ Do not define is_connected / is_running
        #   → Badge is controlled by the enabled value in get_default_settings

    def get_default_settings(self):
        return {"enabled": True}  # Badge lights up blue before the stream starts

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        if hasattr(self, "panel") and self.panel is not None and self.panel.winfo_exists():
            self.panel.lift()
            return

        self.panel = tk.Toplevel(parent_window)
        self.panel.title(self.PLUGIN_NAME)
        self.panel.geometry("320x200")
        self.panel.attributes("-topmost", True)

        ttk.Label(self.panel, text="Message to send to AI", font=("", 11)).pack(pady=10)

        ent = ttk.Entry(self.panel, width=35)
        ent.pack(pady=5)
        ent.insert(0, "Go get a laugh right now!")

        lbl = ttk.Label(self.panel, text="")
        lbl.pack(pady=5)

        def send():
            if not self.plugin_queue:
                messagebox.showwarning("Warning", "Please start the stream first before pressing this")
                return
            msg = ent.get().strip()
            if not msg:
                return
            self.send_text(self.plugin_queue, f"[Director's Instruction] {msg}")
            lbl.config(text="✅ Sent!", foreground="green")
            logger.info(f"[{self.PLUGIN_NAME}] Manual cue sent: {msg}")

        ttk.Button(self.panel, text="📨 Send to AI", command=send).pack(pady=10)
```

### is_connected=True fixed pattern (always-active type)

If your plugin needs no external connection but you want the badge to always be active regardless of the `enabled` setting,
set `is_connected = True` as a fixed value.

```python
class AlwaysActiveToolPlugin(BasePlugin):
    PLUGIN_ID   = "always_active_plugin"
    PLUGIN_NAME = "🔧 Always Active Plugin"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.is_connected = True  # Fixed True → badge is always active
        self.plugin_queue = None

    def get_default_settings(self):
        return {"enabled": True}

    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None
```

### TOOL type with is_connected (external service connection type)

The pattern for plugins that connect to external services like YouTube or Twitch.
The badge turns blue only when `is_connected = True`.

```python
class ExternalServicePlugin(BasePlugin):
    PLUGIN_ID   = "my_service_plugin"
    PLUGIN_NAME = "🔗 External Service Integration"
    PLUGIN_TYPE = "TOOL"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None
        self.is_connected = False  # ← Set to True when connection is established (used for badge display)
        self.is_running   = False  # ← Set to True during live stream

    def get_default_settings(self):
        return {"enabled": True, "url": ""}

    def _connect(self, url):
        # ... connection logic ...
        self.is_connected = True   # ← Badge turns blue on successful connection

    def _disconnect(self):
        self.is_connected = False  # ← Badge turns gray on disconnect

    def start(self, prompt_config, plugin_queue):
        if not self.is_connected:
            return  # Do not start if not connected
        self.is_running   = True
        self.plugin_queue = plugin_queue

    def stop(self):
        self.is_running   = False
        self.plugin_queue = None
```

---

## 9. How to Create a CMD Command Plugin

A plugin that executes processing when the AI writes `[CMD:XXX] argument` in its output.

### How It Works

```
AI output:  "Let me display the image now [CMD:IMG] create sunset landscape"
              ↓
TeloPon detects [CMD:IMG]
              ↓
ImgPlugin's handle("create sunset landscape") is called
```

### Minimal Example

```python
from plugin_manager import BasePlugin
import logger


class MyCommandPlugin(BasePlugin):
    IDENTIFIER = "MYCMD"   # ← Responds to [CMD:MYCMD]. Do not set PLUGIN_ID
    REQUIRED_CONFIG = []

    def handle(self, value: str):
        """The part after [CMD:MYCMD] is passed as value"""
        logger.info(f"[CMD:MYCMD] Received: {value}")
        # Write your processing here


plugin = MyCommandPlugin()
```

### Auto-Discovery

CMD plugins are **automatically registered simply by placing the `.py` file in the `plugins/` folder**.
As long as `IDENTIFIER` is set, the app will auto-scan at startup — no other changes are needed.

```
TeloPon/
 └── plugins/
      └── my_command_plugin.py   ← Placing it here activates [CMD:MYCMD]
```

### How to Use REQUIRED_CONFIG

If the plugin cannot function without certain settings (like API keys), list the setting keys in `REQUIRED_CONFIG`.
If they are not configured, the plugin will automatically be skipped, preventing errors.

```python
class WeatherPlugin(BasePlugin):
    IDENTIFIER      = "WEATHER"
    REQUIRED_CONFIG = ["weather_api_key"]  # config.weather_api_key is required

    def handle(self, value: str):
        import config
        api_key = config.app_config["weather_api_key"]
        # ...
```

---

## 10. Hybrid Plugin: Combining UI Panel and CMD

Setting both `PLUGIN_ID` and `IDENTIFIER` gives a plugin both UI panel and CMD command functionality.

```python
from plugin_manager import BasePlugin
import logger


class HybridPlugin(BasePlugin):
    # Registered as a UI panel
    PLUGIN_ID   = "hybrid_plugin"
    PLUGIN_NAME = "🔀 Hybrid Plugin"
    PLUGIN_TYPE = "TOOL"

    # Also registered as a CMD command
    IDENTIFIER = "HYBRID"

    def __init__(self):
        super().__init__()
        self.plugin_queue = None

    # ----- UI Panel side -----
    def start(self, prompt_config, plugin_queue):
        self.plugin_queue = plugin_queue

    def stop(self):
        self.plugin_queue = None

    def open_settings_ui(self, parent_window):
        # ... settings panel UI

    # ----- CMD Command side -----
    def handle(self, value: str):
        """Processing called when [CMD:HYBRID] is received"""
        logger.info(f"[CMD:HYBRID] Received: {value}")
        if self.plugin_queue:
            self.send_text(self.plugin_queue, f"Hybrid processing: {value}")
```

Hybrid plugins are also auto-registered by placing the file in the `plugins/` folder.

---

## 11. Example of an Existing Plugin: img_plugin.py

An example of a CMD plugin actually in use (`plugins/img_plugin.py`).

```
[CMD:IMG] create sunset cat   → Generate and display an image using Gemini AI
[CMD:IMG] title               → Display TITLE.png in large format
[CMD:IMG] last                → Re-display the previously generated image
[CMD:IMG] hide                → Hide the image
[CMD:IMG] big                 → Display image in large format
[CMD:IMG] small               → Display image in reduced size
```

`ImgPlugin` is CMD-only so it has no `PLUGIN_ID` (does not appear in the UI panel),
and only has `IDENTIFIER = "IMG"` set.

---

## 12. Commonly Used Methods and Utilities Summary

### send_text / send_image (Built into BasePlugin)

```python
# Send text to the AI
self.send_text(self.plugin_queue, "Message to AI")

# Send an image to the AI
# send_image(queue, image_bytes, mime_type="image/jpeg")
#   queue       : pass self.plugin_queue
#   image_bytes : image data in bytes format
#   mime_type   : optional. defaults to "image/jpeg" (positional or keyword argument both accepted)
self.send_image(self.plugin_queue, img_bytes, mime_type="image/jpeg")
```

### logger (TeloPon standard log output)

```python
import logger

logger.info("Normal log (white/green)")
logger.warning("Warning log (yellow)")
logger.error("Error log (red)")
logger.debug("Debug log (faded color. Only shown when launched with -d option)")
```

---

## 13. Plugin Development Checklist

### When creating a UI Panel Plugin

- [ ] Set `PLUGIN_ID` (unique alphanumeric string)
- [ ] Set `PLUGIN_NAME` (the name displayed in the UI)
- [ ] Set `PLUGIN_TYPE` to `"BACKGROUND"` or `"TOOL"`
- [ ] Placed the `.py` file in the `plugins/` folder
- [ ] If starting a thread in `start()`, the flag is set to False in `stop()`
- [ ] Added `time.sleep()` inside the `while` loop (forgetting this causes 100% CPU usage)
- [ ] Wrapped error-prone code in `try-except`
- [ ] Decided on badge display approach (`is_connected` or `enabled` fallback)

### When creating a CMD Command Plugin

- [ ] Set `IDENTIFIER` (uppercase letters recommended; this is the XXX in `[CMD:XXX]`)
- [ ] Implemented `handle(self, value: str)`
- [ ] Placed the `.py` file in the `plugins/` folder
- [ ] If required settings exist, listed the key names in `REQUIRED_CONFIG`

### Common

- [ ] Importing with `from plugin_manager import BasePlugin`
- [ ] Calling `super().__init__()` at the start of `__init__` (required when using `PLUGIN_ID`)
- [ ] Using `logger.info` / `logger.warning` for log output

---

## 14. Common Mistakes and How to Fix Them

**Plugin not appearing in the UI panel**
→ Check that `PLUGIN_ID` is not still set to `""`. If empty, it won't be scanned.

**CMD not responding**
→ Check that `IDENTIFIER` is set and that the `.py` file is placed in the `plugins/` folder.

**System freezes during streaming**
→ The cause is a missing `time.sleep(1.0)` inside the `while self.is_running:` loop.

**Thread doesn't stop**
→ Check that `stop()` sets the flag to `False`. Setting threads to `daemon=True` is a safe practice.

**Settings not being saved**
→ `save_settings()` won't work if `PLUGIN_ID` is not set. Check that `super().__init__()` is being called.

**Badge doesn't light up even with enabled=True**
→ If `self.is_connected = False` or `self.is_running = False` is defined in `__init__`, these take priority over `enabled` and the badge stays gray.
For plugins that need no external service, choose one of the following:
- **Do not define `is_connected` / `is_running`** → `enabled` fallback is used (recommended for simple plugins)
- **Fix `self.is_connected = True`** → badge is always active (use when you always want it lit)

Initializing `is_connected = False` and later switching it to `True` behaves like an external-service plugin (badge is gray while not connected). This is a valid pattern when used intentionally.

**`[CMD:IMG]` settings not being applied**
→ Check that `img_plugin.py` is placed in the `plugins/` folder.
