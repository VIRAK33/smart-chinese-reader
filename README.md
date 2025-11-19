# Smart Chinese Reader

A modern, offline-friendly Chinese reading assistant that lets you:

- Click on any **Chinese word** to see **pinyin + translation tooltip**
- Select text to open a **bottom-sheet dialog** (desktop & mobile) with:
  - **Translate**
  - **Add Note**
  - **Speak (TTS 🔊)**
- Save vocabulary and notes to a **side “Study Tools” drawer**
- Load and read **Markdown / TXT / PDF** files

All UI is optimized for both **desktop and mobile**, with support for **Khmer + English** in the interface.

---

## Features

### 🔤 Chinese Word Picker

- Automatically segments Chinese text into clickable words using `Intl.Segmenter` (where available).
- Each word is rendered as:
  - **Chinese characters** (Noto Serif SC)
  - **Inline pinyin** (optional)
- Click a word to:
  - Highlight it
  - Show a tooltip with pinyin + live translation from the configured provider

### 🌐 Translation Providers

The reader supports multiple translation providers via their official HTTP APIs:

- **Google Translate** (default)
- **Bing / Microsoft Translator**
- **DeepL**
- **Youdao** (placeholder – recommended to use via your own backend proxy)

You can:

1. Enter your API key in **“Translate API Key”**.
2. Select provider in the **Provider** dropdown.
3. Choose target language (**English / Khmer**) in **Translate To**.

#### Provider Setup Notes

##### 1. Google Translate

- Endpoint: `https://translation.googleapis.com/language/translate/v2`
- Input: API key only.
- UI label: **Google Translate**
- Works for:
  - Tooltips when clicking a word
  - Selection dialog “Translate”
  - “Pre-translate Vocabulary” batch feature

##### 2. Bing / Microsoft Translator

- Endpoint: `https://api.cognitive.microsofttranslator.com/translate?api-version=3.0`
- Key format in input field:  
  `KEY|REGION`

  Example:

```text
  my-secret-key-123|eastasia
````

* UI label: **Bing (Microsoft)**
* Internal mapping:

  * From: `zh-Hans`
  * To: `en` (Khmer currently falls back to `en`)

##### 3. DeepL

* Endpoint: `https://api-free.deepl.com/v2/translate`
* Input: your **DeepL auth key**.
* UI label: **DeepL**
* Limitations:

  * DeepL doesn’t support Khmer, so `km` falls back to `EN`.

##### 4. Youdao

* UI label: **Youdao**
* Note: **not implemented on front-end** for security reasons.
  Real Youdao usage requires `appKey/appSecret` signing and SHOULD be done through your own backend API.
  Currently it returns a placeholder message indicating this.

---

## Text-to-Speech (TTS)

* Uses **browser’s Web Speech API** (`speechSynthesis`).
* In the Selection dialog, tap the **🔊 Speak** icon to hear the selected text.
* Language selection:

  * If text contains Chinese characters → `zh-CN`
  * Else:

    * If target language is Khmer → `km-KH` (if supported by the browser)
    * Otherwise → `en-US`

> ⚠️ TTS support depends on the browser & OS. Desktop Chrome/Edge usually work better than mobile.

---

## File Types & Input

You can load text via:

1. **Markdown / TXT / PDF upload**

   * Use the **Drop or choose file** card.
   * Supported extensions: `.md`, `.markdown`, `.txt`, `.pdf`
   * PDFs are processed with **PDF.js** and plain text is extracted page by page.

2. **Direct text input**

   * Paste or type Chinese text in the **Markdown input** editor at the top.
   * Uses `marked` to render Markdown into the page.

All uploaded files and their content are stored in `localStorage` under `uploadedMdFiles`, so they persist in the browser.

---

## Study Tools Drawer

Click the **📝 FAB button** (bottom-right) to open the **Study Tools** drawer.

### Saved Translations

