# TeloPon: AI Prompt Creation Guide 🧠

In TeloPon, you can easily customize the AI's "personality" and "behavior" using **prompt files (text files)**.

This guide explains how to write prompt files to create your own original AI assistant, and the "absolute rules" you must follow to keep the system running at top speed without errors.

---

## 📂 1. Creating and Saving Prompt Files

Prompt files are saved in the **`prompts`** folder inside the TeloPon application folder. Once saved, any file in that folder will be selectable from the main TeloPon screen (AI Personality) at startup.

* **Save location**: `TeloPon/prompts/`
* **File name**: Any name is fine (e.g., `my_assistant.txt`, `snarky_ai.txt`). No special prefix is required.
* **Encoding**: Always save as **UTF-8**.

> **💡 Tip**
> If creating with Windows Notepad, make sure the encoding dropdown in the "Save As" dialog is set to "UTF-8".

---

## 🏗️ 2. Basic File Structure

A prompt file consists of two main blocks: a **"Header (settings)"** and a **"Body (instructions for the AI)"**.
These two blocks must always be separated by a line containing exactly **`---` (three hyphens)**.

```text
NAME: AI's name
START_MSG: System message sent at startup
CMT_MSG: Instructions sent when a comment is received
---
Write your detailed instructions (system prompt) for the AI below this line.
You are an excellent assistant...
```

---

## ⚙️ 3. Writing the Header and Using Placeholder Variables

At the top of the file (above `---`), write the following three items, one per line.

* **`NAME:`**
  The AI's name. Displayed on the UI standby screen and used as the default topic.
* **`START_MSG:`**
  The first message secretly sent from the system to the AI immediately after a live connection starts. This triggers the AI to begin its opening greeting.
* **`CMT_MSG:`**
  Instructions automatically appended to comment data when a YouTube comment or similar arrives.

### 🪄 Magic "Placeholder Variables"
Using the following placeholders in the header or body will cause TeloPon to automatically replace them with the **actual names** at runtime.
This lets you create reusable prompts that work for any streamer without editing the file each time.

* **`{streamer}`**
  Replaced with the name entered in the "🎥 Streamer Name" field on the TeloPon main screen.
  *(Example)* `[MAIN] {streamer} is acting a little weird today.` → if "Miyumiyu" is set in the UI, it becomes `[MAIN] Miyumiyu is acting a little weird today.`
* **`{ai_name}`**
  Replaced with the AI name set in the header's `NAME:` field.
  *(Example)* `Your name is "{ai_name}".` → the AI recognizes itself as `Your name is "TeloPon".`

---

## 🎨 4. Using "Emphasis Tags" to Decorate Telops

Instead of just displaying the AI's lines (the content inside `[MAIN]` tags) as plain text, you can highlight specific words with color to create richer, more TV-like telops.
By instructing the AI in your prompt to "wrap important words in tags", it will automatically apply color formatting.

* **`<b1>text</b1>` (red · emphasis)**
  Use for the most eye-catching word, the punchline of a quip, or an important keyword.
  Displayed in **red** (or the accent color) in the default theme.
* **`<b2>text</b2>` (blue · supplementary)**
  Use for a secondary highlight, supplementary information, etc.
  Displayed in **blue** (or the sub-color) in the default theme.

**[Example instruction for the AI]**
> Wrap important words in your lines with `<b1>...</b1>` (red) or `<b2>...</b2>` (blue) for emphasis.

**[Example output and display]**
`[MAIN] There's a <b1>major announcement</b1> in today's stream! <b2>Don't miss it</b2>!`
👉 "major announcement" appears in red and "Don't miss it" appears in blue on screen.

---

## 🚨 5. [CRITICAL] The "Output Format (Tag System)" the AI Must Always Follow

There is an "output rule (tag system)" that the AI must absolutely follow for TeloPon's system to run at maximum speed without errors. If the AI breaks this rule, telops won't appear in OBS, or the system may freeze.
**Copy and paste the rule text below directly into your prompt (e.g., at the end).**

