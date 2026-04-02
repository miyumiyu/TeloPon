# 🎮 How to Control Telops in OBS

TeloPon telops can be freely **dragged** and **resized** directly in OBS.
This page explains how to interact with telops on an OBS browser source.

---

## 1. Enter OBS "Interact" Mode

Click the **"Interact" button** at the bottom of the OBS preview area.

![OBS Interact Button](../../images/obsctl.png)

> After clicking, the preview area will be outlined in red, indicating that browser source interaction is enabled.

---

## 2. Telop Controls

In Interact mode, you can directly control telops with your mouse.

![OBS Interact Mode](../../images/obsctl2.png)

### Drag to Move

| Target | Action |
|---|---|
| **Normal telops** (variety / news / reply, etc.) | Drag to move to any position. Position is remembered after OBS restarts |
| **Explain frame** | Drag to move. Position is remembered after OBS restarts |
| **Nameplate** | Displayed at the bottom of the screen. Click the "✕" at the top-right to dismiss |

### Scroll Wheel to Resize

Scroll the **mouse wheel** over a telop to zoom in or out.

* **Wheel up** → Zoom in
* **Wheel down** → Zoom out
* Scale is remembered per group

### Double-Click to Reset

**Double-click** a telop to reset its position and scale to the default state.

### Long Press to Dismiss

**Press and hold** a telop for 1 second to manually dismiss it.

---

## 3. Explain Frame Special Controls

The explain frame has several additional features.

| Action | Result |
|---|---|
| **Drag** | Move freely (position remembered after OBS restart) |
| **Scroll wheel** | Zoom in/out (scale also remembered) |
| **Double-click** | Reset position and scale |
| **Long press (1s)** | Dismiss current explain and advance to next |
| **Drag to edge** | Frame compresses horizontally when pushed to viewport edge. Returns when pulled back |
| **"Next" badge** | When items are queued, a badge appears at the bottom-right. Click to advance to the next item |

---
