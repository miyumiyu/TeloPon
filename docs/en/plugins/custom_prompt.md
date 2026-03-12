# 📝 Custom Instruction Addon Tool (custom_prompt.py)

This plugin lets you **add "special rules" or "extra instructions" for the current stream session to the AI, without directly editing the base prompt (AI personality/script)**.

It's extremely convenient for temporary customizations like "I'm playing a specific game today, so please keep the conversation on that topic" or "Today I want to change my speech pattern for a special occasion".

---

## ⚙️ Usage

### 1. Open the Settings Panel
Click the **"⚙️ Settings"** button for **"Custom Instruction Addon Tool"** in the "Extensions (Plugins)" panel on the right side of the TeloPon main screen.

![Custom Instruction Addon Tool settings panel](../../../images/custom_prompt.png)

### 2. Enter Your Custom Instructions
In the white text area in the center, freely write the additional instructions you want the AI to follow.
The trick is to write it in a direct, conversational way, as if you're asking the AI a favor.

**[Input examples]**
> * "Today I'm playing Minecraft. Please react a lot to what happens in the game!"
> * "Today is my birthday stream! Please throw in congratulations whenever you get the chance."
> * "Today please end every sentence with 'nya'."
> * "Even if I make a mistake in the game, please never get mad — just gently comfort me."

### 3. Enable and Save
* **Check** the **"Enable this custom instruction"** checkbox in the upper-left of the screen. (※ If this is not checked, text you've written will not be sent to the AI.)
* Press **"Save"** to close the window.
* You're ready when the plugin badge in the main screen changes from grey `OFF` to green `ON`.

### 4. Start the Live Connection
Press **"🔴 Start Live Connection"** on the main screen.
When the AI connection is established, the special rules you entered will be quietly appended to the end of your selected "AI Personality (Prompt)".

---

## ⚠️ Important Notes

* **Always configure before starting the live connection**
  This tool's instructions are sent to the AI exactly once — at the moment you start the session (when you press "Start Live Connection").
* **Cannot be changed mid-stream**
  If you want to change the instructions during a stream, disconnect once, change the settings, then start the live connection again.
* **Watch out for instructions that conflict with the base prompt**
  For example, if your base prompt says "You are a calm and composed AI" but you instruct this tool to "Be as high-energy as possible and shout everything!", the AI may get confused and stop responding (going silent and not reacting).

---
[⬅️ Back to Plugin List](../../README.md)