```text
[ABSOLUTE RULE: Telop Output Format]
You must output all responses using the following "dedicated tag" format.
Each response must contain exactly ONE telop. Multiple patterns or repeated tags are strictly forbidden.
Do not use decorations (Markdown bold, bullet points, etc.) or any preamble/acknowledgment outside the tags. Output starting directly from the tags.

[TOPIC] topic name [MAIN] main text [WND] window ID [LAY] layout ID [BDG] badge [MEMO] AI's internal note
```

### Meaning and Role of Each Tag
1. **`[TOPIC]`**: Small heading shown in the upper-left of the telop (e.g., `AI Assistant`, `Breaking`)
2. **`[MAIN]`**: The main text displayed on screen. Use emphasis tags (`<b1>`, etc.) here.
3. **`[WND]`**: Telop frame design. (e.g., `window-simple`, `window-explain`, `window-news`)
4. **`[LAY]`**: Telop display style. Generally fixed to `layout-flat`.
5. **`[BDG]`**: A small icon or text in the corner of the telop. (Use `NONE` if not needed)
6. **`[MEMO]`**: An internal note for continuing to the next turn. **The system begins displaying on screen the instant it detects the `[MEMO]` tag, so always write it last.**

---

## 🚑 6. When Things Aren't Working (Using Thinking Mode)

If "the AI keeps going silent", "it only speaks occasionally", or "it gives strange responses", the AI may be confused by the prompt instructions.
In that case, try launching TeloPon with the **`-th` (thinking mode) option**.

**[Example launch command]**
```bash
python telopon_live.py -d -th
```

With thinking mode on, a **`[THOUGHT] 🧠`** entry will appear in the console (black window) logs.
This shows the AI's **internal thought process** — "what it heard from the microphone" and "why it chose that response (or why it skipped)".

* **"It's interpreting background noise as speech…"**
* **"My prompt restrictions are too strict — the AI is hesitating because it thinks it shouldn't speak now…"**
* **"The microphone audio is cutting out and the AI keeps restarting its thinking…"**

This makes it clear exactly where the AI is getting stuck (the bottleneck), which is extremely useful for fixing prompts and improving your microphone setup.
※ Once you've finished diagnosing, it's recommended to remove the `-th` option and restart normally for better real-time performance.

---

## 🎁 7. Ready-to-Use Basic Prompt Sample

If this is your first time, copy the text below, save it as `TeloPon/prompts/sample_assistant.txt`, and load it in TeloPon. You'll have a capable character AI assistant ready to go immediately.

```text
NAME: TeloPon
START_MSG: System startup complete. {streamer}, let's make today's stream great! Please say hello to the viewers first.
CMT_MSG: A new viewer comment has arrived. Use this as a conversation starter to keep the show going.
---
You are "{ai_name}", a virtual character who serves as an assistant to streamer "{streamer}".
React to viewer comments and microphone audio to liven up the live stream.

[Guidelines]
- Always speak in a bright, friendly tone.
- You dislike silence. If {streamer} is at a loss for words, immediately jump in with a quip or ad-lib to help them out.
- Treat viewer comments like words from the heavens — pick them up with top priority and exaggerated enthusiasm.

[ABSOLUTE RULE: Telop Output Format (MOST IMPORTANT)]
To display your responses on screen at maximum speed, output your answer as exactly one line in the following "tag format".
Preambles, acknowledgments, Markdown decorations, and multiple tag outputs are strictly forbidden.
Wrap important words in your lines with <b1>...</b1> (red) or <b2>...</b2> (blue) for emphasis.

[TOPIC] topic [MAIN] your response [WND] window ID [LAY] layout ID [BDG] badge [MEMO] your internal note

[Field details]
- [TOPIC]: {ai_name} or a short title of the current topic
- [MAIN]: Your response content (this is displayed as the telop)
- [WND]: Always use "window-simple".
- [LAY]: Always use "layout-flat".
- [BDG]: A single emoji for emotion (e.g., a light bulb or test tube). Use NONE if not needed.
- [MEMO]: A reminder for future developments (the system triggers on detecting this tag, so always write it last).

[Output example]
[TOPIC] {ai_name} [MAIN] Stream is starting! Let's have a <b1>great time</b1> today! [WND] window-simple [LAY] layout-flat [BDG] sparkles [MEMO] Opening greeting complete. Waiting for comments.
```
