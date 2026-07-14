# AGENTS.md: Raconteur d'Histoires Interactif Telmi

This document explains the **Telmi Story Format** to help an AI agent generate fully functional, interactive stories that can be loaded into **Telmi Sync Studio** for recording voices and compiling to the **Telmi OS** story-teller platform.

---

## 1. Directory Structure

A Telmi story is a folder named with a pattern like `_ID_Title_uuid-hash`. Inside this folder, the following files and folders must exist:

```text
/metadata.json          # Story metadata (Title, description, recommended age, etc.)
/nodes.json             # Game logic graph (Scenes, choices, actions, inventory updates)
/notes.json             # Text contents of each scene, displayed in Telmi Sync Studio for voice acting
/title.mp3              # Audio file playing during story selection on Telmi OS
/title.png              # Poster image displayed in Telmi OS (640x480px PNG)
/cover.png              # Cover image displayed in Telmi Sync (Optional)
/audios/                # Folder containing scene voice lines (s1.mp3, s2.mp3, ...)
/images/                # Folder containing scene background images and item pictures (s1.png, i0.png, ...)
```

---

## 2. JSON Specifications

### 2.1 `metadata.json`
Stores core story metadata.
```json
{
  "title": "Story Title",
  "uuid": "ffffff-19f5799345c",
  "image": "cover.png",
  "version": 1,
  "description": "Short summary of the story.",
  "age": "4"
}
```

### 2.2 `notes.json`
Extremely critical for the user to record their voice later in Telmi Studio. It maps stage/scene keys to their descriptive titles, text to read, and diagram block colors.
```json
{
  "s0": {
    "title": "Introduction",
    "notes": "Here is the story text for scene s0.",
    "color": "green"
  },
  "s1": {
    "title": "Choose your path",
    "notes": "Which path do you want to choose?",
    "color": "yellow"
  }
}
```
*Colors can be:* `pink`, `pink2`, `purple3`, `purple4`, `purple5`, `yellow`, `orange2`, `orange3`, `red`, `red2`, `brown`, `green`, `green2`, `green3`, `green4`, `blue`, `blue2`, `blue3`, `blue4`.

### 2.3 `nodes.json`
Defines the state machine, user inputs, autoplay features, and inventory rules.

#### Structure of `nodes.json`
```json
{
  "startAction": {
    "action": "a0",
    "index": 0
  },
  "inventory": [
    {
      "name": "Item Name",
      "initialNumber": 0,
      "maxNumber": 3,
      "display": 2,
      "image": "i0.png"
    }
  ],
  "stages": {
    "backStage": {
      "image": null,
      "audio": null,
      "ok": { "action": "backChildAction", "index": 0 },
      "home": { "action": "backAction", "index": 0 },
      "control": { "ok": true, "home": false, "autoplay": true }
    },
    "s1": {
      "image": "s1.png",
      "audio": "s1.mp3",
      "ok": { "action": "a3", "index": 0 },
      "home": { "action": "backAction", "index": 0 },
      "control": { "ok": true, "home": true, "autoplay": false },
      "items": [
        { "type": 2, "item": 0, "number": 1 }
      ]
    }
  },
  "actions": {
    "a0": [{"stage": "s1"}],
    "backAction": [{"stage": "backStage"}],
    "backChildAction": []
  }
}
```

* **`inventory` (Optional):**
  * `display` options: `0` (display image + count), `1` (display image + progress gauge), `2` (hidden).
* **`stages`:**
  * `"home": {"action": "backAction", "index": 0}` must be present on almost every user stage to support the "Back" button role correctly.
  * `"ok"`: Tells where to transition when A/B buttons are pressed, or automatically if autoplay is true.
  * `"control"`:
    * `"autoplay": true` means the game automatically moves to the `"ok"` transition when the stage's audio ends.
    * `"autoplay": false` forces the game to pause and wait for user button interaction.
  * `"items"` (Optional): Updates inventory.
    * `type` operations: `0` (`+=`), `1` (`-=`), `2` (`=`), `3` (`*=`), `4` (`/=`), `5` (`%=`).
* **`actions`:**
  * Every key is an action ID pointing to a list of potential target stages.
  * If multiple targets exist, and stage `control.autoplay` is `false`, the left/right buttons allow the player to cycle through them manually (e.g. choice selection).
  * Conditional transitions: You can restrict action entries using the `conditions` list:
    ```json
    "actions": {
      "a3": [
        {
          "stage": "s3",
          "conditions": [
            { "comparator": 2, "item": 0, "number": 1 }
          ]
        },
        {
          "stage": "s4"
        }
      ]
    }
    ```
    * `comparator` values: `0` (`<`), `1` (`<=`), `2` (`==`), `3` (`>`), `4` (`>=`), `5` (`!=`).
    * Use `compareItem` instead of `number` to compare two inventory items.

---

## 3. Designing Interactive Stories

To create a great experience, follow a repeating narrative phase pattern:
1. **Introduction / Context Scene:** A narrative scene with `autoplay: true`. At the end of the audio, it transitions to choices.
2. **Choice Scenes (2-3 options):** Multiple stages in one action. `autoplay: false`. The user cycles with left/right buttons and confirms with A/B. Confirming sets an inventory flag (to remember the choice) and transitions to the consequences.
3. **Consequence Scene:** Plays the resulting text (using conditions if you want to branch dynamically later based on the inventory flag). It has `autoplay: true` and proceeds to the next narrative phase.

---

## 4. Guidelines for the Generation AI

When generating a story:
1. **Scenario & Structure Mapping:** Sketch out all phases. Create state keys carefully (e.g., `s1` to `s40`).
2. **Build `metadata.json`:** Assign a randomized uuid, set appropriate title, recommended age (e.g., `4`), and description.
3. **Build `notes.json`:** Renders the text that will be shown in Telmi Studio for recording. Write a engaging story with rich, narrative elements for kids.
4. **Build `nodes.json`:** Strictly follow the Telmi OS schema. Avoid leaving dangling references. Ensure `"home": {"action": "backAction", "index": 0}` is present on all screens.
5. **Generate Mock Media Files:**
   - Create empty directories `images/` and `audios/`.
   - Create a blank/simple cover image (`cover.png`) and title image (`title.png`).
   - Create a silent MP3 file or placeholder file for `title.mp3` and files under `audios/` if required.
