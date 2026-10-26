<div align="center">

# Memora
### AI Voice to Calendar & Tasks
**30 seconds of voice. Zero forgotten tasks.**
</div>


## 📑 Table of Contents

1. [What is Memora?](#-what-is-memora)
2. [Features](#-features)
3. [System Context](#1-system-context-diagram)
4. [End-to-End User Flow](#2-end-to-end-user-flow)
5. [Capture Pipelines A vs B](#3-capture-pipelines-a-vs-b)
6. [Full Sequence Diagram](#4-full-happy-path-sequence)
7. [Recorder State Machine](#5-recorder-state-machine)
8. [Layered Architecture](#6-layered-architecture)
9. [State Management Graph](#7-riverpod-provider-graph)
10. [ER Diagram](#8-er-diagram-local-database)
11. [Domain Class Diagram](#9-domain-class-diagram)
12. [Data Lifecycle](#10-data-lifecycle)
13. [Offline Queue](#11-offline-queue--auto-sync)
14. [Error Handling](#12-error-handling-decision-flow)
15. [Reminders & Calendar](#13-reminders--calendar-hand-off)
16. [Permissions Flow](#14-permissions-flow)
17. [Security Architecture](#15-security-architecture)
18. [UI Navigation & Wireframes](#16-ui-navigation-map--wireframes)
19. [Monetisation Gating](#17-monetisation--entitlement-flow)
20. [Roadmap & Plan](#18-development-roadmap)
21. [CI/CD & Release](#19-cicd--release-pipeline)
22. [Git Workflow](#20-git-workflow)
23. [Getting Started](#-getting-started)

> 💡 All diagrams are written in [Mermaid](https://mermaid.js.org/) and render natively on GitHub.

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

```mermaid
mindmap
  root((Memora))
    Capture
      Hold to record
      30s countdown ring
      Haptic feedback
      Pipeline A on-device STT
      Pipeline B raw audio Pro
    AI
      Gemini 2.5 Flash
      Structured JSON schema
      Relative dates
      Self-correction handling
      Filler word removal
    Review
      Staging tray
      Quick edit chips
      Swipe to delete
      Shimmer loading
    Actions
      Native calendar hand-off
      Local reminders
      Mark as Done
    Storage
      Isar local DB
      Full text search
      Offline queue
    Business
      Free 10 per day
      Pro 3.99 per month
      Integrations
      Widgets
```

- **Hold-to-ramble capture** with haptics and a 30-second circular countdown ring
- **Dual audio pipelines:** on-device STT (live words) and multimodal raw audio (Pro, noisy/bilingual)
- **Structured AI output** using Gemini JSON schema, with no fragile text parsing
- **Smart understanding:** relative dates, self-corrections, filler-word removal
- **Review before saving:** quick-edit chips and swipe-to-delete
- **Native calendar hand-off** (no invasive OAuth) and **local reminders** with *Mark as Done*
- **Searchable workspace** (All Actions / Tasks / Notes) powered by Isar
- **Offline queue** that auto-processes recordings when you reconnect
- **Dark "Obsidian" UI** (`#0B0F19` background, `#161F30` cards)

---

# 🧭 Architecture & Plan: Everything in Diagrams

## 1. System Context Diagram

Who and what Memora talks to.

```mermaid
flowchart LR
    U(["👤 User"])

    subgraph Device["📱 User Device (iOS / Android)"]
        direction TB
        APP["Memora Flutter App"]
        DB[("Isar Local DB")]
        STT["OS Speech Recognizer"]
        MIC["Microphone"]
        CAL["Native Calendar App"]
        NOTIF["OS Notification Scheduler"]
    end

    subgraph Cloud["☁️ Google Cloud / Firebase"]
        direction TB
        APPCHECK["Firebase App Check"]
        AILOGIC["Firebase AI Logic"]
        GEMINI["Gemini 2.5 Flash"]
        ANALYTICS["Analytics + Crashlytics"]
    end

    STORE["🛒 App Store / Play Store Billing"]

    U -->|"hold and speak"| MIC
    MIC --> APP
    MIC --> STT
    STT -->|"live transcript"| APP
    APP -->|"attest request"| APPCHECK
    APP -->|"transcript or audio + time context"| AILOGIC
    AILOGIC --> GEMINI
    GEMINI -->|"structured JSON"| AILOGIC
    AILOGIC -->|"ExtractionResult"| APP
    APP <-->|"save / query"| DB
    APP -->|"pre-filled event"| CAL
    APP -->|"schedule reminders"| NOTIF
    NOTIF -->|"alarm at due time"| U
    APP -.->|"anonymous events"| ANALYTICS
    APP <-.->|"entitlements"| STORE

    classDef app fill:#6366F1,stroke:#312E81,color:#fff;
    classDef cloud fill:#E0E7FF,stroke:#6366F1,color:#111;
    classDef store fill:#FEF3C7,stroke:#F59E0B,color:#111;
    class APP app;
    class APPCHECK,AILOGIC,GEMINI,ANALYTICS cloud;
    class DB,STORE store;
```

---

## 2. End-to-End User Flow

```mermaid
flowchart TD
    A(["Open Memora"]) --> B{"First launch?"}
    B -->|"Yes"| C["Onboarding + permission walkthrough"]
    B -->|"No"| D["Home / Capture screen"]
    C --> D
    D --> E["Press and HOLD mic button 🎙️"]
    E --> F["Haptic + countdown ring + live transcript/waveform"]
    F --> G{"Released or 30s reached?"}
    G -->|"held under 1.5s"| H["Discard + toast: Hold the mic while speaking"]
    H --> D
    G -->|"valid recording"| I["Staging sheet opens with shimmer"]
    I --> J{"Online?"}
    J -->|"No"| K["Save as pendingAiSync + badge"]
    K --> D
    J -->|"Yes"| L["Gemini extracts JSON"]
    L --> M{"Any actions found?"}
    M -->|"No"| N["Show: Couldn't catch any actions"]
    N --> D
    M -->|"Yes"| O["Render Calendar / To-Do / Note cards"]
    O --> P["User reviews: edit chips, swipe to discard"]
    P --> Q{"Calendar event?"}
    Q -->|"Tap Add to Calendar"| R["Native calendar sheet pre-filled"]
    Q -->|"skip"| S
    R --> S["Tap Confirm & Save All"]
    S --> T["Save To-Dos + Notes to Isar"]
    T --> U["Schedule local reminders"]
    U --> V(["Done ✅ back to Home"])
    V --> D

    classDef cal fill:#6366F1,color:#fff,stroke:#312E81;
    classDef todo fill:#10B981,color:#fff,stroke:#065F46;
    classDef note fill:#F59E0B,color:#111,stroke:#92400E;
    class R cal;
    class T,U todo;
    class O note;
```

---

## 3. Capture Pipelines A vs B

```mermaid
flowchart LR
    subgraph A["Pipeline A: On-device STT (Free + Pro)"]
        direction TB
        A1["Hold mic"] --> A2["speech_to_text streams words"]
        A2 --> A3["Live transcript on screen"]
        A3 --> A4["Release: final text string"]
    end

    subgraph B["Pipeline B: Multimodal audio (Pro)"]
        direction TB
        B1["Hold mic"] --> B2["record: AAC .m4a at 64 kbps"]
        B2 --> B3["audio_waveforms visualiser"]
        B3 --> B4["Release: raw audio bytes"]
    end

    A4 -->|"Content.text(prompt + transcript)"| G["GeminiExtractionService"]
    B4 -->|"Content.multi(prompt + audio/mp4 bytes)"| G
    G --> R["ExtractionResult JSON"]

    classDef pa fill:#DBEAFE,stroke:#3B82F6,color:#111;
    classDef pb fill:#FCE7F3,stroke:#EC4899,color:#111;
    class A1,A2,A3,A4 pa;
    class B1,B2,B3,B4 pb;
```

| | Pipeline A | Pipeline B (Pro) |
| :-- | :-- | :-- |
| Package | `speech_to_text` | `record` + `audio_waveforms` |
| Sent to Gemini | Transcript text | Raw `.m4a` (`audio/mp4`) |
| Best for | Instant feedback, zero audio bandwidth | Noise, heavy accents, bilingual speech |
| Max length | 30 s | 2 min |

---

## 4. Full Happy-Path Sequence

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant UI as PulsingMicButton / HomeScreen
    participant RC as RecorderController (Riverpod)
    participant AR as AudioRecorderRepository
    participant GS as GeminiExtractionService
    participant AC as App Check + AI Logic
    participant GM as Gemini 2.5 Flash
    participant ST as StagingTraySheet
    participant LR as LocalMemoRepository (Isar)
    participant NS as NotificationService
    participant CL as CalendarLauncherService

    User->>UI: long-press start
    UI->>RC: startRecording()
    RC->>AR: check permission and start (STT or record)
    AR-->>UI: live transcript / amplitude stream
    User->>UI: release
    UI->>RC: stopRecording()
    RC->>AR: stop and return transcript or bytes
    alt duration under 1.5 s
        RC-->>UI: discard + toast
    else valid recording
        RC->>ST: open sheet (shimmer)
        RC->>GS: process(transcript or audio)
        GS->>GS: build prompt with now, weekday, timezone
        GS->>AC: generateContent + App Check token
        AC->>GM: schema-constrained request
        GM-->>AC: JSON (calendar_events, todos, notes)
        AC-->>GS: response
        GS-->>RC: ExtractionResult
        RC->>ST: render cards
        User->>ST: edit chips / swipe to discard
        opt user taps Add to Calendar
            ST->>CL: addToCalendar(event)
            CL-->>User: native calendar sheet
        end
        User->>ST: Confirm and Save All
        ST->>LR: save todos, notes, memo log
        LR-->>ST: saved
        ST->>NS: schedule reminders for todos with due
        NS-->>User: notification fires at due time
    end
```

---

## 5. Recorder State Machine

```mermaid
stateDiagram-v2
    [*] --> idle
    idle --> recording: long press start
    recording --> idle: release under 1.5s (discard)
    recording --> processing: release or 30s timeout
    processing --> staging: valid result
    processing --> idle: empty result (hint shown)
    processing --> error: network or parse failure
    processing --> queued: offline
    queued --> processing: connectivity restored
    error --> processing: retry
    error --> queued: save for later
    error --> idle: dismiss
    staging --> idle: discard all
    staging --> saving: Confirm and Save All
    saving --> idle: saved + reminders scheduled

    note right of recording
        haptic, countdown ring,
        live transcript or waveform
    end note
    note right of processing
        shimmer skeleton,
        max 12 s timeout, 1 retry
    end note
```

---

## 6. Layered Architecture

Feature-first clean architecture. Dependencies point **inward** (Presentation → Domain ← Data).

```mermaid
flowchart TB
    subgraph PRES["🎨 Presentation Layer"]
        direction LR
        S1["HomeRecorderScreen"]
        S2["WorkspaceTabsScreen"]
        W1["PulsingMicButton"]
        W2["StagingTraySheet"]
        W3["EventActionCard / TodoTile / NoteGridCard"]
        CT["RecorderController (Notifier)"]
    end

    subgraph DOM["🧠 Domain Layer (pure Dart)"]
        direction LR
        M1["ExtractionResult"]
        M2["ExtractedCalendarEvent"]
        M3["ExtractedTodo"]
        M4["ExtractedNote"]
        M5["SavedTodo / SavedNote / VoiceMemoLog"]
    end

    subgraph DATA["💾 Data Layer"]
        direction LR
        D1["AudioRecorderRepository"]
        D2["GeminiExtractionService"]
        D3["LocalMemoRepository"]
    end

    subgraph CORE["⚙️ Core"]
        direction LR
        C1["NotificationService"]
        C2["CalendarLauncherService"]
        C3["ConnectivityService"]
        C4["AppColors / DateTimeHelper"]
    end

    subgraph EXT["🔌 External"]
        direction LR
        E1["speech_to_text / record"]
        E2["firebase_ai + App Check"]
        E3["Isar"]
        E4["add_2_calendar"]
        E5["flutter_local_notifications"]
        E6["connectivity_plus"]
    end

    PRES --> DOM
    CT --> DATA
    CT --> CORE
    DATA --> DOM
    D1 --> E1
    D2 --> E2
    D3 --> E3
    C2 --> E4
    C1 --> E5
    C3 --> E6

    classDef pres fill:#6366F1,color:#fff,stroke:#312E81;
    classDef dom fill:#10B981,color:#fff,stroke:#065F46;
    classDef data fill:#F59E0B,color:#111,stroke:#92400E;
    classDef core fill:#E5E7EB,color:#111,stroke:#6B7280;
    class S1,S2,W1,W2,W3,CT pres;
    class M1,M2,M3,M4,M5 dom;
    class D1,D2,D3 data;
    class C1,C2,C3,C4 core;
```

### Folder structure

```text
lib/
├── main.dart
├── app.dart
├── core/
│   ├── constants/app_colors.dart
│   ├── services/ (notification, calendar_launcher, connectivity)
│   └── utils/date_time_helper.dart
└── features/
    ├── recorder/
    │   ├── data/         audio_recorder_repository, gemini_extraction_service
    │   ├── domain/       extraction_result_model
    │   └── presentation/ controllers, screens, widgets (pulsing_mic_button, staging_tray_sheet)
    └── workspace/
        ├── data/         local_memo_repository
        ├── domain/       calendar_item, todo_item, note_item
        └── presentation/ screens, widgets (event_action_card, todo_tile, note_grid_card)
```

---

## 7. Riverpod Provider Graph

```mermaid
flowchart LR
    SET["settingsProvider"] --> RC
    CONN["connectivityProvider (Stream)"] --> RC
    GEM["geminiServiceProvider"] --> RC
    REC["audioRecorderProvider"] --> RC
    REPO["localMemoRepoProvider"] --> RC
    NOTI["notificationServiceProvider"] --> RC
    CAL["calendarLauncherProvider"] --> RC

    RC["recorderControllerProvider (Notifier)"] --> HOME["HomeRecorderScreen"]
    RC --> STG["StagingTraySheet"]

    REPO --> PEND["pendingTodoCountProvider (Stream)"]
    REPO --> TODOS["todosProvider (filter + search)"]
    REPO --> NOTES["notesProvider (filter + search)"]
    REPO --> QUEUE["pendingMemosProvider"]

    PEND --> HOME
    QUEUE --> HOME
    TODOS --> WS["WorkspaceTabsScreen"]
    NOTES --> WS

    classDef p fill:#EEF2FF,stroke:#6366F1,color:#111;
    classDef ui fill:#6366F1,color:#fff,stroke:#312E81;
    class SET,CONN,GEM,REC,REPO,NOTI,CAL,PEND,TODOS,NOTES,QUEUE,RC p;
    class HOME,STG,WS ui;
```

---

## 8. ER Diagram (Local Database)

Isar stores everything on-device. `VoiceMemoLog` is the **source record**; every saved To-Do, Note and Event log item links back to it via `sourceMemoUuid`. Tags are stored as an embedded list on `SavedNote` and shown here as a logical `NOTE_TAG` entity.

```mermaid
erDiagram
    VOICE_MEMO_LOG ||--o{ SAVED_TODO : "produces"
    VOICE_MEMO_LOG ||--o{ SAVED_NOTE : "produces"
    VOICE_MEMO_LOG ||--o{ SAVED_EVENT_LOG : "produces"
    SAVED_NOTE ||--o{ NOTE_TAG : "has 1 to 3"
    SAVED_TODO ||--o| SCHEDULED_REMINDER : "triggers"
    APP_SETTINGS ||--o| ENTITLEMENT : "gates features"
    ENTITLEMENT ||--o{ USAGE_COUNTER : "limits"

    VOICE_MEMO_LOG {
        int id PK "Isar auto id"
        string uuid UK "unique, replace on conflict"
        datetime recordedAt
        string transcript "nullable, Pipeline A"
        string audioPath "nullable, offline Pipeline B only"
        enum status "pendingAiSync, processed, failed"
        string errorMessage "nullable"
    }

    SAVED_TODO {
        int id PK
        string uuid UK
        string task "indexed, case-insensitive"
        datetime due "nullable"
        enum priority "low, medium, high"
        bool isCompleted
        int notificationId "nullable"
        string sourceMemoUuid FK
        datetime createdAt
    }

    SAVED_NOTE {
        int id PK
        string uuid UK
        string title "indexed"
        string content
        list tags "indexed list of strings"
        string sourceMemoUuid FK
        datetime createdAt
    }

    NOTE_TAG {
        string name "logical entity, embedded list"
        string noteUuid FK
    }

    SAVED_EVENT_LOG {
        int id PK
        string uuid UK
        string title
        datetime startTime
        int durationMinutes
        string description
        bool handedOffToCalendar
        string sourceMemoUuid FK
    }

    SCHEDULED_REMINDER {
        int notificationId PK "managed by OS scheduler"
        datetime fireAt
        string payloadTodoUuid FK
    }

    APP_SETTINGS {
        int id PK
        enum pipeline "onDeviceStt, rawAudio"
        string language
        enum theme "dark, light"
        bool onboardingDone
    }

    ENTITLEMENT {
        int id PK
        enum tier "free, pro"
        datetime expiresAt "nullable"
    }

    USAGE_COUNTER {
        int id PK
        date day
        int aiExtractions "free limit 10"
    }
```

> `SAVED_EVENT_LOG`, `APP_SETTINGS`, `ENTITLEMENT` and `USAGE_COUNTER` are lightweight MVP-adjacent collections. Calendar events themselves live in the user's **native calendar**, not in Memora's database.

---

## 9. Domain Class Diagram

Transient AI models (parsed from Gemini JSON) and persistent models (Isar).

```mermaid
classDiagram
    direction LR

    class ExtractionResult {
        +List~ExtractedCalendarEvent~ calendarEvents
        +List~ExtractedTodo~ todos
        +List~ExtractedNote~ notes
        +bool isEmpty
        +fromJson(Map json) ExtractionResult
    }
    class ExtractedCalendarEvent {
        +String title
        +DateTime startTime
        +int durationMinutes
        +String description
    }
    class ExtractedTodo {
        +String task
        +DateTime? due
        +String priority
        +bool isCompleted
    }
    class ExtractedNote {
        +String title
        +String content
        +List~String~ tags
    }
    class SavedTodo {
        +Id id
        +String uuid
        +String task
        +DateTime? due
        +Priority priority
        +bool isCompleted
        +int? notificationId
    }
    class SavedNote {
        +Id id
        +String uuid
        +String title
        +String content
        +List~String~ tags
    }
    class VoiceMemoLog {
        +Id id
        +String uuid
        +DateTime recordedAt
        +MemoStatus status
    }
    class GeminiExtractionService {
        +processTranscript(String) ExtractionResult
        +processAudioBytes(Uint8List) ExtractionResult
    }
    class LocalMemoRepository {
        +saveAll(ExtractionResult, memoUuid)
        +watchPendingTodoCount() Stream~int~
        +search(String query)
    }
    class NotificationService {
        +scheduleTodoReminder(SavedTodo)
        +cancel(int id)
    }
    class CalendarLauncherService {
        +addToCalendar(ExtractedCalendarEvent) bool
    }

    ExtractionResult "1" o-- "*" ExtractedCalendarEvent
    ExtractionResult "1" o-- "*" ExtractedTodo
    ExtractionResult "1" o-- "*" ExtractedNote
    GeminiExtractionService ..> ExtractionResult : returns
    LocalMemoRepository ..> SavedTodo : persists
    LocalMemoRepository ..> SavedNote : persists
    LocalMemoRepository ..> VoiceMemoLog : persists
    ExtractedTodo ..> SavedTodo : maps to
    ExtractedNote ..> SavedNote : maps to
    NotificationService ..> SavedTodo : schedules
    CalendarLauncherService ..> ExtractedCalendarEvent : opens sheet
```

### AI JSON contract

```json
{
  "calendar_events": [
    { "title": "Design Sync with Sarah",
      "start_time": "2026-10-29T11:00:00+06:00",
      "duration_minutes": 30,
      "description": "Design sync with Sarah." }
  ],
  "todos": [
    { "task": "Renew domain name", "due": "2026-10-26T20:00:00+06:00", "priority": "high" },
    { "task": "Pick up AA batteries", "due": null, "priority": "low" }
  ],
  "notes": [
    { "title": "Podcast Intro Idea",
      "content": "Open the next episode with the story about the 1998 server crash.",
      "tags": ["Podcast", "Content"] }
  ]
}
```

---

## 10. Data Lifecycle

How raw voice becomes durable data, and what is deleted along the way.

```mermaid
flowchart LR
    V["🎙️ Voice"] --> T["Transcript (RAM)<br/>or .m4a temp file"]
    T --> P["Prompt + time context"]
    P --> G["Gemini"]
    G --> X["ExtractionResult<br/>(transient, in RAM)"]
    X --> E["Staging edits<br/>(user can change or discard)"]
    E -->|"Confirm and Save All"| DB[("Isar<br/>SavedTodo, SavedNote, VoiceMemoLog")]
    E -->|"Add to Calendar"| CAL["Native Calendar<br/>(owned by OS)"]
    DB --> N["OS notification<br/>scheduler"]
    T -.->|"deleted after success"| DEL["🗑️ audio removed"]
    E -.->|"discarded items"| GONE["never stored"]

    classDef temp fill:#FEF3C7,stroke:#F59E0B,color:#111;
    classDef keep fill:#D1FAE5,stroke:#10B981,color:#111;
    class T,X,E,DEL,GONE temp;
    class DB,CAL,N keep;
```

---

## 11. Offline Queue & Auto-Sync

```mermaid
flowchart TD
    A["Recording finished"] --> B{"Internet available?"}
    B -->|"Yes"| C["Call Gemini"]
    B -->|"No"| D["Save VoiceMemoLog<br/>status = pendingAiSync<br/>(transcript or .m4a path)"]
    D --> E["Show badge on Home top bar"]
    E --> F["ConnectivityService listens<br/>(connectivity_plus)"]
    F --> G{"Connection restored?"}
    G -->|"No"| F
    G -->|"Yes"| H["Process queue oldest first"]
    H --> I{"Gemini success?"}
    I -->|"Yes"| J["status = processed<br/>show staging tray / notification<br/>delete audio file"]
    I -->|"Failure after 1 retry"| K["status = failed<br/>keep transcript<br/>manual retry button"]
    C --> I
    J --> L(["Queue empty, badge cleared"])
    K --> L
```

---

## 12. Error Handling Decision Flow

```mermaid
flowchart TD
    S(["Memo released"]) --> A{"Duration >= 1.5 s?"}
    A -->|"No"| A1["Discard buffer<br/>toast: Hold the mic button while speaking"]
    A -->|"Yes"| B{"Mic permission granted?"}
    B -->|"No"| B1["Explainer + Open Settings button"]
    B -->|"Yes"| C{"Daily quota left? (Free: 10)"}
    C -->|"No"| C1["Save transcript locally<br/>show upgrade sheet"]
    C -->|"Yes"| D{"Online?"}
    D -->|"No"| D1["Queue as pendingAiSync"]
    D -->|"Yes"| E["Call Gemini (12 s timeout)"]
    E --> F{"Response valid JSON?"}
    F -->|"No / timeout"| F1{"Retried already?"}
    F1 -->|"No"| E
    F1 -->|"Yes"| F2["status = failed<br/>keep transcript, manual retry"]
    F -->|"Yes"| G{"All arrays empty?"}
    G -->|"Yes"| G1["Couldn't catch any actions,<br/>try speaking closer to the mic"]
    G -->|"No"| H{"Ambiguous time?"}
    H -->|"Yes"| H1["Becomes high-priority To-Do<br/>never a guessed calendar time"]
    H -->|"No"| I["Render staging cards"]
    H1 --> I
    I --> J{"Due date in the past?"}
    J -->|"Yes"| J1["Save without alarm<br/>show Past due chip"]
    J -->|"No"| K["Schedule reminder"]
```

---

## 13. Reminders & Calendar Hand-off

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant ST as StagingTraySheet
    participant LR as LocalMemoRepository
    participant NS as NotificationService
    participant TZ as timezone + flutter_timezone
    participant OS as OS Notification Scheduler
    participant CL as CalendarLauncherService
    participant CA as Native Calendar App

    Note over ST,CA: Calendar events (hand-off, no OAuth)
    User->>ST: tap Add to Calendar
    ST->>CL: addToCalendar(event)
    CL->>CA: Add2Calendar.addEvent2Cal(title, start, end, description)
    CA-->>User: pre-filled editor, user taps Save

    Note over ST,OS: To-Do reminders
    User->>ST: Confirm and Save All
    ST->>LR: persist SavedTodo
    LR-->>ST: ok
    ST->>NS: scheduleTodoReminder(todo)
    NS->>TZ: tz.local from device IANA zone
    alt due is null or in the past
        NS-->>ST: skip scheduling
    else due in the future
        NS->>OS: zonedSchedule(id, due, Mark as Done action)
    end
    OS-->>User: notification at due time
    User->>OS: tap Mark as Done
    OS->>LR: set isCompleted = true
    LR->>NS: cancel(notificationId)
```

---

## 14. Permissions Flow

```mermaid
flowchart TD
    A(["Onboarding / first use"]) --> M{"Microphone"}
    M -->|"Granted"| N{"Notifications"}
    M -->|"Denied"| M1["Explain value + Open Settings<br/>mic button disabled with tooltip"]
    N -->|"Granted"| X{"Android 12+ exact alarm?"}
    N -->|"Denied"| N1["Tasks saved without reminders<br/>show hint banner"]
    X -->|"Allowed"| R(["Ready: full features"])
    X -->|"Denied"| X1["Fallback: inexactAllowWhileIdle"]
    X1 --> R
    N1 --> R

    R --> S{"Using Pipeline A on iOS?"}
    S -->|"Yes"| S1{"Speech recognition permission"}
    S1 -->|"Denied"| S2["Offer Pipeline B or Settings"]
    S1 -->|"Granted"| OK(["Capture enabled"])
    S -->|"No"| OK
```

---

## 15. Security Architecture

No API keys ship inside the production app.

```mermaid
sequenceDiagram
    autonumber
    participant App as Memora App
    participant AC as Firebase App Check
    participant PI as Play Integrity / App Attest
    participant AL as Firebase AI Logic (proxy)
    participant GM as Gemini API

    App->>PI: request attestation
    PI-->>App: integrity verdict
    App->>AC: exchange for App Check token
    AC-->>App: short-lived token
    App->>AL: generateContent + App Check token
    AL->>AL: verify token, enforce per-user quota
    alt token invalid or quota exceeded
        AL-->>App: 401 / 429
    else valid
        AL->>GM: forward request with server-side credentials
        GM-->>AL: structured JSON
        AL-->>App: response
    end
```

```mermaid
flowchart LR
    subgraph DEV["Development"]
        K1["--dart-define GEMINI_API_KEY<br/>(local only, never committed)"]
    end
    subgraph PROD["Production build"]
        K2["Firebase AI Logic<br/>no key in binary"]
        K3["App Check enforced"]
        K4["Budget alerts + quotas"]
    end
    DEV -->|"migrate in Week 4 and rotate old key"| PROD
    K2 --- K3 --- K4
```

**Privacy principles:** local-first storage · audio deleted after processing · no transcripts in analytics · "Delete all data" in settings · AI-processing disclosure in privacy policy.

---

## 16. UI Navigation Map & Wireframes

```mermaid
flowchart LR
    ONB["Onboarding<br/>3 steps"] --> HOME
    HOME["🏠 Home / Capture"] -->|"release mic"| STG["📋 Staging Bottom Sheet"]
    STG -->|"Confirm and Save All"| HOME
    STG -->|"Add to Calendar"| NCAL["Native Calendar Sheet"]
    NCAL --> STG
    HOME -->|"workspace tab"| WS["🗂️ Workspace"]
    WS --> ALL["All Actions"]
    WS --> TASKS["Tasks"]
    WS --> NOTES["Notes"]
    HOME --> SET["⚙️ Settings"]
    SET -->|"upgrade"| PAY["💎 Pro Paywall"]
    HOME -.->|"quota reached"| PAY
```

### Wireframes (text)

```
┌─────────────────────────┐   ┌─────────────────────────┐   ┌─────────────────────────┐
│ ✅ 3 tasks · 📅 2 today  │   │ ▔▔▔▔▔  (drag handle)    │   │ [All] [Tasks] [Notes]   │
│ ☁️ 1 pending sync        │   │ 📅 CALENDAR (1)         │   │ 🔍 search…              │
│                         │   │ Design Sync with Sarah  │   │ ☐ Renew domain   ⏰ 8PM │
│   "Set up a design      │   │ Thu 11:00–11:30  [+Cal] │   │ ☐ Pick up AA batteries  │
│    sync with Sarah…"    │   │ [+15 min] [Change time] │   │ ─────────────────────── │
│   ~~~~ waveform ~~~~    │   │ ✅ TO-DOS (2)           │   │ ┌──────────┐┌─────────┐ │
│                         │   │ ☐ Renew domain · High   │   │ │Podcast   ││ Idea…   │ │
│       ╭───────╮         │   │ ☐ Pick up AA · Low      │   │ │Intro #Pod││ #UI     │ │
│       │  🎙️   │ 28s ◔   │   │ 📝 NOTES (1)            │   │ └──────────┘└─────────┘ │
│       ╰───────╯         │   │ [   Confirm & Save All ]│   │                         │
└─────────────────────────┘   └─────────────────────────┘   └─────────────────────────┘
      Home / Capture                Staging Sheet                  Workspace
```

---

## 17. Monetisation & Entitlement Flow

```mermaid
flowchart TD
    A["User starts recording"] --> B{"Tier?"}
    B -->|"Free"| C{"AI extractions today < 10?"}
    B -->|"Pro"| P["Allow up to 2 min<br/>Pipeline A or B"]
    C -->|"Yes"| D["Allow up to 30 s<br/>Pipeline A only"]
    C -->|"No"| E["Save transcript locally<br/>show upgrade sheet"]
    D --> F["Extract and increment counter"]
    P --> F
    E --> G{"Upgrade?"}
    G -->|"Yes"| H["Store billing via RevenueCat / in_app_purchase"]
    H --> I["Entitlement = Pro"]
    I --> P
    G -->|"No"| J(["Continue free / manual note"])
```

| | Free | Pro: $3.99 / month |
| :-- | :-- | :-- |
| Recording length | 30 s | up to 2 min |
| AI extractions | 10 / day | Fair-use unlimited |
| Audio pipeline | On-device STT | + Multimodal raw audio |
| Integrations | Native calendar hand-off | Google Calendar, Notion, Todoist (OAuth) |
| Widgets | n/a | Home & lock screen (`home_widget`) |

---

## 18. Development Roadmap

```mermaid
gantt
    title Memora 4-Week MVP Plan
    dateFormat YYYY-MM-DD
    axisFormat %d %b
    section Week 1 Capture and UI
    Project init, Riverpod, dark theme       :w1a, 2026-10-26, 2d
    PulsingMicButton and 30s ring            :w1b, after w1a, 2d
    speech_to_text + record + permissions    :w1c, after w1b, 3d
    section Week 2 AI Extraction
    GeminiExtractionService and schema       :w2a, 2026-11-02, 3d
    10-transcript evaluation suite           :w2b, after w2a, 2d
    Staging sheet and shimmer                :w2c, after w2b, 2d
    section Week 3 Native and DB
    Isar collections                         :w3a, 2026-11-09, 2d
    add_2_calendar hand-off                  :w3b, after w3a, 2d
    Local notifications and Mark as Done     :w3c, after w3b, 3d
    section Week 4 Polish and Release
    Offline queue (connectivity_plus)        :w4a, 2026-11-16, 2d
    Firebase AI Logic + App Check migration  :w4b, after w4a, 2d
    Onboarding, icons, store assets          :w4c, after w4b, 2d
    Beta (TestFlight / Play Internal)        :milestone, w4d, after w4c, 1d
    section Buffer
    Store review and bug fixing              :w5, 2026-11-23, 7d
```

| Week | Focus | Exit criteria |
| :-- | :-- | :-- |
| **1** | Audio capture & UI shell | Hold-to-record works on both OSs; live transcript visible; permissions tested |
| **2** | Gemini structured extraction | ≥ 9/10 eval transcripts pass; staging sheet renders all categories |
| **3** | Native actions & local DB | Speak → review → save → reminder fires on time |
| **4** | Offline, security & release prep | No API key in binary; offline memo auto-syncs; beta builds uploaded |

### Post-MVP expansion

```mermaid
flowchart LR
    MVP(["MVP v1.0"]) --> V11["v1.1<br/>Pro tier + raw audio mode"]
    V11 --> V12["v1.2<br/>Widgets + shortcuts (Siri / Assistant)"]
    V12 --> V13["v1.3<br/>Google Calendar, Notion, Todoist sync"]
    V13 --> V20["v2.0<br/>Recurring items, location reminders,<br/>daily digest, team workspaces"]
```

---

## 19. CI/CD & Release Pipeline

```mermaid
flowchart LR
    DEV["Developer push / PR"] --> GH["GitHub Actions"]
    subgraph CI["Continuous Integration"]
        direction TB
        A1["flutter pub get"] --> A2["build_runner"]
        A2 --> A3["flutter analyze"]
        A3 --> A4["flutter test --coverage"]
        A4 --> A5["Nightly: AI eval suite vs live Gemini"]
    end
    GH --> CI
    CI -->|"green + review"| MERGE["Merge to develop"]
    MERGE --> REL["release/x.y.z branch"]
    REL --> BA["Build .aab (Android)"]
    REL --> BI["Build .ipa (iOS)"]
    BA --> IT["Play Internal Testing"]
    BI --> TF["TestFlight"]
    IT --> PROD["Production release"]
    TF --> PROD
    PROD --> MON["Crashlytics + Analytics + AI cost alerts"]
```

---

## 20. Git Workflow

```mermaid
gitGraph
    commit id: "init memora"
    branch develop
    checkout develop
    commit id: "design system"
    branch feature/mic-button
    commit id: "PulsingMicButton"
    commit id: "countdown ring"
    checkout develop
    merge feature/mic-button
    branch feature/gemini-extraction
    commit id: "schema + service"
    commit id: "eval suite"
    checkout develop
    merge feature/gemini-extraction
    branch release/1.0.0
    commit id: "store assets"
    checkout main
    merge release/1.0.0 tag: "v1.0.0"
```

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

## ⚙️ Getting Started

### Prerequisites

- Flutter SDK (stable) and Dart `>=3.5.0`
- Xcode (iOS) and/or Android Studio with an Android SDK
- A [Gemini API key](https://aistudio.google.com/app/apikey) for local prototyping
- (Production) a Firebase project with **Firebase AI Logic** and **App Check** enabled

### Installation

```bash
git clone https://github.com/masumtanjim7/Memora.git
cd Memora
flutter pub get
dart run build_runner build --delete-conflicting-outputs
```

### Run (prototype, direct API key)

```bash
flutter run --dart-define=GEMINI_API_KEY=your_key_here
```

> ⚠️ Never commit API keys. Direct-key mode is for local development only.

### Run (production, Firebase AI Logic)

```bash
dart pub global activate flutterfire_cli
flutterfire configure --project=<your-firebase-project>
flutter run --release
```

### First-time project creation

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

```mermaid
flowchart LR
    U["Unit tests<br/>models, helpers, controller"] --> W["Widget tests<br/>mic button, staging sheet, goldens"]
    W --> I["Integration tests<br/>record → stage → save → reminder"]
    I --> E["AI eval suite<br/>10 transcripts, ≥ 99% schema-valid"]
    E --> M["Manual device matrix<br/>Android 8–15, iOS 15+, offline, reboot"]
```

---

## 🛡️ Privacy

- No account required in the MVP
- Tasks and notes are stored locally on your device
- Only the current memo's transcript/audio is sent to Google Gemini; audio is deleted after successful processing
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
