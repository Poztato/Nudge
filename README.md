# Nudge by Macaroon

**Team:** Leonard Su, Dewmi Anusha, Lim Lee Khang, Tan Bo Yu  
**Problem Statement:** Stress & Workload Manager  
**Video Presentation:** Pending  
**Presentation Slides:** Pending

_Phase 1: concept and UI prototype. The architecture below describes our planned Android application._

## 1. Project Overview

### The Problem

A student's calendar can look organised while their life feels messy. Typical task management systems organise schedules into time blocks and takes note of assignment deadlines. However, these records leave out accumulated loads like poor sleep, burnouts, exercise that keeps getting postponed, or a difficult week that changes how much work feels manageable.

This is the problem we want to address. Students need a way to connect their academic backlog to how they are actually doing mentally and physically, and use that understanding to make better plans before the next week becomes another recursive round of catching up.

Existing planners solve parts of this separately. [ClickUp](https://help.clickup.com/hc/en-us/articles/15428419095831-ClickUp-AI-models-privacy-and-security-FAQ) organises work and provides AI assistance for brainstorming, while [Reclaim](https://help.reclaim.ai/en/articles/13178785-reclaim-ai-disclosure) automates calendar scheduling. However, these platforms are best used for work-related scheduling and brainstorming tasks rather than physical and mental well-being. They are not designed for individuals to log down how they are feeling outside of work, and their scheduling capabilities do not consider personal stress levels and physical health. Additionally, most of these platforms lock their AI-tools behind paywalls and require an internet connection to process data using third-party AI services. In turn, this also creates privacy concerns for users who wish to vent about their personal lives.

### Our Solution

Nudge is a fully offline AI-powered Android personal planning app that gives you a strategy at the start of the day and checks in on you in the evening. Just like traditional planners, it has a built-in task management system with optional connections to Google Calendar or a fully localised calendar. The app is designed to monitor assignment deadlines and calendar events, while keeping track of the user's well-being and private reflections in a SQL + vector database that only the local model and the user can access.

### Core features of the product:

- **Fully localised app.** Unlike the vast majority of AI-powered apps, Nudge uses a capable ~1B param model for summarising and reasoning. This keeps everything running locally, and students can build their personal history and use it to make better daily decisions without an AI subscription, an API key, or uploading their diary online.
- **A planner with personal context.** The assistant combines work-related tasks with preferences and regular check-ins to suggest priorities and recovery time. This AI agent works in a self-improving way, where more context gives it more information to cater to the user's preferences.
- **A workload overview.** The agent derives and interprets workload metrics from recorded data like assignments, exercise, leisure time, deadlines, and everyday responsibilities, then visualises them by day. This makes it easy to notice accumulation across a week instead of treating each deadline as an isolated event.
- **A private, searchable diary.** Reflections become part of a local semantic memory, allowing questions such as "What helped the last time I felt overwhelmed by group work?" to retrieve relevant entries even when their wording is different.
- **A short daily reflection.** Quick choices capture mood, energy, and activity, while optional text preserves the explanation behind them. This supplies information a timetable or wearable cannot reliably infer.
- **Scheduled nudges as notifications.** Daily plans and reminder times are handed to Android's scheduling system. Students can close the app and still receive a reminder to start a task or reflect on their day.
- **An integrated task manager and calendar.** Assignments, errands, deadlines, and work blocks live in the app. Google Calendar is an optional source of existing commitments.

Consider a student approaching another group-project deadline. Their diary includes an earlier entry about how a short walk and splitting the work into smaller sections helped them get started. In the proposed flow, Nudge can retrieve that entry, inspect today's commitments, suggest a suitable work block and break, and schedule the accepted reminder locally. The reflection has become useful planning context, with its original date and wording available to inspect.

## 2. Ideation & Process

### 2.1 Ideas We Considered

We started with the idea of a task manager that understands the person using it. That led to a more specific question: could the student carry the assistant, its tools, and its memory on the same device?

#### **Chosen Architecture**

| Idea                                                 | Why it was chosen                                                                                                                                                                                             |
| :--------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| On-device AI with MiniCPM5-1B and llama.cpp (chosen) | Recent compact models support the instruction following and tool use needed for a personal assistant. Local inference removes the recurring model-service dependency and keeps personal context on the phone. |
| Hybrid structured and vector memory (chosen)         | Tasks and daily metrics need exact queries; diary entries need retrieval by meaning. Combining both gives the assistant factual history and personal context without a separate database server.              |
| Built-in task management and calendar (chosen)       | The assistant can work with records the app controls and turn suggestions into actual planned actions. Google Calendar remains an optional connection.                                                        |
| Scheduled reminders and daily reflection (chosen)    | Planning becomes a continuing routine: make a plan, receive a nudge, record what happened, and use that context tomorrow.                                                                                     |

#### **Dropped Ideas**

| Idea                                                                | Why it was dropped                                                                                                                                                                                              |
| :------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A full Hermes/OpenClaw-style agent                                  | The persistent memory and tool use architecture inspired the design of Nudge, but their broader capabilities exceeded the product's needs. We chose a focused application agent with a defined set of tools.  |
| A larger model hosted on a team workstation                         | Avoided per-call API charges but introduced a shared hardware bottleneck, server availability concerns, and remote handling of personal information.                                                            |
| A website with a hosted model API and Telegram/WhatsApp integration | Offered a familiar interface, but added accounts, messaging integrations, token expenditure, and abuse controls to a product intended for everyday personal use.                                                |
| A hosted Postgres/Supabase database with vector search              | This was the early memory design. We retained the relational-plus-vector approach and moved it onto the device to remove the need for a cloud account and database service.                                     |
| Health-service integration                                          | Useful for importing activity or sleep, but optional. Self-reports keep the core experience available without a wearable or health permissions.                                                                 |
| Contextual follow-up questions after classes                        | Open for further exploration, such as asking whether a lecturer announced a deadline. The first build centres on scheduled reminders and daily reflection while we refine when extra questions would be useful. |
| Using LFM2.5-1.2B (Thinking)                                        | Initial planning proposed this model by Liquid AI, but current benchmarks show that MiniCPM5-1B is a stronger model with a lower parameter count.                                                   |

The architectural turning point was separating **reasoning capacity from memory capacity**. The model does not need to carry a semester of diary entries in its active context. A database preserves that history, retrieval selects what matters, and application code calculates the numerical facts. This allows a compact model to work with a much richer personal history than its active prompt contains.

### 2.2 Ideation Boards

![Early sketch connecting assignments and physical activity](docs/images/early-workload-sketch.png)

This sketch captures the original motivation: progress in one area can hide neglect in another. The curves illustrate the relationship we wanted to explore, rather than measured health signals.

![Early architecture separating raw data, daily metrics, and user notes](docs/images/early-memory-architecture.png)

The early design already separated raw records, derived daily metrics, and contextual memory. The current architecture brings those layers and the assistant onto Android, replacing the hosted Hermes/Supabase arrangement shown here.

### 2.3 Mentor Consultation

| Date | Mentor | Feedback Received | What Was Changed |
| :--- | :----- | :---------------- | :--------------- |
|      |        |                   |                  |

## 3. Design & Prototype

**UI Prototype:** Pending

The interface is organised around six connected views. The existing web scaffold provides the visual reference and sample interactions for the Android design; its data and assistant responses are illustrative.

| Screen         | Role in the experience                                                                                                            |
| :------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| Today          | Bring priorities, available time, and recent observations together, with access to the reasoning and records behind a suggestion. |
| Tasks          | Capture assignments and errands with deadlines, effort estimates, and progress updates.                                           |
| Calendar       | See fixed commitments alongside proposed work and recovery blocks; review adjustments before accepting them.                      |
| Daily Check-in | Record the day's experience through quick choices and an optional diary entry.                                                    |
| Memory         | Search reflections, inspect dated source entries, and manage what the assistant remembers.                                        |
| Preferences    | Set study preferences, a comfortable workload budget, and reminder times.                                                         |

The design follows a daily rhythm: an actionable overview in the morning, lightweight updates during the day, and a short reflection in the evening. Optional questions stay skippable, and missing observations remain visibly unknown. The aim is to make the app useful on difficult days, when maintaining another elaborate productivity system is least appealing.

_Screen images and the public prototype link are pending._

## 4. What Makes It Different

**The complete intelligence loop resides on the device.** Nudge combines inference, structured storage, semantic retrieval, and scheduling in one personal environment. Keeping all four local means the assistant can use sensitive context without handing that context to an inference service. A student's ability to revisit their diary or plan tomorrow is designed to remain available offline after installation.

**Personal memory becomes operational.** A reflection can influence a later priority or recovery suggestion. This goes beyond saving notes next to a calendar: the assistant can retrieve an earlier experience, connect it to current commitments, and propose an action through the app's tools. The student can inspect the source and decide whether the connection is useful.

**Personalisation accumulates without retraining.** As the student records more days, the app gains more relevant history to retrieve. This is retrieval-augmented generation (RAG) applied to a private personal archive. The model's weights stay the same but its answers gain context from records that belong to the student. Editing or deleting a memory updates the archive and its search representation.

**The economics support daily use.** Local inference eliminates per-request model fees, and local storage removes the need to host each student's personal database. Our intention is a free core experience that students can use regularly without rationing AI credits. Growth adds computation on users' devices rather than requiring a proportionally larger inference server.

**Recent compact models make this a practical product direction.** Current models provide decent tool use at roughly one billion parameters with quantised runtimes, and mobile vector search bring capabilities associated with hosted agent systems into a much smaller deployment. Nudge applies that combination to provide a sustained, private support for student workload and recovery.

## 5. Technical Architecture & Feasibility

### Tech stack

| Layer                | Proposed technology                                                                                       | Why it fits Nudge                                                                                                                 |
| :------------------- | :-------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| Android application  | Android APK with native integration for storage, inference, and notifications; UI toolkit to be finalised | Gives the assistant direct access to device capabilities and a persistent daily-use interface.                                      |
| Local reasoning      | MiniCPM5-1B Q4_K_M through llama.cpp                                                                      | Compact inference with tool-calling support, keeping the model and personal context together on the device.                         |
| Agent execution      | Custom orchestrator with a bounded tool registry                                                          | Converts model requests into validated database reads and planning proposals. Each tool has a specific purpose and typed inputs.    |
| Structured memory    | SQLite and a derived `daily_metrics` view                                                                 | Stores authoritative records and supports exact date, count, and time queries without a server.                                     |
| Semantic memory      | EmbeddingGemma-300m, proposed QAT Q4_0 GGUF through llama.cpp, with sqlite-vec | Multilingual diary retrieval on the device, sharing the inference library with the assistant while keeping a separate embedding model. |
| Scheduling           | Android AlarmManager and local notifications                                                              | Delivers saved reminders independently of an active chat session, with permission and power-management handling.                    |
| Optional data source | Google Calendar                                                                                           | Imports existing commitments while the built-in calendar keeps the core independent of a Google account.                            |
| Distribution         | Installable APK and model download                                                                        | No hosted application backend or cloud inference is required for the core experience.                                               |

### Why MiniCPM5-1B

Our starting model has approximately **1.08 billion parameters**, native XML-style tool calls, and selectable thinking and non-thinking modes. Its published Q4_K_M file is **688 MB**, compared with **2.17 GB** for F16, a roughly **68% reduction in weight-file storage**. [Model capabilities](https://huggingface.co/openbmb/MiniCPM5-1B), [published model files](https://huggingface.co/openbmb/MiniCPM5-1B-GGUF/tree/main)

Tool use matters because Nudge must translate a request into operations on real records. Selected results from OpenBMB's published comparison are:

| Benchmark           | MiniCPM5-1B | Qwen3.5-0.8B | LFM2.5-1.2B |
| :------------------ | ----------: | -----------: | ----------: |
| τ²-Bench Telecom-AA |   **79.53** |        47.70 |       19.60 |
| BFCLv4              |       25.15 |    **25.53** |       10.60 |
| IFEval              |       80.41 |        59.89 |   **84.84** |

All three are the **thinking variants** in the publisher's table. These publisher-reported scores inform model selection; validation of the Q4_K_M Android build is part of our build plan. [Original benchmark table](https://raw.githubusercontent.com/OpenBMB/MiniCPM/main/assets/minicpm5/public_leaderboard_en.png)

The strong Telecom-AA result is a useful selection signal for an assistant that must coordinate conversation and tools. BFCLv4 also shows why we keep the action space focused. Nudge gives the model a small set of relevant operations, validates their arguments, and handles schedule constraints in code. Routine summaries can use non-thinking mode; its speed and quality will be measured separately.

For a request such as “Help me make tomorrow less packed,” the proposed execution path is:

1. Read tomorrow's tasks, commitments, and workload budget.
2. Retrieve relevant preferences and recent reflections.
3. Generate a priority or schedule proposal grounded in those records.
4. Check that proposed blocks fit, preserve fixed commitments and deadlines, and show the change for approval.
5. Save the accepted plan and schedule its reminders.

OpenBMB provides an Android demonstration supporting MiniCPM5-1B, while llama.cpp documents native Android integration. We will implement and test the model's tool-format adapter in that local runtime. [OpenBMB Android demo](https://github.com/OpenBMB/MiniCPM-V-Apps), [llama.cpp Android guide](https://github.com/ggml-org/llama.cpp/blob/master/docs/android.md)

### Memory that can grow beyond the prompt

Nudge uses **hybrid relational and semantic memory**, with different responsibilities for each layer.

SQLite answers questions such as “How many assignments are due this week?” from exact records. Derived daily metrics support a longitudinal view without asking a language model to invent scores or perform the bookkeeping. Notes preserve the student's explanation of those days, and vector search retrieves relevant passages by meaning. Together, these give the assistant both a factual timeline and the context behind it.

This also makes the workload view explainable. Two hours of classes plus 135 minutes of planned work occupy 255 minutes, or about 71% of a student-selected six-hour commitment budget. Overlapping calendar blocks are counted once, and a missed check-in stays unknown. The assistant can trace that number back to its inputs and discuss what to change, while the underlying calculation stays reproducible.

Our proposed embedding model is **Google's EmbeddingGemma-300m**, designed for multilingual retrieval across more than 100 languages. That makes it a better fit for a student diary that may contain several languages than the earlier English-focused MiniLM candidate. We propose the **QAT Q4_0 GGUF** conversion published by ggml-org, allowing both models to use llama.cpp within the Android app. sqlite-vec keeps vector search inside SQLite. [Embedding model](https://huggingface.co/google/embeddinggemma-300m), [GGUF runtime instructions](https://huggingface.co/ggml-org/embeddinggemma-300M-qat-q4_0-GGUF), [sqlite-vec mobile support](https://alexgarcia.xyz/sqlite-vec/android-ios.html)

Quantisation-aware training makes this a useful mobile candidate: Google's published Q4_0 results score **60.62 versus 61.15** for full precision on MTEB Multilingual v2, and **69.31 versus 69.67** on MTEB English v2 (Mean Task, 768 dimensions). The reported losses are just **0.53 and 0.36 points** respectively. These results support the compact deployment choice; retrieval quality on student reflections will be tested in Nudge. [Published evaluation](https://huggingface.co/google/embeddinggemma-300m#benchmark-results)

We will cap each embedding input at **256 tokens**, reserving space within that budget for the retrieval prefix and special tokens. Longer diary entries will be split into passages with their original dates and source IDs. Short inputs bound the work per passage; saving only new or edited passage embeddings avoids reprocessing the whole archive. The model supports up to **2,048 input tokens**, but a larger limit is unnecessary for this initial passage-based design. Actual indexing latency depends on the phone, runtime, and number of chunks.

The baseline uses the full **768-dimensional output**. At four bytes per float32 value, one vector occupies **3,072 bytes**. A collection of **3,650 stored chunks** therefore needs about **10.69 MiB for raw vectors**. This represents 3,650 diary entries only when each fits into one chunk; original text, metadata, database overhead, and extra passages are additional. Matryoshka Representation Learning also permits smaller outputs: **256-dimensional vectors** would reduce that example to **3.56 MiB**, with retrieval quality to compare before adopting them. Output dimensionality controls vector storage and search cost; the 256-token input cap controls passage length. [Calculation details](docs/TECHNICAL_EVIDENCE.md)

Only relevant passages enter the assistant's prompt. The rest remain in the database, ready for later retrieval. We plan to retain dates and source IDs throughout, so an answer can point back to what the student actually wrote.

### Device footprint and available test phones

The proposed EmbeddingGemma QAT Q4_0 GGUF file is approximately **278 MB**. Together with MiniCPM5-1B Q4_K_M at **688 MB**, the two model files total approximately **966 MB**, before the application, any additional tokenizer assets, and personal data. [Embedding file listing](https://huggingface.co/ggml-org/embeddinggemma-300M-qat-q4_0-GGUF/tree/main)

Runtime RAM is a separate budget. For **MiniCPM5-1B**, a conventional FP16 attention cache is approximately **48 MiB at 2,048 tokens** or **96 MiB at 4,096 tokens**, calculated from its published configuration. This assistant-context calculation is unchanged by EmbeddingGemma's 256-token passage limit. We will retrieve a few relevant memories per assistant request and measure embedding and generation separately, including whether unloading one model before running the other is useful on the older phones. [Model configuration](https://huggingface.co/openbmb/MiniCPM5-1B/blob/main/config.json), [memory calculation](docs/TECHNICAL_EVIDENCE.md)

OpenBMB recommends **at least 4 GB of device RAM** for its MiniCPM5-1B demonstration. That provides a starting reference; application peak RAM also includes loaded weights, compute buffers, the embedding runtime, and the UI. [Publisher hardware guidance](https://github.com/OpenBMB/MiniCPM-V-Apps#hardware-requirements)

We have access to five phones for development and testing:

| Available device                                                                                                                           | Published hardware                                                                               | Planned role                                                           |
| :----------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------- |
| [Xiaomi 13T](https://www.mi.com/mx/product/xiaomi-13t/specs/)                                                                              | Dimensity 8200-Ultra; 8 GB or 12 GB RAM variants                                                 | Primary development and demo device.                                   |
| [Nothing Phone (1)](https://ae.nothing.tech/en/pages/phone-1) | Snapdragon 778G+; 8 GB or 12 GB RAM variants | Evaluate responsiveness on a midrange Qualcomm platform. |
| [Samsung Galaxy S23 Ultra](https://news.samsung.com/global/take-your-passions-further-with-the-new-samsung-galaxy-s23-series-designed-for-a-premium-experience-today-and-beyond) | Snapdragon 8 Gen 2 for Galaxy; 8 GB or 12 GB RAM variants | Provide a flagship comparison for inference and indexing. |
| [Sony Xperia 1 II](https://www.sony.co.uk/electronics/support/mobile-phones-tablets-mobile-phones/xperia-1-ii/specifications) | Snapdragon 865; 8 GB in the linked specification, with a [12 GB variant also released](https://www.sony.com.hk/press/pdf/20201016_e.pdf) | Check an older flagship and its installed Android version. |
| [Samsung Galaxy A52](https://news.samsung.com/in/the-galaxy-a52-a52-5g-and-a72-make-innovation-accessible-to-all) | A52: Snapdragon 720G, 4/6/8 GB RAM variants; A52 5G has different hardware, so the unit's exact variant will be recorded | Test the lower-performance end of the available lineup. |

These devices provide a concrete test path across MediaTek and Qualcomm platforms, older flagships, and midrange phones. The RAM figures above describe published variants, not the confirmed memory of our individual units. We will record each unit's model number, RAM, and installed Android version alongside measured peak memory, embedding latency, and assistant response times. Compatibility testing is pending.

### Local scheduling and data ownership

The model is invoked for reasoning and retrieval; scheduled reminders use Android's own alarm and notification mechanisms. Accepted reminder times are persisted so the app can restore them after reboot. Quiet hours, notification permissions, and Android power-management behaviour are part of the implementation. This keeps routine nudges independent of continuous inference. [Android scheduling documentation](https://developer.android.com/develop/background-work/services/alarms)

Personal records, embeddings, and inference context stay in app-private storage and local execution. Export and deletion apply to both source notes and their derived vectors. Optional Google Calendar access is a separate, explicit connection. The privacy benefit comes from removing routine diary transfers to a server; device access and backup settings still need deliberate handling.

### Build plan & scope

The building phase will deliver the complete local loop: capture, remember, reason, schedule, and reflect.

1. **Establish on-device inference.** Run the selected model on the test phones and verify structured tool requests, response quality, and memory use.
2. **Build durable personal records.** Implement tasks, calendar commitments, preferences, diary entries, and daily check-ins in SQLite.
3. **Connect both memory layers.** Derive daily metrics and add local embeddings, semantic retrieval, and source inspection.
4. **Make the assistant actionable.** Integrate priorities and schedule proposals with constraint checks and user acceptance.
5. **Complete the daily routine.** Add local reminders, recovery suggestions, and a check-in that informs the following day.
6. **Validate the APK across devices.** Measure model loading, response time, peak RAM, retrieval relevance, and reminder delivery, including offline operation after installation.
