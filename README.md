# SMW AI — Native Android Personal Assistant (Kotlin + Jetpack Compose)

## 1. Setup Instructions
1. Open the `android-app/` directory in **Android Studio Ladybug (2024.2+)** or newer with JDK 17.
2. In `gradle.properties`, set `ZARIYA_BACKEND_URL` to your deployed Express + Gemini backend URL (or `http://10.0.2.2:3000` for Android Emulator testing against local `npm run dev`).
3. Sync Gradle (`File -> Sync Project with Gradle Files`).

> **Local Command Execution**: Native device commands (app launch, screen reading, scrolling, screen recording, WhatsApp reader/reply, call handling, flashlight, volume, media, alarms, notes) are matched on-device via `LocalCommandMatcher` without waiting on external AI APIs. Open-ended queries use the server-side Express Gemini proxy (`GEMINI_PRIMARY_MODEL=gemini-4-hydrogen`, `GEMINI_MODEL_FALLBACKS=gemini-4-hydrogen,gemini-3.1-flash-lite`) with adaptive 503 failover.

## 2. APK Compilation Instructions
### Debug APK (CLI)
```bash
cd android-app
./gradlew assembleDebug
# Output: app/build/outputs/apk/debug/app-debug.apk
```
### Release APK (Signed)
```bash
cd android-app
./gradlew assembleRelease
# Configure signingConfigs in app/build.gradle.kts before distributing
```

## 3. Working Native Features vs. Android Platform Restrictions
### Fully Implemented Native Capabilities
- **Voice Input & Barge-In Speech**: `SpeechRecognizer` + `TextToSpeech` with Male/Female voice selection, Urdu/Roman Urdu/Hindi/English support, and immediate TTS interruption when the user speaks.
- **Jarvis Live Foreground Service**: Explicit user-activated `FOREGROUND_SERVICE_MICROPHONE` with persistent notification.
- **Screen Reading & Navigation (`ZariyaAccessibilityService`)**: Reads visible non-password text, headings, buttons, editable fields, and performs Back, Scroll Forward/Backward, Click by label, and `ACTION_SET_TEXT`.
- **MediaProjection Screen Recording & Vision Analysis (`ScreenRecorderService`)**: User-consented screen recording (MP4 saved to `Movies/ZariyaRecordings`), pause/resume (API 24+), optional mic audio, and single-frame JPEG capture sent to Gemini Vision.
- **WhatsApp Notification Reader & Reply (`ZariyaNotificationListenerService`)**: Reads authorized WhatsApp / WhatsApp Business notifications, detects urgency via Gemini AI, enforces contact/schedule/cooldown rules, and sends replies via `RemoteInput`.
- **Call Screening & Telecom Control (`ZariyaCallScreeningService`, `CallControlHelper`)**: Screens incoming calls via `ROLE_CALL_SCREENING`, announces caller name aloud, answers/rejects/ends via `TelecomManager` (`ANSWER_PHONE_CALLS`), toggles speakerphone, and opens SMS busy composer with mandatory AI assistant disclosure.
- **Device Control (`DeviceControlManager`)**: Opens apps, searches YouTube, toggles Flashlight (`CameraManager`), adjusts volume, dispatches media keys, sets Alarms/Timers (`AlarmClock`), opens Settings/Camera/Dialer, copies to clipboard, and shares text.
- **Encrypted Local Memory (`EncryptedMemoryStore`)**: AES256-GCM `EncryptedSharedPreferences`.

### Restricted by Android OS (Handled Transparently)
1. **Background Silent Microphone / Always-On Hotword**: Android 14+ prohibits starting microphone access silently from background. Continuous listening requires explicit user activation with a visible foreground notification (`JarvisForegroundService`). Hardware-level closed-screen DSP wake words are restricted to system-privileged assistant apps.
2. **Injecting AI Voice Directly Into Cellular Call Audio**: Standard third-party Android apps cannot inject synthesized TTS audio directly into the cellular carrier voice uplink stream (`AudioTrack` -> cellular uplink is blocked without system/telecom firmware permissions). Supported alternatives implemented: `CallScreeningService` caller announcement, `TelecomManager` answer/end, speakerphone hands-free, and authorized SMS busy notice identifying itself as an AI assistant.
3. **Protected / Flagged Apps & Password Fields**: `FLAG_SECURE` banking screens, biometric prompts, and `isPassword` fields are strictly skipped and never bypassed.
4. **WhatsApp Direct Background Send Without Notification**: Android isolates WhatsApp's local encrypted database. Replying works via authorized `Notification.Action` (`RemoteInput`) when an incoming notification is active, or via user-assisted `AccessibilityService` when WhatsApp is foregrounded.
