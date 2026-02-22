# GenAI Assignment:

**Evaluation Criteria**

We will score your submission on:

* Clarity and practicality of architecture
* Robust JSON schema design
* Prompt quality (zero-shot, reliable, minimal hallucination risk)
* Handling of ambiguity + user review flow
* Bulk generation thinking (errors, naming, report)

---

## Problem 1: **Proposal for "Video-to-Notes"**

We have a local folder of long videos (3–4 hours each, 200MB+). Watching them fully is slow. We need an automated way to generate a "summary package" per video: **Summary.md** + highlight clips + screenshots, all organized per video. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

### **Task**

Prepare a **pre-processed solution proposal** comparing **three approaches**:

1. **Online/Cloud-Based (Already Available Solutions)**
2. **Build Our Own Using LLM APIs (Hybrid: local media processing + cloud LLM)**
3. **Build Fully Offline Using Open-Source Models (Local transcription + local LLM + pipeline)**

No code required. We want a **clear, practical proposal** with architecture and tradeoffs.

### Your Solution for problem 1:

**Objective**

Automate generation of a summary package per long video (3–4 hours):

- `Summary.md`
- Highlight clips
- Key screenshots
- All organized per video folder

---

#### Approach 1: Online / Cloud-Based (Existing SaaS)

**High-Level Architecture**

```
Video Upload → Cloud Transcription → AI Summary → Auto Highlights → Export
```

- User uploads or syncs video to SaaS platform.
- SaaS performs transcription, summarization, and highlight detection.
- User exports notes, clips, and screenshots.

**Example Tools**

- Descript, Notta, Otter, Fireflies, Riverside

**Pros**

- Fastest time-to-value; product is ready to use out of the box.
- Minimal engineering effort; mostly integration and workflow setup.
- Mature UX for playback, commenting, and search.

**Cons**

- Uploading 200MB+ files repeatedly increases bandwidth usage and wait time.
- Privacy and IP risk when sending internal or sensitive videos to third-party services.
- Limited customization of summary structure and formatting.
- High recurring cost at scale (per minute / per seat pricing).
- Poor control over bulk processing, queueing, and detailed error handling.

**Use Case Fit**

- Small teams or individual creators.
- Non-sensitive content with low customization requirements.

---

#### Approach 2: Hybrid (Local Media Processing + Cloud LLM APIs)

**Architecture**

```
Local Video Folder
       ↓
Local Audio Extraction (FFmpeg)
       ↓
Local Transcription (Whisper / Faster-Whisper)
       ↓
Chunking + Metadata (timestamps, speaker tags)
       ↓
Cloud LLM (OpenAI / Gemini)
       ↓
  ┌────────────────────────┐
  │ -  Summary.md           │
  │ -  Timestamped Highlights│
  │ -  Screenshot Timestamps │
  └────────────────────────┘
       ↓
Local Folder Output per Video
```

**Flow Details**

- Local service watches a folder and extracts audio from new videos using FFmpeg.
- Transcription runs locally (Whisper / Faster-Whisper) to avoid uploading raw media.
- Transcript is chunked by time or semantic boundaries, with timestamps preserved.
- Cloud LLM consumes chunks + metadata to:
  - Generate multi-level summaries (bullet + detailed).
  - Suggest highlight segments with start/end timestamps.
  - Identify key frames / scene changes for screenshot extraction.
- Local worker then:
  - Cuts highlight clips using FFmpeg.
  - Extracts screenshots at suggested timestamps.
  - Writes `Summary.md` and stores all assets under a per-video directory.

**Pros**

- Video and audio never leave the local system; only text transcript goes to cloud.
- Scalable and customizable pipeline adaptable to different summary templates.
- High-quality summaries leveraging frontier LLMs (OpenAI GPT-4 / Gemini).
- Lower cost than full SaaS at large video volumes.

**Cons**

- Requires engineering effort to build and maintain the pipeline.
- Depends on external APIs for LLM inference.
- Must handle rate limits, retries, and batching logic.

**Assessment**

- Recommended approach for most teams handling many long videos.
- Strong privacy + performance tradeoff.
- Easily extensible: add transcript search via vector stores, multi-language support, or per-role summary variants (e.g., "engineer-focused", "manager-focused").

---

#### Approach 3: Fully Offline (Open Source)

**Architecture**

```
Local Video
    → Local ASR (Whisper / Vosk)
    → Local Chunking + Topic Detection
    → Local LLM (LLaMA / Mistral, quantized)
    → Local Clip + Screenshot Generator (FFmpeg + heuristics)
    → Summary Package (Markdown + media assets)
```

**Flow Details**

- Everything runs on local hardware or on-prem servers.
- Open-source ASR models generate transcripts with timestamps.
- Local LLM summarizes text and suggests highlight segments.
- Heuristics or local embeddings help pick representative frames for screenshots.

**Pros**

- Zero cloud dependency; suitable for air-gapped environments.
- Maximum privacy and full control over data.
- Predictable costs once hardware is provisioned.

**Cons**

