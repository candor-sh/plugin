---
name: candor
description: Run AI-moderated user studies, usability tests, and human evaluations with real participants via the Candor CLI (create, approve, publish, monitor, and read findings/results). Use when the user wants user feedback, user testing, participant recruiting, preference comparisons, or human ratings/labels.
---

# Candor — AI-Moderated User Studies & Human Evaluation

Candor lets you create and manage studies directly from the terminal. By default, you get a shareable link to send to colleagues. Add `--recruit` to have Candor find external participants for you.

A study is a single primitive with orthogonal dimensions: **goal**, **stimulus** (items/url/topic), **task** (compare/rank/categorize/label/rate/score/scorecard/describe/transcribe/respond/review/use/share), **moderator** (none/session/study), **moderator output** (voice/text), **audience**, and **participants**.

## Setup

Before running any `candor` command, check it is available with `candor --version`.

- **Not installed**: install it with `npm install -g @candor.sh/cli` (requires Node 20+). If global installs aren't possible, use `npx -y @candor.sh/cli` in place of `candor`.
- **Not authenticated** (commands fail with an auth error, or `candor doctor` reports it): in an interactive local terminal, tell the user to run `candor init`. In non-interactive or remote environments (e.g. cloud agent sessions) where browser login isn't possible, ask the user to create an API key at https://candor.sh/dashboard/api-keys and provide it as the `CANDOR_API_KEY` environment variable (secret).

## Available Commands

Run these via your shell tool. Always use `--json` for machine-readable output. Run any command with `--help` for full usage details (e.g. `candor study create --help`).

- `candor study create --goal "<goal>" [--items|--url|--topic] [--input-modes <modes>] [--duration <minutes>] [--batch-size <n>] [options] --json` — Create a study
- `candor study list [--archived] --json` — List all studies (or `candor study --json`). Archived studies hidden by default; use `--archived` to include them.
- `candor study status <id> [--show-preview] --json` — Check progress
- `candor study status <id> --live [--show-preview]` — Real-time terminal monitoring (do NOT combine with `--json`; use a long timeout, e.g. `timeout: 300000`)
- `candor study results <id> [--show-preview] --json` — View results and rankings (item-based studies)
- `candor study findings <id> [--show-preview] --json` — Get prioritized feedback (P0–P3, moderated studies)
- `candor study approve <id> --json` — Approve and create draft on recruitment platform (human-in-the-loop gate)
- `candor study publish <id> --json` — Run health checks and begin recruiting (publishes the draft)
- `candor study coverage <id> --json` — Adaptive coverage report (study-scoped studies)
- `candor study demographics <id> --json` — View participant demographics (recruited studies)
- `candor study pause <id> --json` — Pause recruitment (keeps data)
- `candor study resume <id> --json` — Resume a paused study
- `candor study cancel <id> --json` — Cancel and stop recruiting (keeps collected responses)
- `candor study archive <id> --json` — Archive a study (hides from list, keeps all data)
- `candor study unarchive <id> --json` — Restore an archived study

## Study Types by Stimulus

### Item-based studies (`--items`)

For comparing, rating, categorizing, or getting open-ended responses on items (audio files, images, text, etc.). Returns a shareable link — anyone with the link can respond.

```
candor study create --goal "which logo is better" --items "a.png,b.png" --json
candor study create --goal "which sounds better" --items "*.mp3" --json
candor study create --goal "flag violations" --items "posts.jsonl" --task categorize --labels "safe,borderline,violation" --json
candor study create --goal "rate quality" --items "a.png,b.png" --task rate --json
candor study create --goal "describe what you see" --items "*.png" --task describe --json
candor study create --goal "transcribe these clips" --items "*.mp3" --task transcribe --json
candor study create --goal "evaluate model outputs" --items "outputs.csv" --task scorecard --criteria '[{"name":"Correctness","weight":10,"levels":["Excellent","Good","Satisfactory","Poor","Failing"]},{"name":"Clarity","weight":6,"levels":["Excellent","Good","Poor"]}]' --json
candor study create --goal "compare landing page headlines" --items "Headline A,Headline B" --moderator session --json
candor study create --goal "rate these outputs, let evaluators explain" --items "*.txt" --task rate --moderator session --moderator-output text --json
```

