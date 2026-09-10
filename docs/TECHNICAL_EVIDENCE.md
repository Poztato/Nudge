# Technical evidence behind the Nudge proposal

Model and device sources checked on 8 September 2026. Metric sources and examples added on 10 September 2026. This document supports the findings in the README with source links, calculations, and integration details.

## Model files and runtime memory

The [official GGUF file listing](https://huggingface.co/openbmb/MiniCPM5-1B-GGUF/tree/main) shows MiniCPM5-1B-Q4_K_M.gguf at 688 MB and the F16 file at 2.17 GB. These are the site's rounded decimal file sizes.

- Weight-file reduction: `1 - 688 / 2170 = 0.682949...`, approximately 68%.
- Selected embedding artifact: ggml-org's [EmbeddingGemma-300M-qat-Q4_0.gguf](https://huggingface.co/ggml-org/embeddinggemma-300M-qat-q4_0-GGUF/tree/main) is listed at 278 MB. This is a GGUF conversion of Google's quantisation-aware-trained model.
- Combined model-file size: `688 + 278 = 966 MB`. This excludes additional tokenizer assets where needed, APK/runtime libraries, stored records, and temporary download space. Compared with the earlier 23 MB MiniLM artifact, the model-file budget increases by 255 MB.

The [OpenBMB demonstration](https://github.com/OpenBMB/MiniCPM-V-Apps#hardware-requirements) recommends at least 4 GB of device RAM for MiniCPM5-1B. Its table rounds the model download to approximately 0.5 GB; the README uses the specific artifact's 688 MB listing for storage estimates.

The cache calculation uses the model's [published configuration](https://huggingface.co/openbmb/MiniCPM5-1B/blob/main/config.json): 24 layers, two key/value heads, and an explicitly configured head dimension of 128. With FP16 keys and values, a standard full attention cache has this payload:

```text
KV bytes = 2 (K and V)
         * 24 layers
         * 2 KV heads
         * 128 values per head
         * context tokens
         * 2 bytes per FP16 value

2,048 tokens:  50,331,648 bytes = 48 MiB
4,096 tokens: 100,663,296 bytes = 96 MiB
```

These are calculated MiniCPM cache payloads, not observed process RAM or an EmbeddingGemma memory estimate. The assistant's 2,048/4,096-token context and the embedding model's 256-token input budget are independent. Loaded weight representation, runtime buffers, cache implementation, allocator overhead, the embedding session, and the UI all contribute to actual memory use. MB means 1,000,000 bytes; MiB means 1,048,576 bytes. No total peak-RAM or tokens-per-second result has been measured for Nudge.

Google advertises quantized EmbeddingGemma deployments using less than 200 MB RAM on its [product page](https://deepmind.google/models/gemma/embeddinggemma/). This is not a measurement of the selected 278 MB GGUF in Nudge, so it is not used as the app's RAM budget. Likewise, accelerator-specific published latency is not transferable to this phone lineup. Benchmark separate embedding and generation sessions, then compare simultaneous residency with unloading between stages.

## Interpreting the model benchmarks

The README transcribes three rows from [OpenBMB's original evaluation image](https://raw.githubusercontent.com/OpenBMB/MiniCPM/main/assets/minicpm5/public_leaderboard_en.png), linked from the [model card](https://huggingface.co/openbmb/MiniCPM5-1B). The image was visually inspected on the model-source review date. All compared columns are labelled Thinking, and the first selected row is labelled “Telecom-AA”.

The table describes publisher-reported model performance. It does not establish the quantisation, runtime, phone latency, or success rate of a Nudge deployment. MiniCPM's result varies by benchmark, so the README includes function calling and instruction following alongside its stronger Telecom-AA result.

The [BFCL project](https://gorilla.cs.berkeley.edu/leaderboard.html) evaluates function calling. The [tau benchmark project](https://github.com/sierra-research/tau2-bench) concerns agents interacting with tools and users in simulated domains. Neither measures student wellbeing or the usefulness of Nudge's recommendations.

Local validation must use the exact model file, runtime revision, chat template, tool adapter, and thinking mode. It should record incorrect tool selection, malformed arguments, unsupported claims, and rejected schedule proposals as well as successful examples.

## Local semantic memory

[EmbeddingGemma-300m](https://huggingface.co/google/embeddinggemma-300m) replaces the earlier MiniLM candidate. Google describes it as a roughly 300M-parameter multilingual embedding model, trained across more than 100 languages, with a 2,048-token maximum input. Its default output has 768 dimensions; Matryoshka Representation Learning permits 512, 256, or 128 dimensions with re-normalisation. We use the full output as the initial baseline, with smaller vectors as an evaluation option.

Nudge uses a **256-token embedding-input budget**, including prefixes and special tokens. Longer entries are split at passage boundaries with their source IDs and dates preserved. Total indexing work depends on both passage length and chunk count. New or changed passages are embedded once, and later searches reuse their vectors.

For float32 stored vectors:

```text
One vector = 768 dimensions * 4 bytes = 3,072 bytes
3,650 vectors = 3,650 * 3,072 = 11,212,800 bytes
11,212,800 / 1,048,576 = 10.693359375 MiB

Optional 256-dimensional representation:
One vector = 256 * 4 = 1,024 bytes
3,650 vectors = 3,737,600 bytes = 3.564453125 MiB

General payload = number of stored chunks * dimensions * 4 bytes
```

The 10.69 MiB example counts 3,650 chunks, not necessarily 3,650 days. It excludes source text, metadata, database overhead, and extra passages or overlap. Q4 model weights do not imply four-bit output vectors; the calculation uses float32 embeddings. Reducing output dimensionality decreases stored vector payload and similarity-search work, but does not proportionally reduce the encoder's forward-pass cost.

The [ggml-org model instructions](https://huggingface.co/ggml-org/embeddinggemma-300M-qat-q4_0-GGUF) demonstrate llama.cpp embedding execution. This replaces the separate ONNX Runtime Mobile route in the earlier proposal. Android integration must preserve the encoder's tokenisation, pooling/projection pipeline, and vector normalisation. Google's retrieval formats distinguish queries (`task: search result | query: ...`) from documents (`title: ... | text: ...`); apply them consistently when indexing and searching. The model card warns against float16 activations, so a generic FP16 conversion is not interchangeable with the selected quantized runtime path.

[sqlite-vec](https://alexgarcia.xyz/sqlite-vec/android-ios.html) publishes mobile libraries. Integration still requires compatible native SQLite packaging; a stock Android SQLite connection should not be assumed to load arbitrary extensions. Changing the embedding model requires rebuilding existing vectors, even if two encoders happen to have the same output dimensions.

Google's [published evaluation](https://huggingface.co/google/embeddinggemma-300m#benchmark-results) reports MTEB Mean Task scores at 768 dimensions: Q4_0 scores 60.62 against full precision's 61.15 on Multilingual v2, and 69.31 against 69.67 on English v2. The differences are 0.53 and 0.36 points. These are publisher results after quantisation, not a Nudge retrieval evaluation or evidence of the current overall leaderboard position. The README consequently motivates the model with its measured quality/size tradeoff rather than declaring it the universally best embedding model.

The proposed retrieval sequence is: apply relevant date constraints, rank note passages by semantic similarity, fetch original text and dates, and provide a bounded selection to the assistant. Exact counts and time calculations come from structured queries. Deleting or editing a source note must update its embeddings.

## Device test scope

The available lineup is Xiaomi 13T, Nothing Phone (1), Samsung Galaxy S23 Ultra, Sony Xperia 1 II, and Samsung Galaxy A52. The README summarises their test roles. These manufacturer specifications provide the detailed hardware references:

- [Xiaomi 13T](https://www.mi.com/mx/product/xiaomi-13t/specs/).
- [Nothing Phone (1)](https://ae.nothing.tech/en/pages/phone-1): Snapdragon 778G+, 8/12 GB RAM.
- [Samsung Galaxy S23 Ultra launch specifications](https://news.samsung.com/global/take-your-passions-further-with-the-new-samsung-galaxy-s23-series-designed-for-a-premium-experience-today-and-beyond): Snapdragon 8 Gen 2 for Galaxy, 8/12 GB configurations.
- [Sony Xperia 1 II specifications](https://www.sony.co.uk/electronics/support/mobile-phones-tablets-mobile-phones/xperia-1-ii/specifications): Snapdragon 865, 8 GB; Sony also announced a [12 GB variant](https://www.sony.com.hk/press/pdf/20201016_e.pdf).
- [Samsung Galaxy A52 launch specifications](https://news.samsung.com/in/the-galaxy-a52-a52-5g-and-a72-make-innovation-accessible-to-all) and [manufacturer catalogue](https://stg-images.samsung.com/is/content/samsung/p5/iran/smartphones/mobile-catalogue/samsung-mobile-digital-catalogue-20211122-mob.pdf): A52 has Snapdragon 720G and 4/6/8 GB variants. The [A52 5G datasheet](https://image-us.samsung.com/SamsungUS/samsungbusiness/resources/firstnet-datasheets/pdfs/Galaxy_A52_Series_Datasheet_%28FirstNet%29.pdf) lists Snapdragon 750G. The team's exact A52 variant remains to be recorded.

Exact RAM variants and installed OS versions of the available units remain to be recorded. The Xiaomi 13T is the user's primary test phone. Device availability is confirmed; Nudge compatibility and performance are not yet measured.

Measure cold load, time to first token, completed-response time, peak process memory, and repeated-session behaviour. Exercise reminder delivery with the app closed, after reboot, and under the device's power-management settings. Verify offline retrieval and inference after model installation.

## Metric definitions and sources

The README uses five separate indicators. Application code calculates them from saved records. The commitment, deadline, exercise-goal, and sleep-difference formulas are Nudge's planning calculations. WHO-5 uses the published instrument and scoring method.

### Commitment utilisation

```text
utilisation (%) = non-overlapping committed minutes / daily budget minutes * 100

Example:
2 hours of classes + 135 minutes of planned work = 255 minutes
6-hour budget = 360 minutes
255 / 360 * 100 = 70.8333...%, displayed as about 71%
```

The budget covers the same commitments counted in the numerator: classes, study, paid work, and other scheduled responsibilities. Merge overlapping intervals for the time total and retain the conflicting events for display. A missing budget displays a setup prompt; a zero budget displays "No goal set". Values above 100% remain visible.

### Deadline pressure

```text
pressure = estimated remaining work hours / available work hours before the deadline

Example:
8 remaining hours / 6 available hours = 1.33
Estimated time shortfall = 8 - 6 = 2 hours

Range example:
5-10 remaining hours / 6 available hours = 0.83-1.67
```

Available work time is calculated within the student's daily commitment budget and free calendar windows, up to the deadline. Fixed commitments consume capacity. Shared time is allocated across competing assignments; a task's own reserved work blocks count towards the time available to that task. This avoids counting one free block as fully available to several assignments.

Store the estimate's lower and upper bounds and any open-ended state. An open-ended bucket such as "over 10 hours" stays open-ended until refined. Missing deadlines or estimates display the required field. Outstanding work with zero available hours displays "No available work time". Completed and cancelled assignments leave the calculation.

### Exercise-goal completion

```text
goal completion (%) = reported exercise minutes in the week / weekly target minutes * 100

Example:
90 reported minutes / 150 target minutes * 100 = 60%
```

The target is chosen by the student. Exercise scheduled in the calendar remains planned until completion is reported. Each report has a date and a duration or duration range. Weekly totals add the reports for that week and show the number of recorded days. An explicitly reported zero is a recorded day; an unanswered day stays missing. A zero target displays "No goal set". This indicator describes progress towards the user's time goal.

### Sleep relative to target

```text
sleep difference (hours) = reported duration - personal target

Example:
6.5 reported hours - 8 target hours = -1.5 hours

Range example:
6-7 reported hours - 8 target hours = -2 to -1 hours
```

The [CDC's age-based guidance](https://www.cdc.gov/sleep/about/index.html) lists 8-10 hours for ages 13-17, at least 7 hours for ages 18-60, 7-9 hours for ages 61-64, and 7-8 hours for ages 65 and older. Setup presents the relevant guidance when an age range is supplied, then saves the student's chosen target. A skipped age question still allows a personal target.

Duration belongs to the selected night. Mood, energy, and feelings of restfulness stay separate from this duration calculation. The CDC also describes sleep quality in terms of uninterrupted and refreshing sleep.

### WHO-5 scoring and timing

The [WHO-5 publication](https://www.who.int/publications/m/item/WHO-UCN-MSD-MHE-2024.01) provides five statements about well-being over the past two weeks, each with six response options. Responses run from 0 to 5. The [WHO scoring description](https://www.who.int/data/gho/indicator-metadata-registry/imr-details/10282) specifies the sum and multiplication by four.

```text
raw score = sum of five item responses, each 0-5
display score = raw score * 4
raw range = 0-25
display range = 0-100

Example:
Responses: 3, 2, 4, 3, 3
Raw score: 15
Display score: 15 * 4 = 60
```

Nudge offers WHO-5 every two weeks using the original statements and response options. It stores all five responses, the instrument version, the reporting period, and the assessment date. It calculates the score after all five responses are supplied. The weekly review shows the latest completed score, its date, and the change from the previous completed assessment. Scores are shown as well-being observations alongside workload and reflection.

The Tuesday, Friday, and Sunday check-ins collect task progress, optional mood and energy ratings, dated exercise and sleep reports, and free text. These records support the weekly review separately from the two-week WHO-5 assessments. The three-day reminder schedule and every-two-week assessment schedule are Nudge's product choices.
