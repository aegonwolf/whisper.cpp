diff --git a//dev/null b/plan-layout.md
index 0000000000000000000000000000000000000000..38442ebd634acca7644f227f076b37ccbe0ba846 100644
--- a//dev/null
+++ b/plan-layout.md
@@ -0,0 +1,59 @@
+# Minimal MVP Plan for Hotkey-based Speech-to-Text using whisper.cpp
+
+## 1. Core Goal
+Provide fast, offline transcription triggered by a user-defined hotkey. After recording, text is placed on the clipboard and optionally auto-pasted into the active text field.
+
+## 2. Rubber Duck Reasoning
+- We need a trigger: a global hotkey is simpler and reliable; voice phrase can be explored later.
+- Using whisper.cpp allows local inference with no internet and good accuracy.
+- A lightweight Win32 app avoids external dependencies and keeps binary small.
+- Miniaudio or WASAPI handles audio capture with minimal setup.
+- Clipboard operations are easy with Win32 API.
+- Auto-pasting should happen only when the active window accepts text to avoid unwanted behavior.
+
+## 3. Action Plan
+
+1. **Build whisper.cpp library**
+   - Compile whisper.cpp as a static or dynamic library using MSVC and CMake.
+   - Justification: reuse existing build system; output library for integration.
+
+2. **Set up minimal Win32 application**
+   - Create a Win32 GUI app with no visible window; provide system tray icon for exit.
+   - Justification: keeps UI out of the way yet allows control.
+
+3. **Register global hotkey**
+   - Use `RegisterHotKey` for Ctrl+Shift+Space (configurable).
+   - On first press, begin recording; on second press, stop and transcribe.
+   - Justification: easiest way to start/stop capture without complex UI.
+
+4. **Record microphone audio**
+   - Capture mono PCM at 16 kHz using WASAPI or miniaudio.
+   - Buffer audio in memory.
+   - Justification: whisper.cpp expects 16 kHz PCM; miniaudio/WASAPI are lightweight.
+
+5. **Run transcription**
+   - Load selected GGML model on startup.
+   - After recording stops, feed audio to whisper.cpp and obtain text.
+   - Justification: processing after recording keeps pipeline simple.
+
+6. **Copy text to clipboard**
+   - Use `OpenClipboard`/`SetClipboardData` to place UTF-8/Unicode text.
+   - Justification: clipboard makes transcript available to any app.
+
+7. **Optional auto-paste**
+   - Query foreground window; if it contains an edit control, send `Ctrl+V`.
+   - Else, notify user that text is on clipboard.
+   - Justification: automatic paste improves flow but should not interfere.
+
+8. **Testing**
+   - Manual test: press hotkey, speak, verify transcription and clipboard content.
+   - Stress test with longer speech and different applications.
+
+9. **Packaging**
+   - Bundle executable with tiny model (tiny.en) in ZIP.
+   - Provide README for setup and configuring hotkey.
+
+## 4. Future Work
+- Voice phrase trigger ("Hey Whisper") using tiny VAD + keyword spotting.
+- Settings UI for model selection, hotkey customization.
+- Continuous listening with real-time transcription.