For item studies, adding `--moderator session` enables a **per-item follow-up button** under each task. After the participant answers, they can click "Follow up" to start a short voice (or text) conversation with the AI moderator scoped to that one item. The moderator opens with a task-aware probe ("You picked A over B — what tipped it?"), and the conversation completes after 2 substantive turns + 5s of speech (or 80 typed chars). Voice is the default; text is the fallback if mic is denied. Use `--moderator-output text` to force text-only chat.

### URL-based studies (`--url`)

For getting feedback on a web page. By default, the AI conducts a full voice interview while the participant browses (session + voice output). Add `--recruit` to bring in external participants from a recruitment platform; otherwise you get a shareable link. Add `--tab` when the site blocks iframe embedding or requires login — the URL opens in a new tab and participants share their screen instead. Pass `--moderator-output text` if you specifically want text-only monitoring without an AI voice.

```
candor study create --goal "feedback on our new landing page" --url https://example.com --json
candor study create --goal "feedback on our new landing page" --url https://example.com --input-modes text --json
candor study create --goal "test the onboarding flow" --url https://example.com --recruit --participants 10 --json
candor study create --goal "test the onboarding flow" --url https://example.com --recruit --interview INTERVIEW.md --json
candor study create --goal "test the login flow" --url https://app.example.com --recruit --participants 5 --json
candor study create --goal "feedback on landing page" --url https://example.com --iframe --json
candor study create --goal "feedback on landing page" --url https://example.com --moderator session --moderator-output text --json
```

### Topic-based studies (`--topic`)

For interviews or discussions about a topic, without a specific product URL.

```
candor study create --goal "coaching experience" --topic "health coaching" --json
candor study create --goal "coaching experience" --topic "health coaching" --input-modes text --json
candor study create --goal "coaching experience" --topic "health coaching" --recruit --audience "coaching app users" --json
candor study create --goal "coaching experience" --topic "health coaching" --moderator session --moderator-output text --json
```

## Inference Rules

The CLI infers `--task` and `--moderator` automatically:

| You provide | Inferred task | Inferred moderator | Inferred output |
|-------------|---------------|-------------------|-----------------|
| `--items` only | `compare` | `none` | — |
| `--items` + `--labels` | `categorize` | `none` | — |
| `--items` + `--moderator session` | (as inferred) | `session` | `text` (per-item text-chat follow-ups, no mic prompt) |
| `--url` | `use` | `session` | `voice` |
| `--topic` | `share` | `session` | `voice` |
| `--goal` only | `share` | `session` | `voice` |

**Session duration (`--duration`):** Sets the target session length in minutes for moderated studies (URL/topic). Default is 5 minutes. Affects the study script, reward calculation, and estimated completion time. Example: `--duration 10` for a 10-minute interview.

**Scorecard criteria (`--criteria`):** Defines the rubric dimensions for `--task scorecard`. Accepts inline JSON or a path to a JSON file. Each criterion has `name`, `weight` (integer, higher = more important), and `levels` (ordered best-to-worst). Example: `--criteria '[{"name":"Correctness","weight":10,"levels":["Excellent","Good","Poor"]}]'`. If omitted, defaults to a single "Overall Quality" dimension. The `--criteria` flag also auto-sets `--task scorecard`.

**Batch size (`--batch-size`):** Number of tasks per assignment (default 20, default 10 for scorecard). Use when the user says something like "make sure each user sees 10 pairs" or "show 5 items per page". Example: `--batch-size 10`. Smaller batches = more assignments = higher cost (due to minimum reward floor). Only set when explicitly requested.

**Reward multiplier (`--reward-multiplier`):** Multiplies the auto-calculated reward to incentivize faster pickup. Example: `--reward-multiplier 1.5` pays 50% more, `--reward-multiplier 2` pays double. Use when the user says "pay more", "faster pickup", "higher reward", "boost reward", or "urgent". Only set when explicitly requested.