- Lower summary quality compared to latest proprietary models.
- Heavy hardware requirements (GPU/VRAM) for reasonable speed on 3–4 hour videos.
- Slow inference when processing multiple concurrent videos.
- Higher maintenance complexity (model updates, optimization, monitoring).

**Use Case Fit**

- Air-gapped environments.
- Strict compliance or regulated organizations.
- Teams with strong internal infrastructure and ML expertise.

---

#### Recommendation

The **Hybrid approach (Approach 2)** offers the optimal ROI and flexibility:

- Local control over heavy media processing and sensitive content.
- Cloud LLM intelligence applied where it provides the most value (reasoning, summarization, highlight logic).
- Scales cleanly to bulk processing by adding job queues and workers.

---

## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Posts**

Design a **single zero-shot prompt** that takes a user's persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user's voice and constraints. The output must be structured so the app can show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

---

**SYSTEM PROMPT**

```
You are a professional LinkedIn content assistant.
You must produce strictly valid JSON.
Do not include explanations, markdown formatting, code fences, or any text outside the JSON object.
```

---

**USER PROMPT TEMPLATE**

```
TASK:
Generate 3 LinkedIn post drafts based on the persona configuration and topic provided below.

INPUT:

Persona:
- Role: {{role}}
- Industry: {{industry}}
- Writing tone: {{tone}}
- Target audience: {{audience}}
- Constraints (words to avoid, emoji usage, CTA preference): {{constraints}}

Topic:
- Core topic to write about: {{topic}}

RULES:
1. Generate exactly 3 posts.
2. Each post must use a DISTINCT style from the following three — one per post:
   - insight-driven  (share a perspective, observation, or data point that reframes thinking)
   - story-based     (open with a personal or relatable narrative, build to a lesson)
   - actionable/tactical (give the reader a clear, practical takeaway or steps to act on)
3. Maintain the persona's tone and honor all constraints — respect word bans, emoji rules, and CTA preferences.
4. If any context is missing or vague, assume reasonable generic context to complete the post. Do NOT invent specific unverifiable facts such as company names, revenue numbers, or named clients.
5. Each post must be suitable for LinkedIn: strong opening hook, coherent body, and an optional closing CTA aligned with the persona's constraints.
6. Output must match the JSON schema below exactly — no extra keys, no missing keys.
7. Escape any internal quotation marks within string values so the JSON remains valid.

OUTPUT JSON SCHEMA (MANDATORY — output only this, nothing else):
{
  "posts": [
    {
      "style": "string (one of: insight-driven | story-based | actionable/tactical)",
      "content": "string (the full LinkedIn post text, ready to publish)",
      "assumptions": "string (brief note on any assumptions made; write 'none' if no assumptions)"
    },
    {
      "style": "string",
      "content": "string",
      "assumptions": "string"
    },
    {
      "style": "string",
      "content": "string",
      "assumptions": "string"
    }
  ]
}

Output only valid JSON. Do not wrap in markdown or add any explanation.
```

---

## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for "template field detection" and "field schema generation". We want a practical design, not code.

### Your Solution for problem 3:

---

#### Step 1: Template Ingestion

- User uploads a DOCX file via the UI.
- Backend parses and extracts:
  - Full text across all paragraphs and tables.
  - Basic structure metadata (section order, table positions).
- Extracted content is serialized into a structured plain-text or JSON representation for LLM consumption.

---

#### Step 2: Field Detection (GenAI)

- The structured document text is sent to an LLM (OpenAI / Gemini) with a detection prompt.
- The LLM is instructed to identify:
  - Editable variables (e.g., employee name, date of joining, salary, designation, address).
  - Field data type for each variable: `string`, `number`, `date`, `email`, `enum`, etc.
  - Whether each field is required or optional, inferred from surrounding document language (e.g., "must", "shall", "this certificate is awarded to").
- The LLM also deduplicates: if "Employee Name" appears in three clauses, it maps all occurrences to a single field key `employee_name`.

**Detection Prompt (LLM Instruction)**

```
You are a document analysis assistant.
Given the text of a business document, identify all fields that a user would need to fill in to generate a complete, personalized copy.
For each field, output:
- key: snake_case identifier
- label: human-readable name
- type: one of string | number | date | email | enum
- required: true or false
- occurrences: number of times this field appears in the document

Output strictly valid JSON matching this schema:
{
  "fields": [
    { "key": "string", "label": "string", "type": "string", "required": boolean, "occurrences": number }
  ]
}
```

---

#### Step 3: Schema Generation

- The LLM's detection output is used to build a locked JSON schema:

```json
{
  "template_name": "Offer Letter v1",
  "fields": [
    {
      "key": "employee_name",
      "label": "Employee Name",
      "type": "string",
      "required": true,
      "example": "Rahul Sharma"
    },
    {
      "key": "joining_date",
      "label": "Date of Joining",
      "type": "date",
      "required": true
    },
    {
      "key": "annual_ctc",
      "label": "Annual CTC (INR)",
      "type": "number",
      "required": true
    },
    {
      "key": "designation",
      "label": "Designation",
      "type": "string",
      "required": true
    }
  ]
}
```

