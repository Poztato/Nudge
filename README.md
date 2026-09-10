# Nudge by Macaroon

**Team:** Leonard Su, Dewmi Anusha, Lim Lee Khang, Tan Bo Yu  
**Problem Statement:** Stress & Workload Manager  
**Video Presentation:** Pending  
**Presentation Slides:** Pending

_Phase 1: concept submission. The prototype and presentation are in progress. This README describes the Android app we will build._

## 1. Project Overview

### The Problem

A student's calendar can look organised while their life feels messy. Typical task managers record classes, work blocks, and assignment deadlines. However, these records leave out poor sleep, unfinished work, postponed exercise, and difficult weeks, which greatly affects how much a student can manage.

Students need a way to connect their academic workload with how they are feeling. Knowing that three assignments are due next week is useful. Knowing how much work remains, when it can fit, and what helped during a similar week gives them a clearer next step.

Existing tools address parts of this problem. [ClickUp](https://help.clickup.com/hc/en-us/articles/15428419095831-ClickUp-AI-models-privacy-and-security-FAQ) provides AI assistance within a work-management platform, while [Reclaim](https://help.reclaim.ai/en/articles/13178785-reclaim-ai-disclosure) automates calendar scheduling. Our focus is a student planner that brings assignments, personal reflections, and well-being check-ins together, with AI processing and personal memory on the student's own phone. As a bonus, everything will be completely free of charge without the need of API keys or subscriptions. 

### Our Solution

Nudge is an Android personal planning app that connects academic workload with sleep, exercise, and well-being. It uses an on-device AI assistant to suggest priorities, explain workload, and recall useful experiences from the student's diary. The core app works offline after the models are downloaded, with a built-in calendar and an optional Google Calendar connection. Students can start with a task name, add details later, and choose when they want to check in.

### Core features

- **Local AI architecture.** A compact language model handles summaries and planning with local tools and storage. Students can use the core app without an AI subscription, API key, or uploading their diary to an inference service.
- **A planner with personal context.** The assistant uses preferences, assignment progress, and past reflections to suggest work blocks and breaks that fit the student.
- **A clear workload overview.** Application code calculates commitment utilisation and deadline pressure. The assistant explains the results alongside sleep, exercise, and well-being observations.
- **Quick assignment capture.** Save an assignment with just its name. Missing deadlines and effort estimates go into a pending-information list for later review.
- **Flexible check-ins and weekly reviews.** Short check-ins run on Tuesday, Friday, and Sunday by default. Sunday brings together progress, difficulties, and next week's priorities. Students can change the schedule or log an update at any time.
- **A private, searchable diary.** Ask questions such as "What helped the last time I felt overwhelmed by group work?" and retrieve relevant entries, including their original text and dates.
- **An integrated calendar and local reminders.** Assignments, errands, commitments, and work blocks live in one place. Accepted reminders use Android notifications, so they can arrive while the app is closed.

Consider a student approaching a group-project deadline. They have eight hours of work left and only six available work hours before submission. Nudge shows the time shortfall and retrieves an earlier reflection about how splitting a project into smaller sections helped them get started. It suggests the next section to work on, a suitable work block, and a break. The student reviews the plan and saves the reminder.

## 2. Ideation & Process

### 2.1 Ideas We Considered

We started with a task manager that understands the person using it. Persistent-memory agents such as Hermes and OpenClaw inspired us to connect conversation, stored preferences, and tools. We narrowed that idea to student planning and placed the assistant and its memory on the same device.

#### Chosen ideas

| Idea | Why we chose it |
| :--- | :--- |
| On-device AI with MiniCPM5-1B and llama.cpp | Keeps personal context on the phone and removes per-request model fees. |
| Structured records and vector memory | SQL supports exact queries about tasks and time. Vector search finds diary passages by meaning. |
| Built-in task management and calendar | Gives the assistant records it can use to suggest and save practical plans. |
| A compact, editable user profile | Carries useful preferences into future conversations and lets students correct what the assistant remembers. |
| Skippable task details with follow-ups | Lets students capture work immediately and complete missing information later. |
| Flexible check-ins and a Sunday review | Connects planning with progress and reflection while keeping the routine short. |
| WHO-5 every two weeks | Gives students a regular view of their well-being alongside shorter check-ins about everyday progress. |

#### Dropped ideas

| Idea | Why we dropped it |
| :--- | :--- |
| A full Hermes/OpenClaw-style agent | Although powerful, its broad action space exceeded our needs. Nudge uses a small set of tools for records, retrieval, and planning. |
| A larger model on a team workstation | Added a shared hardware bottleneck, server upkeep, and remote handling of personal information. |
| A website with hosted AI and Telegram/WhatsApp integration | Added accounts, messaging services, and API costs to a product intended for private everyday use. |
| Hosted Postgres/Supabase with vector search | We kept the relational-plus-vector design and moved storage onto the phone to remove the database-service dependency. |
| LFM2.5-1.2B (Thinking) | We selected MiniCPM5-1B for its smaller parameter count and stronger tool-use results in OpenBMB's comparison. |

#### Deferred integrations

| Idea | Place in the product |
| :--- | :--- |
| Google Calendar | An optional source of commitments after the built-in calendar works end to end. |
| Health-service integration | A later way to import sleep and activity. The first build uses self-reports. |
| Follow-up questions after classes | A later extension for capturing details such as a newly announced deadline. The first build uses the pending-information list and scheduled reviews. |

Our team discussions also refined the input flow. We changed daily reflection reminders to a configurable three-day schedule, with optional updates between check-ins. We replaced difficulty labels such as "Easy" and "Hard" with working-time ranges, so estimates connect directly to available hours. We also replaced warnings based only on unanswered reminders with a clear "Details needed" state, snoozing, and follow-ups tied to known deadlines.

The main architecture decision was to separate **reasoning from memory and calculation**. SQLite stores the history, retrieval selects relevant passages, and application code calculates time and scores. The model uses those results to explain the situation and suggest an action.

### 2.2 Ideation Boards

![Early sketch connecting assignments and physical activity](docs/images/early-workload-sketch.png)

This concept sketch shows our starting concern: academic progress can take attention away from exercise and recovery. It led us to bring workload and personal check-ins into the same planner.

![Early architecture separating raw data, daily metrics, and user notes](docs/images/early-memory-architecture.png)

The early design separated raw records, calculated metrics, and personal notes. We kept those roles and moved the hosted Hermes/Supabase design shown here onto Android.

### 2.3 Mentor Consultation

| Date | Mentor | Feedback Received | What Was Changed |
| :--- | :----- | :---------------- | :--------------- |
|      |        |                   |                  |

## 3. Design & Prototype

**UI Prototype:** Pending

The existing web scaffold supplies sample screens and interactions for the Android design. Its records and assistant responses are sample content. The interface has six connected views:

| Screen | Role in the experience |
| :--- | :--- |
| Today | Show priorities, available time, recent observations, and the records behind a suggestion. |
| Tasks | Add assignments, review missing details, and update progress or remaining effort. |
| Calendar | See fixed commitments, work blocks, and breaks; review changes before saving them. |
| Check-in | Record progress, mood or energy, exercise, sleep, and an optional reflection; open the weekly review and WHO-5. |
| Memory | Search dated reflections and inspect, edit, or remove saved preferences. |
| Preferences | Set time budgets, exercise and sleep targets, check-in days, and reminder times. |

### First-time setup

Setup asks for a few details that make the first plan useful:

1. **Daily commitment budget:** how many hours the student wants to commit each day, including classes, study, paid work, and other scheduled responsibilities. The app shows how much remains for independent work after fixed commitments.
2. **Weekly exercise target:** how many minutes the student wants to spend exercising.
3. **Sleep target:** the student's chosen nightly duration. An optional age range brings up age-based guidance to help them choose it. [Sleep guidance](https://www.cdc.gov/sleep/about/index.html)
4. **Check-in schedule:** preferred days, reminder time, and quiet hours. The default days are Tuesday, Friday, and Sunday.

Targets and reminders can be changed later. Students can skip a target and set it when they first use that part of the app.

### Adding and updating an assignment

The student enters an assignment name, then optionally adds a deadline and estimated remaining effort. Effort is measured in working hours, with choices of **under 2 hours**, **2 to under 5 hours**, **5 to 10 hours**, **over 10 hours**, **a custom estimate**, or **Unsure**.

If a range is selected, the app treats it as an uncertain amount of time until the student provides a more specific estimate. Workload calculations show that range. An open-ended estimate such as "over 10 hours" prompts the student to refine it or split the assignment into smaller parts.

Missing deadlines and estimates appear in a **Details needed** list. Nudge brings them into the weekly review and sends a weekly reminder that can be snoozed. A known approaching deadline also brings unresolved details into the Today view. Tasks with missing calculation inputs display "Needs estimate" or "Needs deadline".

As work progresses, the student can mark an assignment as started, update its remaining effort, or mark it completed or cancelled. This keeps the next plan based on the work still left to do.

### Check-ins and the Sunday review

Regular check-ins use short questions:

| Question | Input |
| :--- | :--- |
| Did you start, continue, or finish any assignments? | Select tasks and update progress or remaining effort. |
| How are your mood and energy? | Optional quick ratings. |
| How much did you exercise? | A duration range or exact minutes for a selected day. |
| How much did you sleep? | A duration range or exact hours for a selected night. |
| Anything you want to remember? | Optional free-text reflection. |

Students can log earlier dates or add updates between reminders. Each answer stays attached to its selected date. Summaries show how many days were recorded, and missing days stay visible as gaps. Duration ranges stay as ranges in the resulting metrics.

Sunday adds **"How was your weekend?"** and a weekly review: progress made, difficulties encountered, exercise against the weekly target, sleep observations, and upcoming deadlines. The assistant suggests one practical adjustment for the next week and brings up any missing assignment details.

Every two weeks, Nudge offers the **WHO-5 Well-Being Index**. Its five statements cover how the student has felt over the past two weeks, with responses scored from 0 to 5. The weekly review shows the latest score, its date, and the change from the previous completed assessment. [WHO-5](https://www.who.int/publications/m/item/WHO-UCN-MSD-MHE-2024.01)

_Screen images and the public prototype link are pending._

## 4. What Makes It Different

**Uses local AI on mobile.** As opposed to using cloud-based AI models for inference, our product exploits the recent advancement in smaller AI models that runs locally on mobile.

**Personal history helps shape the next plan.** A reflection about group work, a useful break, or a preferred study time becomes context for later suggestions. The student can open the original entry and see why the assistant brought it up.

**The app works with incomplete information.** A student can capture a task before knowing its deadline or effort. Nudge keeps the task visible, explains which details are missing, and returns to them during a review. This makes it easier to use when the student is already busy.

**Personalisation grows through memory.** Nudge uses retrieval-augmented generation (RAG) and an editable profile to bring relevant history into each conversation. The model's weights stay the same; its context grows as the student records more experiences.

**Planning and private reflection share one local system.** Inference, SQL storage, semantic retrieval, and reminders run on the phone. The student can use personal context for planning while keeping diary processing local.

**Daily use has no per-request AI fee.** Our aim is a free core app. Each phone runs its own assistant and stores its own records, so adding users does not require a matching increase in hosted inference or database capacity.

## 5. Technical Architecture & Feasibility

### Tech stack

| Layer | Technology | Why it fits Nudge |
| :--- | :--- | :--- |
| Android app | APK with native integration for storage, inference, and notifications; UI toolkit selection during implementation | Connects the interface to local device capabilities. |
| Local reasoning | MiniCPM5-1B Q4_K_M through llama.cpp | Runs a compact assistant with tool-calling support on the phone. |
| Agent execution | Custom orchestrator with a bounded tool registry | Gives the model typed operations for reading records, retrieving memories, and proposing plans. |
| Structured storage | SQLite with derived metric views | Stores source records and calculates exact dates, counts, and time totals. |
| Semantic memory | EmbeddingGemma-300m QAT Q4_0 GGUF through llama.cpp, with sqlite-vec | Supports multilingual diary retrieval using the same inference library as the assistant. |
| Scheduling | Android AlarmManager and local notifications | Delivers saved reminders independently of an active conversation. |
| Optional integration | Google Calendar | Imports existing commitments alongside the built-in calendar. |
| Distribution | Installable APK and downloadable models | Keeps the core app independent of a hosted backend or inference API. |

### How the assistant works

For a request such as "Help me make tomorrow less packed," Nudge follows this flow:

1. Read tomorrow's tasks, commitments, remaining effort, and time budget.
2. Calculate available capacity and retrieve relevant preferences and reflections.
3. Give those records to the local model to produce a plan and explanation.
4. Check the proposed blocks against fixed commitments, deadlines, and available time.
5. Show the suggestion and its supporting records to the student.
6. Save the accepted changes and schedule the reminders.

Application code handles calculations, input validation, and record updates. The model chooses from a small set of tools and explains the results. This gives Nudge a clear path from a conversation to an action the student can review.

### Personal memory and profile updates

The profile is inspired by the compact `SOUL.md`-style files used to carry persistent context into agent conversations. Nudge stores the authoritative preferences in SQLite and generates a short profile summary for the model. This keeps the profile consistent with the rest of the app's records.

When a student submits a reflection, Nudge:

1. Saves the original text and date.
2. Extracts useful observations and possible preferences.
3. Compares them with the saved profile.
4. Adds clear new preferences and applies explicit corrections.
5. Asks a short follow-up when a possible preference conflicts with the profile and the student's intent is unclear.
6. Refreshes the profile summary and links each saved preference to its source.

For example, "I studied late tonight" stays an observation about that night. "I now prefer studying after dinner" updates a study preference. Students can inspect, edit, or remove both their reflections and saved preferences.

### Records and metric calculations

SQLite holds the source data. The main record groups are:

| Records | Contents |
| :--- | :--- |
| Assignments | Name, status, start date, deadline, initial estimate, and current remaining effort, including estimate ranges. |
| Commitments and work blocks | Fixed events, planned study time, breaks, and links to assignments. |
| Progress updates | Dated changes to task status and remaining effort. |
| Check-ins and reflections | Original text, individual ratings, reported durations or ranges, and the dates they describe. |
| WHO-5 assessments | Five item responses, assessment date, two-week reporting period, and score. |
| Preferences and profile | Time budgets, targets, reminder settings, saved preferences, and links to their sources. |
| Pending information and reminders | The assignment and field needing an answer, reminder time, snooze state, and resolution status. |
| Derived metrics | Daily commitment utilisation, deadline pressure, sleep differences, and weekly exercise totals calculated from source records. |
| Embedding chunks | Search vectors linked to the original entry, passage, and date. |

Nudge displays five separate indicators:

| Indicator | Calculation | What it tells the student |
| :--- | :--- | :--- |
| Commitment utilisation | Non-overlapping committed minutes / daily commitment budget in minutes × 100 | How much of the day's chosen time budget is occupied. Values above 100% show that commitments exceed the budget. |
| Deadline pressure | Estimated remaining work hours / available work hours before the deadline | Whether the remaining work fits. A ratio above 1 shows a time shortfall. |
| Exercise-goal completion | Reported completed exercise minutes this week / weekly exercise target in minutes × 100 | Progress towards the student's weekly exercise goal. |
| Sleep relative to target | Reported sleep duration − personal sleep target | How far a recorded night's sleep is above or below the chosen target. |
| WHO-5 well-being | 4 × sum of five responses, each scored 0–5 | A score from 0 to 100, with higher scores indicating better reported well-being over the past two weeks. |

For example, two hours of classes plus 135 minutes of planned work occupy 255 minutes. Against a six-hour commitment budget, that is about **71% utilisation**. Overlapping blocks count once and appear as scheduling conflicts. Deadline pressure uses the student's available work time within their daily budget and allocates shared time across competing assignments.

The app keeps planned exercise separate from reported completion. Ranges produce range-based results, while missing inputs display the detail needed. Periods without reports display "No data". A zero target displays "No goal set"; remaining work with zero available time displays "No available work time". Completed and cancelled assignments leave the deadline-pressure calculation. An incomplete WHO-5 assessment stays incomplete until all five answers are supplied.

Sleep targets use age-based guidance during setup and remain editable. WHO-5 follows the original five-item scoring method. Sources and worked examples are in the [metric evidence](docs/TECHNICAL_EVIDENCE.md#metric-definitions-and-sources).

### Why MiniCPM5-1B

MiniCPM5-1B has approximately **1.08 billion parameters**, XML-style tool calls, and thinking and non-thinking modes. Its **688 MB Q4_K_M** file is about **68% smaller** than the **2.17 GB F16** file. We selected it to run a focused planning assistant within a phone-sized storage budget. [Model capabilities](https://huggingface.co/openbmb/MiniCPM5-1B), [model files](https://huggingface.co/openbmb/MiniCPM5-1B-GGUF/tree/main)

OpenBMB reports the following results for the thinking variants:

| Benchmark | MiniCPM5-1B | Qwen3.5-0.8B | LFM2.5-1.2B |
| :--- | ---: | ---: | ---: |
| τ²-Bench Telecom-AA | **79.53** | 47.70 | 19.60 |
| BFCLv4 | 25.15 | **25.53** | 10.60 |
| IFEval | 80.41 | 59.89 | **84.84** |

The Telecom-AA and BFCLv4 results support our choice for conversation and tool use, while IFEval covers instruction following. Nudge gives the model a focused tool set and uses non-thinking mode for routine summaries. [Publisher benchmark table](https://raw.githubusercontent.com/OpenBMB/MiniCPM/main/assets/minicpm5/public_leaderboard_en.png)

OpenBMB provides an Android demonstration with MiniCPM5-1B support, and llama.cpp provides an Android integration guide. We will build the tool-format adapter on this runtime and measure response quality, speed, and memory use on our test phones. [Android demo](https://github.com/OpenBMB/MiniCPM-V-Apps), [llama.cpp Android guide](https://github.com/ggml-org/llama.cpp/blob/master/docs/android.md)

### Local diary retrieval

**EmbeddingGemma-300m** supports more than 100 languages, which suits a student diary containing different languages. We chose its **QAT Q4_0 GGUF** version to share llama.cpp with the assistant. This replaces the earlier English-focused MiniLM and separate ONNX Runtime Mobile route. [Embedding model](https://huggingface.co/google/embeddinggemma-300m), [GGUF integration](https://huggingface.co/ggml-org/embeddinggemma-300M-qat-q4_0-GGUF)

Google's published MTEB results show that Q4_0 retains most of the full-precision model's retrieval performance: **60.62 versus 61.15** on Multilingual v2 and **69.31 versus 69.67** on English v2, differences of **0.53 and 0.36 points**. [Published evaluation](https://huggingface.co/google/embeddinggemma-300m#benchmark-results)

The model supports **2,048 input tokens**. Nudge uses **256-token inputs**, including the retrieval prefix, and splits longer reflections into dated passages. It embeds new or changed passages once and reuses their vectors. Search first applies relevant date filters, ranks passages by meaning, and supplies a few original passages to the assistant with their source IDs.

Our baseline uses **768-dimensional float32 vectors**. For **3,650 chunks**, the raw vectors occupy about **10.69 MiB**, with source text and database storage added separately. The model also supports smaller vectors; **256 dimensions** would reduce the same vector payload to **3.56 MiB**. Shorter input passages limit encoding work, while smaller output vectors reduce storage and search work. We will compare retrieval quality before switching from the full output.

The integration uses the model's query/document prefixes, tokenisation, pooling, vector normalisation, and supported activation precision. sqlite-vec runs within a compatible native SQLite package. Editing or deleting a reflection updates its vectors; changing the embedding model rebuilds the search index. [sqlite-vec mobile support](https://alexgarcia.xyz/sqlite-vec/android-ios.html), [retrieval details and calculations](docs/TECHNICAL_EVIDENCE.md#local-semantic-memory)

### Device footprint and testing

The two model files total approximately **966 MB**: **688 MB** for MiniCPM5-1B and **278 MB** for EmbeddingGemma. The embedding choice adds about **255 MB** over the earlier MiniLM file budget. Installation also needs room for the APK, supporting assets, downloads, and personal records.

For MiniCPM5-1B, the calculated FP16 attention-cache payload is **48 MiB at 2,048 tokens** or **96 MiB at 4,096 tokens**. Total runtime memory also includes model weights, runtime buffers, the embedding session, and the interface. Google reports a runtime footprint below 200 MB for its quantised EmbeddingGemma deployments. For our selected GGUF, the download is 278 MB; phone tests will measure loaded memory. [Model and memory evidence](docs/TECHNICAL_EVIDENCE.md#model-files-and-runtime-memory)

OpenBMB recommends **at least 4 GB of RAM** for its MiniCPM5-1B demonstration. Our test set covers a primary demo device, midrange phones, and older hardware:

| Available phone | Test role |
| :--- | :--- |
| Xiaomi 13T | Primary development and demo device; MediaTek platform. |
| Nothing Phone (1) | Midrange Qualcomm comparison. |
| Samsung Galaxy S23 Ultra | Flagship performance comparison. |
| Sony Xperia 1 II | Older flagship and Android-version coverage. |
| Samsung Galaxy A52 | Lower-performance comparison. |

We will record each unit's model, RAM, and Android version, then measure cold loading, time to first token, full-response time, peak RAM, indexing speed, and repeated use. Tests will compare keeping both models loaded with unloading one before running the other. Retrieval tests will check whether the assistant finds the right student entry, and tool tests will check valid actions and schedule conflicts. [Hardware specifications and test details](docs/TECHNICAL_EVIDENCE.md#device-test-scope)

### Local scheduling and data ownership

Android schedules saved reminders, including task nudges, pending-information follow-ups, and check-ins. Reminder records survive app closure and are restored after reboot. The implementation handles notification permissions, quiet hours, and device power-management settings. [Android scheduling documentation](https://developer.android.com/develop/background-work/services/alarms)

Personal records and embeddings stay in app-private storage. Diary retrieval and inference run locally. Export and deletion cover source records, saved profile information, and related vectors. Backup behaviour will be an explicit storage setting. Google Calendar is a separate opt-in connection for calendar data.

### Build plan & scope

The first build centres on one complete student journey on the **Xiaomi 13T**: capture an assignment, complete its details, inspect workload, receive a memory-informed suggestion, save a reminder, and review progress.

1. **Run the local models.** Integrate MiniCPM5-1B and EmbeddingGemma with llama.cpp, including structured tool calls and diary retrieval.
2. **Build the records and calculations.** Add setup, tasks, progress updates, manual commitments, pending details, and the five indicators in SQLite.
3. **Connect memory to planning.** Add the editable profile, source-linked retrieval, schedule checks, and acceptance of suggested changes.
4. **Complete the routine.** Add local reminders, flexible check-ins, the Sunday review, and WHO-5 every two weeks.
5. **Demonstrate and test the full flow.** Verify saved data after reopening, offline use after model installation, source-backed answers, and reminder delivery with the app closed and after reboot.

The remaining phones extend performance and compatibility testing after the primary flow works. Google Calendar, health imports, and after-class questions follow the core build. The prototype and video will show the same student journey, so the submission explains both the problem and how Nudge helps the student take the next step.