**Input modes (`--input-modes`):** Controls how participants provide input. Values: `voice` (default — mic on, voice transcribed), `text` (text chat only, no mic), or `voice,text` (both). Use `--input-modes text` when participants may not have a microphone or when text responses are preferred.

**Display mode (tab vs iframe):** URL studies default to **tab mode** — the website opens in a new browser tab and participants share their screen. This works with any site including those behind login, with strict CSP, or that block iframes. To embed the URL in an iframe instead, pass `--iframe`. Only use `--iframe` for simple public pages (marketing sites, landing pages) where you're confident the site allows iframe embedding.

**When to add `--recruit`:** When the user wants external participants. Trigger words: "users", "participants", "testers", "people", "recruit", "find participants", "real users". For example, "get 3 users to rate these" means `--recruit --participants 3`. Only omit `--recruit` when the user explicitly says they want to share links themselves (e.g., "send to my team", "share with colleagues"). By default when a participant count is mentioned, assume `--recruit`.

**When to add `--moderator session`:** When the user wants an AI-moderated interview. Trigger words: "interview", "ask them", "opinion", "what they think", "talk to them", "conversation". For URL studies, `--moderator session` means the AI conducts a live interview while the participant browses. Combine with `--duration` if a time is mentioned (e.g., "3 minute interview" → `--duration 3`).

**Moderator scopes:**
- `none` — No moderator, no mic. Structured tasks only (compare/rate/label).
- `session` — AI moderator conducts a live interview within a single session. Adapts follow-ups in real time.
- `study` — AI moderator operates across sessions. Learns from prior sessions to adaptively focus future ones on gaps and emerging themes. Use `candor study coverage <id>` to see what's been explored. Requires `--recruit`.

**When to add `--moderator session` for items studies:** When the user wants per-item follow-up probes — short, opt-in conversations after each task where the AI asks why the participant chose what they did. Trigger words: "follow up", "ask why", "probe reasoning", "explain their choice", "drill down", "let them elaborate", "ask them to expand", "understand their thinking". For item studies (`--items`), this enables a "Follow up" button under each task. The participant clicks it to start a short text (or voice) conversation scoped to that one item — the moderator opens with a task-aware probe ("You picked A over B — what tipped it?") and the conversation completes after 1 substantive turn (with at least 10 chars typed or 1s of speech). Example: "have the AI ask follow-ups after each comparison" → `--moderator session`. Default mode for items follow-ups is `text` (chat-only, no mic prompt). Add `--moderator-output voice` to switch to voice follow-ups with text fallback.

**Moderator output modes (`--moderator-output`):**
- `voice` — AI speaks aloud via TTS. **Default for URL/topic moderated studies** — a full voice interview. For items studies, opt in via `--moderator-output voice` and mic is requested only when the participant clicks "Follow up"; the panel falls back to text chat if mic permission is denied.
- `text` — Text-only. **Default for items studies with `--moderator session`** — chat-only follow-ups with no mic prompt, lower friction for batch raters. For URL/topic studies, this means the AI monitors via mic transcription and intervenes with text prompts only (no AI voice).

## Workflow

### Creating a Study

1. **Determine the stimulus type** based on what the user wants:
   - Items to compare/rate/label → use `--items`
   - Product to test → use `--url`
   - Topic to discuss → use `--topic`
   - Just a goal → omit stimulus (defaults to topic-based)

2. **For item-based studies**, identify the items. The `--items` flag is comma-separated and the CLI auto-detects:
   - **Text labels**: `--items "option_a,option_b,option_c"` — plain strings, no upload
   - **File paths**: `--items "sample_a.mp3,sample_b.mp3"` — uploaded automatically
   - **Glob patterns**: `--items "*.mp3"` or `--items "outputs/*.png"` — expanded and uploaded
   - **CSV file**: `--items "items.csv"` — rows parsed into items (see CSV format below)
   - **Mixed**: `--items "baseline.mp3,challenger.mp3,control_text"`

   **Supported media types** (auto-detected by extension): audio (mp3, wav, ogg, flac, m4a, aac, webm, wma), image (png, jpg, jpeg, gif, webp, svg, bmp, tiff), video (mp4, mov, avi, mkv, wmv).

   **Size limits**: 100 MB per file, 1 GB total, max 50 files per study.

