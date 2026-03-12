# TeloPon: Theme & CSS Customization Guide 🎨

In TeloPon, you can freely customize the design of the telops displayed in OBS using "Themes".

TeloPon's rendering system is highly efficient — it uses a **"common HTML skeleton with swappable individual CSS designs"** architecture. This means creating a new theme requires **nothing more than creating a single CSS file**.

---

## 📁 1. Where to Save Theme Files

To create a new theme, create and save a CSS file inside the `TeloPon/themes/` folder.

```text
TeloPon/
 └── themes/
      ├── default.css        ← Standard theme
      └── my_custom.css      ← 🌟 Just add your custom CSS file here!
```
※ Any CSS file you add will automatically appear in the "⚙️ Telop Theme" dropdown in the TeloPon UI.

---

## 🏗️ 2. [Important] HTML Structure and the Two Telop Containers

TeloPon's display (`base.html`) has two telop containers built in: the **"Normal container"** and the **"Special container"**.
Which container is used is determined automatically by the "Window ID" (described below) that the AI specifies.

### ① Normal Telop Container (normal group)
The standard telop container that displays in sequence, typically in the upper-right of the screen.
```html
<div id="teloop-container" class="Window ID and Layout ID go here">
    <div id="teloop-badge"></div>
    <div id="text-wrapper">
        <div class="topic-line">Topic</div>
        <div class="main-line">Main text</div>
    </div>
</div>
```

### ② Special Container (special group)
Used when you want to display a telop "independently (overlapping)" with the normal telop — such as explanation telops at the bottom of the screen or name introductions.
```html
<div id="explain-container" class="Window ID goes here">
    <div id="exp-badge"></div>
    <div id="exp-text-wrapper">
        <div class="topic-line">Topic</div>
        <div class="main-line">Main text</div>
    </div>
</div>
```

---

## 🪪 3. Available Window IDs and System Control

TeloPon instructs the AI to use predefined "Window IDs". In CSS, you write design rules and system commands targeting the class (`.window-xxx`) for each Window ID.

### 📌 Normal Container IDs (normal group)
These are applied to `#teloop-container` and disappear after the time set in the UI's "Normal display duration (sec)".

* **`.window-simple`**: For the most standard, simple telops.
* **`.window-variety`**: For pop/flashy designs in a variety show style.
* **`.window-news`**: For formal designs in a news broadcast style.
* **`.window-cute`**: For handwritten or cute/kawaii designs.
* **`.window-digital`**: For cyberpunk or digital-feel designs.
* **`.window-reply`**: For viewer replies or mentions (e.g., "Hey @username!") — wide-format designs that stand out.

### 📌 Special Container IDs (special group)
These are applied to `#explain-container`. To tell the system "this is a special container", you **must specify `--window-group: special;` in the CSS**.

* **`.window-explain`**: For long explanations or supplementary info displayed prominently at the bottom of the screen.
* **`.window-nameplate`**: For displaying a TV-style "nickname + name" in large text.

---

## 🔊 4. Playing Sounds and Controlling the System (CSS Variables)

One of TeloPon's greatest features is the ability to **"inject settings back into the system (Python/JS side) from CSS"**. This is done using "CSS variables (declarations starting with `--`)".

### 💡 Playing Sound Effects (`--sound-file`)
To play a sound effect when a telop appears, write `--sound-file` in your CSS. Specify an audio file from the `TeloPon/sounds/` folder.

```css
/* Example: play a "pop" sound when the variety window appears */
.window-variety {
    background: linear-gradient(135deg, #ff00cc, #333399);
    color: white;
    /* Instructs the system to play TeloPon/sounds/sound1.mp3 */
    --sound-file: "sound1.mp3";
}
```

### 💡 Switching a Container to "Special" (`--window-group`)
As mentioned above, when creating special container designs like `window-explain`, omitting this variable will cause it to be treated as a normal container.

```css
.window-explain {
    position: absolute;
    bottom: 40px;
    /* Including this causes rendering in "#explain-container" */
    --window-group: special;
    --sound-file: "sound2.mp3";
}
```

### 💡 Force-Override the Display Duration (`--display-duration`)
You can force a specific telop to display for a fixed duration, ignoring the seconds configured in the UI (e.g., 5 seconds).

```css
.window-nameplate {
    /* Override UI setting — force nameplate to disappear after exactly 5 seconds */
    --window-group: special;
    --display-duration: 5;
}
```

---

## 📐 5. Layout IDs (Adjusting Text Balance)

In addition to the Window ID, the normal container (`#teloop-container`) also receives a "Layout ID". Use this to adjust the **size balance between the topic and main text**.

* **`.layout-flat`**: Flat layout — topic and main text are the same size.
* **`.layout-big-top`**: Larger text in the upper row (topic), smaller in the lower row (main).
* **`.layout-big-bottom`**: Smaller text in the upper row (topic), larger and more prominent in the lower row (main).

**CSS usage example:**
```css
/* When flat is specified, you can apply exceptions like removing borders for a text-only look */
.window-variety.layout-flat .topic-line {
    background: none !important;
    font-size: 36px !important;
}
```

---

## 🎬 6. Implementing Animations (`.visible`)

When a telop appears on screen, the system adds the **`.visible` class** to the container.
Use this with CSS `transition` and `transform` to create smooth entrance animations.

```css
/* =========================================
   Normal telop (scales up and fades in from upper-right)
   ========================================= */
#teloop-container {
    position: absolute; right: 30px; top: 60px;
    opacity: 0;
    transform: scale(0.8);
    transition: all 0.5s;
}
#teloop-container.visible {
    opacity: 1;
    transform: scale(1);
}

/* =========================================
   Explanation telop (slides up from the bottom)
   ========================================= */
.window-explain {
    position: absolute; left: 40px; bottom: 40px;
    opacity: 0;
    transform: translateY(30px);
    transition: all 0.6s cubic-bezier(0.2, 0.8, 0.2, 1);
    --window-group: special;
}
.window-explain.visible {
    opacity: 1;
    transform: translateY(0);
}
```

---

## 💡 7. Development Tips (Debugging)

Adjusting theme CSS by having the AI speak each time is time-consuming.
When you want to check and tweak your design, launch TeloPon in debug mode (`-d`).

```bash
python telopon_live.py -d
```

A **"Manual Control Panel"** will appear at the bottom of the UI.
Enter a topic, main text, and the Window ID you want to test, then press the "Force Display" button to run rendering tests (including sound playback confirmation) as many times as you want — no AI needed.
Fine-tune your design while watching the CSS changes apply in real time in the OBS browser source!