- This schema is the single source of truth for both single and bulk generation downstream.

---

#### Step 4: User Review Loop

- The UI presents a field editor screen based on the generated schema:
  - Shows each detected field with its label, inferred type, required flag, and a sample occurrence from the document.
- User can:
  - Rename keys and labels.
  - Change field types (e.g., correct `string` to `date`).
  - Toggle required / optional.
  - Add missing fields or remove incorrect detections.
- Once satisfied, user clicks **Lock Schema**.
- Locked schema is stored against the template; all future generations use it without re-running LLM detection.

---

#### Step 5: Generation Modes

**Single Generation**

- UI dynamically renders a form from the locked JSON schema.
- User fills in all required fields and submits.
- Backend:
  - Loads the original DOCX.
  - Replaces all mapped field occurrences with provided values using a template engine (e.g., token replacement).
  - Generates output DOCX and optionally converts to PDF via a server-side library.
  - Returns the file as a direct download.

**Bulk Generation**

- User uploads an Excel file or links a Google Sheet.
- Each row = one output document; column headers must match schema field keys.
- Backend:
  - Validates all rows against the locked schema before generation begins.
  - For each valid row:
    - Generates DOCX and optional PDF.
    - Names each file using a configurable pattern, e.g., `OfferLetter-{{employee_name}}-{{joining_date}}.docx`.
  - Bundles all generated files into a ZIP for download or pushes to a configured cloud storage bucket.

---

#### Step 6: Error Handling and Reporting

- Per-row validation rules:
  - Missing required field → row skipped; error logged.
  - Type mismatch (e.g., alphabetic string in a `date` field) → row flagged; optionally attempt auto-coercion or mark as error per configuration.
  - Empty row → ignored silently.
- After bulk run completes, the system generates an output report (CSV or JSON):

```json
[
  { "row": 1, "status": "success", "output_file": "OfferLetter-Rahul-2026-03-01.docx" },
  { "row": 2, "status": "error", "reason": "Missing required field: joining_date" },
  { "row": 3, "status": "skipped", "reason": "Type mismatch: annual_ctc must be a number" }
]
```

- UI displays a run summary: total rows, success count, skipped count, errors with reasons.

---

## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

---

#### Core Components

**1. Character Manager**

- Stores per character:
  - Reference image (uploaded or AI-generated, used for visual consistency).
  - Personality traits, speech style, backstory.
  - Relationships to other characters (mentor, rival, sibling, etc.) stored as a graph.
- Exposes a character profile API consumed by the Script Generator and Asset Generation layer.
- Characters are defined once and reused across all episodes in the series.

**2. Story Input Layer**

- UI where the user:
  - Selects an existing character set or defines new characters.
  - Provides a short episode idea, situation, or outline (a few sentences to a paragraph).
  - Sets episode parameters: tone (funny, dramatic, educational), target audience, approximate duration (~5 minutes).
- Input is validated and normalized into a structured **Episode Brief** (topic, characters involved, tone, constraints).

**3. Script Generator**

- Consumes the Episode Brief + relevant character profiles.
- Produces:
  - Episode outline: list of scenes with locations, conflicts, and character arcs.
  - Full script: scene descriptions, dialogue lines tagged per character, and optional basic shot/camera notes.
- Applies a word-count or token-count heuristic to keep the script aligned with the ~5-minute target duration.

**4. Scene Engine**

- Breaks the full script into discrete, ordered scenes.
- For each scene, generates a **Scene Specification**:
  - Characters present and their emotional state / action in the scene.
  - Visual setting: location, time of day, mood, background style.
  - Key poses or actions required from each character.
- Scene Specifications are the interface between script generation and asset generation; both the visual and audio layers consume them independently.

**5. Asset Generation**

- **Visuals**
  - Character images and expressions:
    - Reuse stored reference images for consistency.
    - Generate additional pose or expression variants per scene using image generation (reference-image-conditioned).
  - Backgrounds:
    - Select from a template library (classroom, office, city street) or generate from the scene's setting description.
- **Audio**
  - Each character has a mapped voice model (TTS / voice synthesis).
  - Dialogue lines from the script are synthesized per character, per line or per scene chunk.
  - Optional: ambient sound effects added per scene setting.

**6. Assembly Pipeline**

- For each scene:
  - Composite the visual frame or short clip: character layer(s) positioned over background.
  - Synchronize synthesized voice audio with character visuals (lip-sync or simple timing alignment).
  - Layer in any ambient sound effects.
- Stitch all scenes into a single timeline in order:
  - Add scene transitions (cut, fade, etc.).
  - Prepend/append intro and outro sequences if configured for the series.
- Export final output:
  - Rendered MP4 episode (~5 minutes).
  - Episode metadata document (script, scene list, character usage, timestamps).

**7. Output Package**

- Script (structured JSON + human-readable Markdown).
- Scene plan: ordered list of scenes with character assignments, settings, and timestamps.
- Individual assets: character images/poses used, background images, per-line audio files.
- Optional final MP4 video ready for direct sharing or further editing.