3. **For URL-based studies**, infer the URL from context if not provided:
   - `package.json` `homepage` field
   - README for a deployed URL
   - Ask the user if nothing is found

4. **Create the study** — Run `candor study create` with the appropriate flags.

5. **Show the plan and preview link** — Display stimulus type, task, participant count, estimated cost and time. Always include the preview link (`https://candor.sh/study/preview/<study-id>`) so the user can see what participants will experience. Then ask the user if they want to approve using AskUserQuestion (or equivalent interactive prompt).

6. **Wait for explicit approval** — Do NOT run `candor study approve` without the user saying "yes", "approve", "go ahead", or similar.

7. **Approve** — Run `candor study approve <id> --json`.
   - **If it fails with a 402 error**, parse the JSON response and share the URL with the user:
     - `"needsPaymentMethod": true` → Share the `billingUrl` from the response: "You need to add a payment method first. Open **{billingUrl}** to add a credit card, then let me know and I'll retry the approval."
     - `"Insufficient balance"` → Share the `topUpUrl` from the response: "Your balance is too low (have $X, need $Y). Open **{topUpUrl}** to add funds, then let me know and I'll retry."
     - Always use the URL from the JSON response — it deep-links to the right action. **Wait for the user to confirm** before re-running `candor study approve`.
   - **Default (link) studies**: Returns shareable URL(s) immediately. Share them with your team via Slack, email, etc.
   - **Recruited studies** (`--recruit`): Creates a draft on the recruitment platform. After approving, **ask the user if they want to publish now**. If yes, run `candor study publish <id>` — it will automatically wait if the draft is still being created. If no (e.g., the user wants to inspect the draft first), stop and let them publish manually later.

8. **Publish** (recruited studies only) — Run `candor study publish <id>` to go live. Only run this after the user confirms they want to publish.

9. **Monitor** — Run `candor study status <id> --live` for real-time progress. Use a long timeout (at least 5 minutes). The output includes a **live dashboard link** (e.g. `https://candor.sh/live/<token>`) — share this link with the user so they can watch sessions in real time from their browser. Continue monitoring with `--live` in the terminal.

10. **Results** — Use `candor study results <id>` for item-based rankings/labels/ratings, or `candor study findings <id>` for moderated study findings.

11. **Pause/Cancel/Archive** — Use `candor study pause <id>` to temporarily stop recruitment (keeps data, can resume later). Use `candor study cancel <id>` to permanently stop (credits refunded proportionally). Use `candor study archive <id>` to hide a completed, cancelled, or paused study from the list — all data preserved, reversible with `candor study unarchive <id>`.

### Choosing the right task

**Item-based tasks** (used with `--items`):
- `compare` / `rank` — Workers compare pairs and pick a winner. Produces a stack ranking with win rates. Use when the user says "rank", "compare", "which is better", "stack rank", "prefer".
- `categorize` / `label` — Workers assign a label from a set of categories. Use when the user says "label", "categorize", "classify", "tag". Requires `--labels`.
- `rate` / `score` — Workers rate items on a numeric scale. Use when the user says "rate", "score", "quality score".
- `scorecard` — Workers evaluate items across multiple rubric dimensions. Each dimension has a name, weight, and ordered quality levels. Use when the user says "scorecard", "rubric", "multi-dimensional evaluation", "grade across criteria". Requires `--criteria` (JSON array or path to JSON file). Each criterion has `name` (string), `weight` (integer, higher = more important), and `levels` (ordered best-to-worst, e.g. `["Excellent","Good","Poor"]`). Results include per-dimension breakdowns and an overall weighted score normalized to [0,1].
- `describe` — Workers write a free-text description per item. Use when the user says "describe", "what do you see", "annotate".
- `transcribe` — Workers write out what they hear/see. Use when the user says "transcribe", "write out", "caption".
- `respond` — Workers provide open-ended text feedback per item. Generic catch-all for free-text tasks. Use when the user says "respond", "feedback", "thoughts on each".
- `review` — Workers write a review per item. Use when the user says "review", "evaluate", "critique".

