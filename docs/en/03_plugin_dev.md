# TeloPon: Complete Plugin Development Guide 🧩

TeloPon plugins come in **2 main types**.

| Type | Description | Registration |
|------|------|----------|
| **UI Panel Plugin** | Displayed in TeloPon's "Extensions" screen. Provides AI data injection and settings panels. | Auto-registered by placing the file in the `plugins/` folder |
| **CMD Command Plugin** | Handles processing called when the AI says `[CMD:XXX]` in its output. | Requires adding one line to `plugin_loader.py` |

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

## 3. How to Create a UI Panel Plugin

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

## 4. Injecting Information into the AI (The Main Role of UI Panel Plugins)

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

    # Using the dedicated method is convenient
    self.send_image(self.plugin_queue, buf.getvalue())
    # Then send instructions about what the image is — this is the key!
    self.send_text(self.plugin_queue, "Please comment on the image you just saw.")
```

---

## 5. Full Template for a BACKGROUND Plugin

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

## 6. Template for a TOOL Plugin

A manual tool that can be operated even during streaming. Simply setting `PLUGIN_TYPE = "TOOL"` makes the panel operable during streams.

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

    def get_default_settings(self):
        return {"enabled": True}  # TOOL type is always True by default

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

---

## 7. How to Create a CMD Command Plugin

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

### Registering in plugin_loader.py

CMD plugins **must be manually added to `plugin_loader.py`**.
(Unlike UI Panel Plugins, they are not auto-scanned.)

```python
# plugin_loader.py

import cmd_dispatcher
import logger
from plugins.img_plugin import ImgPlugin
from plugins.my_command_plugin import MyCommandPlugin  # ← Add this


def load_plugins(ui):
    plugins = [
        ImgPlugin(
            notify_new_img=lambda p: ui.after(0, lambda: ui.show_new_img(p))
        ),
        MyCommandPlugin(),  # ← Add this
    ]

    for plugin in plugins:
        cmd_dispatcher.dispatcher.register_plugin(plugin)

    logger.info(f"[PluginLoader] Registered {len(plugins)} plugins")
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

## 8. Hybrid Plugin: Combining UI Panel and CMD

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

Hybrid plugins also require adding an entry to `plugin_loader.py` (for CMD registration).

---

## 9. Example of an Existing Plugin: img_plugin.py

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

## 10. Commonly Used Methods and Utilities Summary

### send_text / send_image (Built into BasePlugin)

```python
# Send text to the AI
self.send_text(self.plugin_queue, "Message to AI")

# Send an image to the AI (pass as bytes)
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

## 11. Plugin Development Checklist

### When creating a UI Panel Plugin

- [ ] Set `PLUGIN_ID` (unique alphanumeric string)
- [ ] Set `PLUGIN_NAME` (the name displayed in the UI)
- [ ] Set `PLUGIN_TYPE` to `"BACKGROUND"` or `"TOOL"`
- [ ] Placed the `.py` file in the `plugins/` folder
- [ ] If starting a thread in `start()`, the flag is set to False in `stop()`
- [ ] Added `time.sleep()` inside the `while` loop (forgetting this causes 100% CPU usage)
- [ ] Wrapped error-prone code in `try-except`

### When creating a CMD Command Plugin

- [ ] Set `IDENTIFIER` (uppercase letters recommended; this is the XXX in `[CMD:XXX]`)
- [ ] Implemented `handle(self, value: str)`
- [ ] Added the import and registration to `plugin_loader.py`
- [ ] If required settings exist, listed the key names in `REQUIRED_CONFIG`

### Common

- [ ] Importing with `from plugin_manager import BasePlugin`
- [ ] Calling `super().__init__()` at the start of `__init__` (required when using `PLUGIN_ID`)
- [ ] Using `logger.info` / `logger.warning` for log output

---

## 12. Common Mistakes and How to Fix Them

**Plugin not appearing in the UI panel**
→ Check that `PLUGIN_ID` is not still set to `""`. If empty, it won't be scanned.

**CMD not responding**
→ Check that the registration has been added to `plugin_loader.py`. Unlike UI plugins, it is not auto-scanned.

**System freezes during streaming**
→ The cause is a missing `time.sleep(1.0)` inside the `while self.is_running:` loop.

**Thread doesn't stop**
→ Check that `stop()` sets the flag to `False`. Setting threads to `daemon=True` is a safe practice.

**Settings not being saved**
→ `save_settings()` won't work if `PLUGIN_ID` is not set. Check that `super().__init__()` is being called.

**`[CMD:IMG]` settings not being applied**
→ If `img_plugin.py` is not in the `plugins/` folder, it won't appear in the UI and the `plugin_loader.py` registration will also fail. Confirm the file exists.
