<div align="center">

# 🎙️ Memora

### AI Voice to Calendar & Tasks

**30 seconds of voice. Zero forgotten tasks.**

![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-%E2%89%A53.5-0175C2?logo=dart&logoColor=white)
![Platforms](https://img.shields.io/badge/Platforms-iOS%20%7C%20Android-6366F1)
![AI](https://img.shields.io/badge/AI-Gemini%202.5%20Flash-4285F4?logo=google&logoColor=white)
![State](https://img.shields.io/badge/State-Riverpod-10B981)
![Status](https://img.shields.io/badge/Status-MVP%20in%20development-F59E0B)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

</div>

---

## ✨ What is Memora?

Capturing thoughts on the go is painful: unlock the phone, open a calendar, a to-do app and a notes app, then type everything out. Plain voice memos are fast, but they become a graveyard of audio files nobody listens to.

**Memora** is a single-button app. **Press and hold the mic, ramble for up to 30 seconds, release.** Within about two seconds, Google Gemini turns your unstructured speech into structured actions:

| Category | Colour | What you get |
| :-- | :-- | :-- |
| 📅 **Calendar events** | Electric Indigo `#6366F1` | Title, resolved date/time, duration, one-tap add to your native calendar |
| ✅ **To-Dos** | Emerald Mint `#10B981` | Actionable tasks with priority and an optional reminder alarm |
| 📝 **Notes** | Amber Gold `#F59E0B` | Cleaned-up thoughts with automatic tags |

### Example

> *"Set up a design sync with Sarah this Thursday at 11 AM for half an hour. Also remind me to renew the domain name tonight at 8 PM and pick up AA batteries. Oh, and a quick note for my podcast: open the next episode with the story about the 1998 server crash."*

```
📅 Calendar (1)  Design Sync with Sarah · Thu 11:00–11:30 AM        [+ Add to Calendar]
✅ To-Dos (2)    ☐ Renew domain name   ⏰ Today 8:00 PM · High
                 ☐ Pick up AA batteries · Low
📝 Notes (1)     Podcast Intro Idea — "Open the next episode with the story
                 about the 1998 server crash."  #Podcast #Content
```

---

## 🚀 Features

- **Hold-to-ramble capture** with haptics and a 30-second circular countdown ring
- **Dual audio pipelines**
  - **A – On-device STT** (`speech_to_text`): live words on screen, no audio bandwidth cost
  - **B – Multimodal audio** (`record` → Gemini, Pro): best for noise, accents and bilingual speech
- **Structured AI output** using Gemini JSON schema (no fragile text parsing)
- **Smart language understanding**: relative dates ("next Wednesday"), self-corrections ("2 PM… no, 3:30"), filler-word removal
- **Review before saving**: staging tray with quick-edit chips (`+15 min`, `Change Time`, `Toggle Priority`) and swipe-to-delete
- **Native calendar hand-off**: no invasive OAuth scopes
- **Local reminders** with a *Mark as Done* notification action
- **Searchable workspace** (All Actions / Tasks / Notes) powered by Isar
- **Offline queue**: recordings are saved and auto-processed when you reconnect
- **Dark "Obsidian" UI** (`#0B0F19` background, `#161F30` cards)

---

## 🧱 Tech Stack

| Area | Technology |
| :-- | :-- |
| Framework | Flutter (Dart `>=3.5.0 <4.0.0`) |
| State management | `flutter_riverpod`, `riverpod_annotation` |
| Models | `freezed`, `json_serializable` |
| Audio | `speech_to_text`, `record`, `audio_waveforms`, `permission_handler` |
| AI | Gemini 2.5 Flash via `firebase_ai` (production) / `google_generative_ai` (prototype) |
| Device actions | `add_2_calendar`, `flutter_local_notifications`, `timezone`, `flutter_timezone` |
| Storage | `isar` |
| Connectivity | `connectivity_plus` |
| UI | `lucide_icons`, `shimmer`, `intl` |

---

## 🏗️ Architecture

Feature-first clean architecture with Riverpod.

```
[Hold Mic] ──> [Voice Ramble ≤30s] ──> [Release]
                                           │
                                           ▼
[Local DB (Isar)] <── [Staging Tray UI] <── [Gemini JSON Extraction]
       │                     │
       ▼                     ▼
 (To-Dos & Notes)   (Native Calendar Sheet)
```

```text
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/app_colors.dart
│   ├── services/ (notification, calendar_launcher, connectivity)
│   └── utils/date_time_helper.dart
└── features/
    ├── recorder/    (data · domain · presentation)
    └── workspace/   (data · domain · presentation)
```

---

## ⚙️ Getting Started

### Prerequisites

- Flutter SDK (stable) and Dart `>=3.5.0`
- Xcode (for iOS) and/or Android Studio with an Android SDK
- A [Gemini API key](https://aistudio.google.com/app/apikey) for local prototyping
- (Production) a Firebase project with **Firebase AI Logic** and **App Check** enabled

### Installation

```bash
# 1. Clone
git clone https://github.com/masumtanjim7/Memora.git
cd Memora

# 2. Install dependencies
flutter pub get

# 3. Generate code (Isar, Freezed, JSON)
dart run build_runner build --delete-conflicting-outputs
```

### Run (prototype mode, direct API key)

```bash
flutter run --dart-define=GEMINI_API_KEY=your_key_here
```

> ⚠️ Never commit API keys. The direct-key mode is for local development only.

### Run (production mode, Firebase AI Logic)

```bash
dart pub global activate flutterfire_cli
flutterfire configure --project=<your-firebase-project>
flutter run --release
```

### Creating the project from scratch (first-time setup)

```bash
flutter create --org com.spideytech --project-name memora --platforms=android,ios memora
```

---

## 🔐 Permissions

| Platform | Permissions |
| :-- | :-- |
| **iOS** (`Info.plist`) | `NSMicrophoneUsageDescription`, `NSSpeechRecognitionUsageDescription`, `NSCalendarsUsageDescription`, `NSCalendarsFullAccessUsageDescription` |
| **Android** (`AndroidManifest.xml`) | `RECORD_AUDIO`, `INTERNET`, `ACCESS_NETWORK_STATE`, `READ_CALENDAR`, `WRITE_CALENDAR`, `POST_NOTIFICATIONS`, `SCHEDULE_EXACT_ALARM`, `RECEIVE_BOOT_COMPLETED`, `VIBRATE` |

---

## 🧪 Testing

```bash
flutter analyze
flutter test --coverage
flutter test integration_test
```

The AI layer ships with a 10-transcript evaluation suite (relative dates, self-corrections, filler words, ambiguous times, silence, code-switching) to guard extraction quality.

---

## 🗺️ Roadmap

- [ ] **Week 1** – Audio capture & UI shell (`PulsingMicButton`, STT + `record`, permissions)
- [ ] **Week 2** – Gemini structured extraction, evaluation suite, staging sheet
- [ ] **Week 3** – Isar database, native calendar hand-off, local notifications
- [ ] **Week 4** – Offline queue, Firebase AI Logic + App Check, onboarding, store assets

**Post-MVP:** Pro tier (2-minute raw-audio mode, bilingual support), Google Calendar / Notion / Todoist sync, home & lock-screen widgets, recurring items, location reminders.

---

## 💎 Pricing Plan (planned)

| | Free | Pro – $3.99/mo |
| :-- | :-- | :-- |
| Recording length | 30 s | up to 2 min |
| AI extractions | 10 / day | Fair-use unlimited |
| Audio pipeline | On-device STT | + Multimodal raw audio |
| Integrations | Native calendar | Google Calendar, Notion, Todoist |
| Widgets | – | Home & lock screen |

---

## 🛡️ Privacy

- No account required in the MVP
- Tasks and notes are stored locally on your device
- Only the current memo's transcript/audio is sent to Google Gemini for processing; audio is deleted after successful processing
- No API keys are shipped in production builds (Firebase AI Logic + App Check)

---

## 🤝 Contributing

1. Open an issue to discuss your idea
2. Branch from `develop`: `feature/<short-name>`
3. Use [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:` …)
4. Ensure `flutter analyze` and `flutter test` pass
5. Open a pull request

---

## 📚 Documentation

Full product, architecture and release documentation: [`docs/Memora_Project_Documentation.docx`](docs/Memora_Project_Documentation.docx)

---

## 📄 License

Released under the MIT License (change this section if you choose a different license).

---

<div align="center">

**Memora** · Built with ❤️ by **SpideyTech**
`com.spideytech.memora`

</div>