**Interview tasks** (used with `--url` or `--topic`):
- `use` — Moderated usability test on a URL. Use when the user says "test", "try", "use".
- `share` — Moderated interview about a topic. Use when the user says "interview", "discuss", "talk about".

**CSV format:**

When `--items` points to a `.csv` file, the CLI parses it row by row:
- **Single column** — each value is auto-detected (URL, file path, or text label)
- **Two+ columns** — first column is the label, second is the URL or file path
- **Header row** — auto-detected if first row contains "label", "url", "path", or "file"
- **URLs** (`https://...`) are passed through as asset URLs directly — no upload needed
- **Local file paths** in the CSV are uploaded just like inline file paths

**Anchored pairwise CSV (`anchor,a,b` columns):**

When the CSV header is exactly `anchor,a,b`, each row defines one pairwise comparison against a reference. The participant sees the anchor above the two options and picks which one better matches. Use this when the user wants to compare two candidates against a fixed reference — e.g. "which audio sounds more like the speaker in this recording", "which voice matches the person in this image", "which clip sounds most like the original". The anchor can be any media type (audio, image, video), independent of what A/B are. Each row becomes one task; files referenced in multiple rows upload only once. The CLI automatically sets `--task compare`.

```csv
anchor,a,b
alice_ref.wav,alice_candA.wav,alice_candB.wav
bob_ref.jpg,bob_voice1.wav,bob_voice2.wav
video_clip.mp4,sync_attempt1.wav,sync_attempt2.wav
```

Usage: `candor study create --goal "which voice matches the reference" --items "pairs.csv" --json`

Trigger phrases: *"which sounds like the speaker in"*, *"which matches this reference"*, *"which looks most like the person in"*, *"compare against this recording"*, *"pick the voice that best matches"*.

**How to build the --items flag:**

When the user mentions files, use Bash tools to find them first:
- User says "evaluate the audio files in outputs/" → run `ls outputs/*.mp3` to confirm, then `--items "outputs/*.mp3"`
- User says "compare these two images" and you see `a.png`, `b.png` → `--items "a.png,b.png"`
- User has a spreadsheet of URLs → save as CSV, then `--items "urls.csv"`
- User says "rank A, B, C" (no files) → `--items "A,B,C"` (text labels)

**Example natural language → study mapping:**

