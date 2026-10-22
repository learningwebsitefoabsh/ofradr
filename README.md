# Gemini Desktop AI Chatbot — Project Specification

> **Copy-and-paste development prompt:** Build a polished, lightweight Windows desktop AI assistant for writing, chat, and on-demand screenshot analysis. The assistant must run locally as a Python application and use **Google Gemini API only** for AI responses.

## 1. Project goal

Create an easy-to-use desktop chatbot that can stay accessible while the user writes in another application. It should offer a floating, always-on-top chat panel, let the user write inside that panel, and optionally send **one screenshot plus one typed instruction** to Gemini for context-aware help. It must be possible to hide, reopen, and fully exit the assistant at any time.

## 2. Required functionality

### A. Floating desktop interface
- Create a modern, minimal UI using **PySide6**.
- Provide a compact **always-on-top** window that is movable, resizable, and optionally semi-transparent.
- Include a conversation display, multiline prompt editor, **Send**, **Capture Screen + Ask**, **Copy Response**, **Clear Chat**, and **Close** controls.
- Make the conversation readable with distinct user/assistant message styles, scrollable history, text selection, and copy support.
- Support a single-panel workflow with no unnecessary tabs.
- Include a **pin/unpin** option for always-on-top behavior and a clear indicator when screen capture is enabled for a request.
- Keep the panel usable without taking focus from the writing app except when the user intentionally opens or interacts with it.

### B. Global shortcut and lifecycle
- The requested toggle hotkey is **Ctrl+Z+X**: open or hide the panel.
- Register the shortcut when the assistant starts and unregister it on exit. Handle shortcut conflicts gracefully.
- Provide a system-tray icon with **Show/Hide** and **Exit Completely** options.
- The app may launch without an extra terminal window in a normal Windows desktop session; it must remain visible in Task Manager and must not hide itself from security or application controls.
- **Exit Completely** must stop global hotkeys, worker threads, tray components, and the Python process.

### C. Gemini-only AI chat
- Use **Google Gemini API exclusively**; do not add OpenAI or another AI provider.
- Allow ordinary text conversations, follow-up messages, and writing-related requests.
- Preserve conversation context for the current session, and provide a **New Chat** action to reset it.
- Show a loading state while a response is generated and display a useful error if the request fails.
- Keep network calls off the GUI thread so the interface stays responsive.
- Make the Gemini model selectable through a small setting or configuration value. Use a currently available Gemini model that supports image input for screenshot requests.

### D. Screenshot + custom instruction
- Capture the **currently selected screen/display once per request** using `mss`.
- Let the user enter a custom instruction such as: “Improve the story on this screen,” “Explain this paragraph,” or “Suggest what to write next.”
- On **Capture Screen + Ask**, send exactly **one freshly captured screenshot and the user's typed instruction** in a single request to Gemini.
- Show Gemini's returned text in the chat panel, with easy copy support.
- Do **not** continuously record, take screenshots in the background, or capture automatically when the hotkey is pressed. Capture only after explicit user action.
- Provide a preview or clear confirmation before sending an image. Do not retain screenshot files after the request unless the user explicitly saves one.

### E. Writing and editing tools
- Allow the user to type and edit longer content inside the assistant's multiline writing area.
- Support **Copy**, **Paste**, **Select All**, keyboard navigation, and common editing shortcuts.
- Add optional quick prompts: **Continue Story**, **Rewrite**, **Fix Grammar**, **Summarize**, **Expand**, and **Change Tone**.
- Never replace or type into another application's content automatically without the user's explicit action. Provide a copy-to-clipboard workflow for manual pasting.

### F. API key and privacy
- On first run, prompt the user to enter their Gemini API key and store it securely with **`keyring`**.
- Never hard-code the API key, include it in logs, or expose it in visible error messages.
- Make it clear that text and any screenshot submitted for analysis are sent to Gemini over the network.
- Do not transmit screenshots or text to other services.
- Provide **Change API Key** and **Clear Chat** actions.

## 3. Technical requirements

| Component | Technology |
| --- | --- |
| Language | Python 3.10+ |
| Desktop interface | `PySide6>=6.5,<7` |
| Global shortcut | `keyboard>=0.13.5` |
| Screenshot capture | `mss>=9.0` |
| Gemini API requests | `requests>=2.31` |
| Secure API-key storage | `keyring>=24.0` |
| Target platform | Windows desktop |

**Implementation preference:** Put the application's main logic in **one well-organized `.py` file**. A separate `requirements.txt` and this Markdown specification are fine. Structure the file with clear classes and functions for the UI, hotkey, screenshots, Gemini requests, API-key settings, and application shutdown.

## 4. Suggested interface

```text
┌──────────────────────────────────────────────┐
│ ✦ Gemini Desktop Assistant      [Pin] [—] [×]│
├──────────────────────────────────────────────┤
│ You: Help me make this story more exciting. │
│                                              │
│ Gemini: Here is a more engaging version...  │
│                                              │
│ [Copy Response]                              │
├──────────────────────────────────────────────┤
│ Write your message or paste story text...   │
│                                              │
├──────────────────────────────────────────────┤
│ [Send] [Capture Screen + Ask] [New Chat]     │
└──────────────────────────────────────────────┘
```

**Design direction:** clean spacing, readable fonts, subtle dark theme by default, rounded controls, sensible keyboard navigation, and responsive resizing. Avoid clutter.

## 5. Expected user workflow

1. Start the Python application on a normal Windows desktop and configure the Gemini API key once.
2. Use **Ctrl+Z+X** to show or hide the floating panel.
3. Type a normal question or a story-writing instruction and select **Send** for text-only chat.
4. For screen-aware help, type an instruction, select **Capture Screen + Ask**, review/confirm the screenshot, and submit it to Gemini.
5. Read and copy the answer; continue chatting, rewrite content, or paste copied text into the writing application manually.
6. Hide the panel to keep it available, or select **Exit Completely** to stop the assistant.

## 6. Reliability and boundaries

- Handle missing/invalid API keys, no internet, timeouts, Gemini rate limits, hotkey registration failures, screenshot errors, and closed windows gracefully.
- Never freeze the interface while the network request runs.
- Allow users to cancel or dismiss a pending operation where practical.
- Respect normal OS and application security boundaries. **Do not bypass protected or exclusive full-screen applications, proctoring systems, or applications that intentionally block other programs.** The overlay works only where the OS and other applications permit it.
- Do not implement stealth mechanisms, monitoring evasion, or hidden screenshot collection.

## 7. Deliverables for the developer or coding AI

Produce:

1. A complete, working **single-file Python application** (for example, `gemini_desktop_assistant.py`).
2. A `requirements.txt` with the specified dependency versions.
3. Simple Windows setup instructions, including how to run the app without a separate console window when appropriate.
4. Clear steps to configure the Gemini API key and test both text chat and screenshot analysis.
5. Brief troubleshooting notes for unsupported overlay environments, conflicting hotkeys, permissions, and Gemini API errors.

**Acceptance criteria:** I can open the assistant using **Ctrl+Z+X**, chat with Gemini, intentionally submit one screenshot with one instruction, see and copy responses in a floating panel, write inside the panel, hide it, and fully quit it—without unexpected captures or background uploads.