* Created when you:

  1. Select text.
  2. Click **Translate**.
  3. Then click **Save**.
* Each translation card shows:

  * Original text (with optional pinyin)
  * Translated text
* “Show Pinyin” checkbox per card.
* Delete translations with the **Delete** button.

### Sticky Notes

* Created when you:

  1. Select text.
  2. Click **Note** in the selection dialog.
  3. Enter your note.
* Text in the main document is highlighted with a yellow **note marker + 📌 pin**.
* Clicking a note in the drawer:

  * Scrolls to the note in the main text.
  * Briefly opens a small **Note** popup near the highlight.

---

## Inline Pinyin

* Global toggle: **Show Inline Pinyin** (checkbox in the sidebar).
* When enabled:

  * All `.zh-word` spans show pinyin above characters.
* For saved translations in the drawer:

  * Additional per-card checkbox “Show Pinyin”.

---

## Batch Vocab Translation

Button: **⚡ Pre-translate Vocabulary**

* Scans the main document and collects all unique Chinese words.
* For each word that is **not yet in the translation cache**, it calls the current provider:

  * **Google**: batched requests (50 words per request).
  * **Other providers**: words are translated one by one.
* Progress is shown in the button text: `Trans: current/total...`
* Results are cached in `localStorage` (`translationCache`).

This dramatically speeds up later word-click tooltips.

---

## Mobile Experience

* Sidebar (`Files + API settings`) becomes a **slide-in panel** on small screens:

  * Toggle with **☰ Files** button in the top-left.
* Study Tools drawer remains a right-side **sheet**.
* Selection dialog:

  * On mobile, text selection is detected via `selectionchange`.
  * A **bottom-sheet “Selection” dialog** appears with:

    * **🔊 Speak**
    * **Translate**
    * **Note**
  * A dedicated **selection overlay** ensures taps do not “fall through” to the text behind.

---

## Storage

The app uses `localStorage` for:

* `uploadedMdFiles` – list of uploaded files `{ name, content }`.
* `stickyNotes` – notes per file.
* `savedTranslations` – saved vocabulary.
* `translateApiKey` / `googleTranslateApiKey` – translation key.
* `translateProvider` – active provider (`google`, `bing`, `deepl`, `youdao`).
* `targetLang` – `en` or `km`.
* `translationCache` – map of `(provider|text_lang)` → translation.

> 🔐 Your API keys are stored **only in your browser** (no server).


---

## Tech Stack

* **Core:**

  * Vanilla HTML + CSS + JS
* **Libraries:**

  * [`marked`](https://github.com/markedjs/marked) for Markdown parsing
  * [`pinyin-pro`](https://github.com/zh-lx/pinyin-pro) for pinyin
  * [`pdf.js`](https://github.com/mozilla/pdf.js) for PDF text extraction
* **Browser APIs:**

  * `Intl.Segmenter` for Chinese word segmentation
  * Web Speech API (`speechSynthesis`) for TTS
  * `localStorage` for persistence

---

## Known Limitations / Notes

* **Youdao**: only a placeholder in the front-end. You should proxy to Youdao via your own backend to keep secrets safe.
* **TTS**:

  * Quality and voice availability depend on the browser and OS.
  * Some browsers may not support `km-KH`.
* **PDF extraction**:

  * Complex layouts / scanned PDFs may not extract perfectly (depends on PDF.js and text layer quality).

---

## Customization

You can customize:

* Colors, spacing, and radii via the `:root` CSS variables in the `<style>` section.
* Supported languages in:

  * Target language dropdown (`#lang-select`).
  * Provider language mappings inside the JS (`getTranslation`).
* Fonts:

  * UI font: `"Noto Sans Khmer"` + system fonts.
  * Chinese: `"Noto Serif SC"`.

---

Happy reading! 📚✨
If you’d like, I can also generate a **short “Getting Started” guide** in Khmer or Chinese to include in this README.