| User says | Command |
|-----------|---------|
| "rank these audio samples by TTS quality" | `--goal "rank by TTS quality" --items "*.mp3"` |
| "label these images as cat or dog" | `--goal "classify animals" --items "*.png" --task categorize --labels "cat,dog"` |
| "rate these UI mockups on a 1-5 scale" | `--goal "rate UI quality" --items "mockup_a.png,mockup_b.png" --task rate` |
| "which of these headlines is more engaging" | `--goal "which headline is more engaging" --items "Headline A,Headline B"` |
| "get feedback from my team on this page" | `--goal "landing page feedback" --url https://example.com` |
| "test the onboarding flow with 10 users" | `--goal "test onboarding" --url https://example.com --recruit --participants 10` |
| "test the login flow on our app" | `--goal "test login flow" --url https://app.example.com --recruit --participants 5` |
| "get feedback on our landing page, embed it" | `--goal "landing page feedback" --url https://example.com --iframe` |
| "describe what's in these images" | `--goal "describe contents" --items "*.png" --task describe` |
| "transcribe these audio clips" | `--goal "transcribe audio" --items "*.mp3" --task transcribe` |
| "get feedback on each design" | `--goal "design feedback" --items "*.png" --task review --recruit --participants 5` |
| "grade these essays on correctness and clarity" | `--goal "grade essays" --items "essays.csv" --task scorecard --criteria '[{"name":"Correctness","weight":10,"levels":["Excellent","Good","Poor"]},{"name":"Clarity","weight":6,"levels":["Excellent","Good","Poor"]}]'` |
| "compare these headlines and ask reviewers why they picked" | `--goal "which headline lands harder" --items "Headline A,Headline B" --moderator session` |
| "which of these voices sounds most like the reference speaker" | `--goal "voice match to reference" --items "pairs.csv"` (CSV with `anchor,a,b` header) |
| "for each image, which audio sounds like the person shown" | `--goal "audio-face match" --items "pairs.csv"` (image anchors, audio candidates) |
| "rate these outputs, let raters explain their reasoning" | `--goal "rate quality" --items "*.txt" --task rate --moderator session` |
| "label these images as cat or dog and probe edge cases" | `--goal "classify animals" --items "*.png" --task categorize --labels "cat,dog" --moderator session` |
| "quick vote on which feature to build next" | `--goal "feature prioritization" --items "Feature A,Feature B,Feature C"` |
| "get 1 person's opinion on this page, 3 min" | `--goal "what do users think this does" --url https://example.com --moderator session --duration 3 --recruit --participants 1` |
| "interview users about coaching" | `--goal "coaching experience" --topic "health coaching" --recruit` |

### Pricing

Rewards are **auto-calculated** based on media type (text/image/audio/video), task type, and batch size — targeting $12/hr to ensure fast pickup. You do not need to set `--reward`. Use `--reward-multiplier` to pay more for faster pickup (e.g. `--reward-multiplier 1.5`).

**Cost formula:**
```
assignments = ceil(totalTasks / batchSize)
Platform cost = assignments × participants × autoReward × 1.4 (platform fee)
Total cost = Platform cost × 1.4 (Candor fee)
```

All task types default to **20 tasks per assignment**. Override with `--batch-size <n>` if the user requests a specific number of tasks per participant.

Do NOT try to estimate cost yourself — always rely on the `Est. cost` returned by `candor study create --json`. The CLI computes it correctly accounting for media type, batching, and fees.

### Checking Results

- Use `candor study status <id> --json` to show progress
- Use `candor study results <id> --json` to show rankings and agreement metrics (item-based)
- Use `candor study findings <id> --json` to show prioritized findings (moderated)
- Offer to create GitHub issues for P0/P1 items if the user wants

### Preview Mode

Preview sessions (from the preview link shown after `candor study create`) are hidden by default in `status`, `findings`, and `results`. Add `--show-preview` to include preview data in the output. This is useful for testing a study before recruiting participants.

## Billing & Balance

**Link studies (default) are free** — no balance needed. Just share the link.

**Recruited studies** (`--recruit`) require a funded balance.

- **First time**: `candor init` opens Stripe to add a card. New users get **$5 free credit**.
- **Balance check**: Balance is shown during `candor init` and at `candor study create` (as `Est. cost`).
- **Charge timing**: Balance is debited when you run `candor study approve` — not at draft creation.
- **Insufficient funds**: If balance is too low, the CLI returns a JSON error with a `topUpUrl` that deep-links to Stripe Checkout. Share this URL with the user — they pay, then you re-run the approve command.
- **Auto-reload**: Users can configure auto-reload at https://candor.sh/billing — set a threshold and target, and the saved card is charged automatically.
- **Cancellation refund**: `candor study cancel` refunds the proportional cost of incomplete tasks.
- **Billing dashboard**: https://candor.sh/billing shows balance, transaction history, and auto-reload settings.

When a 402 error occurs, parse the JSON response and share the `billingUrl` or `topUpUrl` with the user. Do not retry — the user needs to complete payment first and tell you to retry.

## Brainstorming & Interview Guide

When a user wants help designing a study before creating it, use this brainstorming workflow:

### 1. Discover context

Ask the user about:
- What product/feature/topic they're researching
- Who their target audience is
- What hypotheses or assumptions they want to test
- What decisions the research will inform
- Any specific questions they already have in mind

### 2. Draft an interview guide

Create `.candor/studies/draft/INTERVIEW.md` with this structure:

```markdown
# Interview Guide: [Topic]

## Research Goals
- Primary: ...
- Secondary: ...

## Target Audience
- ...

## Sections

### 1. Warm-up (3-5 min)
- [Ice-breaker questions to build rapport]

### 2. [Topic Area] (5-10 min)
- [Open-ended questions]
- [Follow-up probes]
- [Tasks to observe, if applicable]

### 3. [Topic Area] (5-10 min)
- ...

### N. Wrap-up (3-5 min)
- Overall impressions
- Anything else they want to share

## Moderator Notes
- [Guidance on tone, follow-ups, things to watch for]
- [Topics to probe deeper on if mentioned]
```

**Question design rules (IMPORTANT — follow these when writing questions):**
- NEVER write yes/no or leading questions. "Is this confusing?" → "Walk me through what you understand so far." "Would you recommend this?" → "What would you tell a friend about this?"
- Every section must have at least one friction question that forces critical thinking: "What's the first thing you'd change?", "What's missing?", "What confused you even for a moment?"
- Never write confirmation-seeking questions ("So you'd say it's X?"). Use "What makes you say that?" instead.
- Avoid questions that prime for positive responses. "What do you like?" → "What stands out to you?" "Is there anything you'd change?" → "What's the first thing you'd change?"
- Include at least one comparison or tradeoff question per guide: "How does this compare to [alternative]?", "If you could only keep one feature, which would it be?"

### 3. Iterate

Show the draft to the user. Incorporate their feedback — add questions, remove sections, adjust tone, reorder. Repeat until they're satisfied.

### 4. Create the study

Once the guide is finalized, pass it to the study creation command:

```
candor study create --goal "<goal>" --url <url> --interview .candor/studies/draft/INTERVIEW.md --json
```

or for topic-based:

```
candor study create --goal "<goal>" --topic "<topic>" --interview .candor/studies/draft/INTERVIEW.md --json
```

The server uses the interview guide to generate a tailored study script with Haiku. The guide shapes the sections, questions, and tasks — so a well-crafted guide produces a much better study than the default template.

**Key:** The brainstorming happens locally using your model. The server only runs Haiku to convert the final guide into a structured script format.

## Important Rules

- NEVER approve a study without explicit human confirmation
- For link studies: after approving, share the returned URL(s) with the user so they can distribute them
- For recruited studies: show the `Est. cost` before approving, then remind the user to run `candor study publish <id>` to go live
- If Candor is not installed or not authenticated, follow the **Setup** section above
- Default to `--recruit` when users mention a participant count or say "users"/"testers"/"participants". Only use link mode when they explicitly want to share links themselves

## Reporting Issues & Feedback

Use `candor report` to send bug reports, feature requests, or general feedback to the Candor team. The command auto-attaches debug context (CLI version, OS, Node version, auth status).

```
candor report "description of issue or suggestion" --json
candor report "study create failed with 500 error" --category bug --study-id std_abc123 --json
candor report "would be nice to have CSV export" --category feature --json
candor report "publish timed out after 60s" --category bug --context "error: API error 504 Gateway Timeout" --json
```

### When to use

- **After a command fails unexpectedly**: If any `candor` command returns an error that seems like a platform bug (500 errors, unexpected responses, timeouts), offer to send a report. Include the error output in `--context`.
- **When the user says "tell candor" or "report this"**: Any explicit request to communicate with the Candor team.
- **When something seems broken**: If you notice inconsistent state, missing data, or unexpected behavior while using Candor commands.

### How to use

1. Summarize the issue clearly in the message (what happened, what was expected)
2. Set `--category bug` for bugs, `--category feature` for feature requests, or omit for general feedback
3. If a study is involved, include `--study-id <id>`
4. If there's error output, pass it via `--context "the error text here"`
5. Always use `--json` for machine-readable confirmation
