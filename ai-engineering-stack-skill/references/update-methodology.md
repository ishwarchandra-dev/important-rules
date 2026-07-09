# Prompt Engineering Skill — Update Methodology

This documents the repeatable approach for keeping this skill current with the fast-moving
prompt engineering landscape.

## Trigger

The skill was referenced (via `skill_view` or loaded by the agent) and found to reference
outdated model versions, missing key developments, or lacking coverage of new domains.
Known signal: model version in the description doesn't match latest frontier.

## Research Phase

### Step 1 — Cast a Wide Net (5–10 parallel searches)

Use `web_search` across these query categories in parallel. This is the most important step —
good queries uncover the gaps.

| Category | Example Queries |
|----------|----------------|
| **Latest models** | `prompt engineering [year] frontier models latest` |
| **New techniques** | `new prompting techniques [year]` |
| **Model-specific** | `[Model Name] [version] prompt engineering best practices` |
| **Emerging tools** | `prompt evaluation frameworks [year]`, `MCP model context protocol prompt engineering [year]` |
| **New modalities** | `video generation prompting [year]`, `multi-modal prompting [year]` |
| **Agentic systems** | `agentic prompting orchestration patterns [year]`, `multi-agent prompt design [year]` |
| **Security** | `OWASP prompt injection [year]`, `prompt security agent MCP [year]` |

### Step 2 — Dig Into Promising Results (2–5 web extracts)

For the most relevant-seeming results, use `web_extract` to get full content. Look for:
- **Retrospectives** — "2026 retrospective", "H1 2026 review"
- **Comparison articles** — "prompt engineering frameworks compared"
- **Comprehensive guides** — "complete guide to [topic]"
- **Official vendor announcements** — anthropic.com, openai.com, deepmind.google, x.ai, etc.
- **Reputable AI publications** — Latent Space, Simon Willison's blog, Hugging Face blog, etc.

### Step 3 — Parallel Subagent Research (Pro Tip)

For comprehensive coverage, spawn 4-5 parallel research subagents (one per topic area):

1. **Frontier LLM models** — latest from Anthropic, OpenAI, Google, xAI, DeepSeek, GLM, Qwen, Meta
2. **Agentic + MCP + context engineering** — frameworks, patterns, governance
3. **Image/video gen** — Midjourney, gpt-image, Imagen, FLUX, Veo, Kling, etc.
4. **PromptOps + evaluation** — frameworks, benchmarks, automated optimization
5. **Security** — OWASP, NIST, EU AI Act, jailbreak research, red-teaming tools

Each subagent should: cross-validate dates with multiple sources, prioritize official vendor
pages, explicitly note what has NOT changed since the last update, and flag any claims that
rest on a single secondary source.

### Step 4 — Gap Analysis

Read the current skill files (SKILL.md + key references) and compare against findings.
Look for:

| Gap Type | Examples |
|----------|----------|
| **Model version drift** | Skill says "Claude Opus 4.7" but 4.8 is out |
| **Paradigm shifts** | "Be honest" sycophancy fix → literal/contract-style → harness engineering |
| **New categories** | Video gen, PromptOps, MCP, Agent Skills — completely absent |
| **Thin coverage** | Agentic runbooks mentioned but no depth |
| **Tool names** | Promptfoo, DeepEval — not present |
| **Discontinued products** | Sora discontinued April 2026 — needs removal |
| **Regulatory deadlines** | EU AI Act full enforcement Aug 2, 2026 — needs mention |
| **New attack classes** | Tool Poisoning, Lethal Trifecta — needs coverage |

### Step 5 — Patch the Skill

Apply updates via `patch` (preferred) or `edit` (for major restructures):

1. **Description** — update model names in frontmatter
2. **Technique table** — add new rows for new techniques
3. **Model defaults** — update per-model guidance
4. **Templates** — adjust examples for new model behavior
5. **New steps** — add entirely new sections for emerged categories
6. **References** — update model-guide.md with new models/sections
7. **Migration guides** — keep old model sections in a "migration" pattern so users upgrading existing workflows can find the changes

### Step 6 — Verify

`skill_view` the result. Check:
- TOC matches actual sections
- No stale model references remain
- New sections are reachable from the TOC
- Frontmatter description mentions current models
- File line endings are LF (not CRLF — Edit tool fails on CRLF)
- All links/references are reachable

## Update History

### End-of-June 2026 Update #2 — Horse / Loop Engineering (this revision)

**Trigger:** User asked for in-depth research on "horse engineering" and "loop engineering"
and to add them to the skill. User also emphasized: "always make sure not to hit the rate
limiting."

**Research method (rate-limit-safe):** Ran 11 sequential web searches with 8–12 second
delays between each call. Zero rate-limit hits. All searches succeeded. Saved all research
JSONs to `/home/z/my-project/research/horse_loop/`.

**Key findings:**

1. **"Horse engineering" = "harness engineering"** (etymology confirmed):
   - A harness is **horse tack** — the reins, saddle, and bit that channel a powerful but
     unpredictable animal in a useful direction. The LLM is the horse; the harness is the
     system around it.
   - Some practitioners colloquially call this "horse engineering" — same concept, emphasizing
     the underlying metaphor.
   - Important disambiguation found: Karpathy coined **context engineering** (Dec 2025) and
     **agentic engineering** (Feb 2026), NOT "harness engineering" — the term emerged
     separately from practitioner writing (Louis Bouchard, Augment Code, etc.).
   - The skill already covered harness engineering; added the horse-tack etymology note for
     clarity and to make the metaphor discoverable.

2. **"Loop engineering" = brand-new 2026 concept** (June 2026):
   - Coined/popularized by **Addy Osmani** (Google Engineering Director) on **June 8, 2026**:
     *"Loop engineering is replacing yourself as the person who prompts the agent. You design
     the system that does it instead."*
   - Followed by **LangChain's** "The Art of Loop Engineering" blog post and **Adnan Masood's**
     "Loop Engineering: A Guide for Engineers and Practitioners" (Medium, June 24, 2026).
   - Definition: Loop engineering is the **next layer above harness engineering**. The harness
     equips a *single* agent run; the loop is the *recurring control system* that triggers,
     verifies, and stops many agent runs.
   - Quote (Cobus Greyling): *"The harness equips a single agent run; the loop is what keeps
     poking agents on a schedule, spawning helpers, and feeding itself."*
   - **Anatomy:** trigger → verifiable goal → agent invocation (harness) → verification →
     iteration → stopping condition.
   - **Topologies:** single-shot, memory-aware, scheduled (cron), event-driven (webhook),
     nested, fan-out, self-feeding.
   - **Verification problem (biggest unsolved challenge):** 70–95% of agents fail in
     production (Fiddler AI, 2026). Common failures: verification gaming, goal drift,
     runaway loops, context pollution, state loss.
   - **Mitigations:** machine-checkable success criteria, independent verifier agent,
     human-in-the-loop, hard caps, never trust self-reports, durable state stores.
   - **Tools:** LangChain/LangGraph 1.0, LangSmith, Langfuse, Temporal, Inngest, Trigger.dev,
     Modal, E2B, Daytona, MLflow, Arize Phoenix, Helicone.

**Files updated:**

- `SKILL.md`:
  - Frontmatter: added "Loop Engineering" to description; added `loop-engineering` tag.
  - Step 1 technique table: added Loop Engineering row.
  - Step 2.5 heading: renamed to "Context Engineering, Harness Engineering & Loop Engineering".
  - Step 2.5 intro: expanded paradigm shift to four stages; added horse-tack etymology note;
    added harness-vs-loop disambiguation with Cobus Greyling quote.
  - Step 2.5: added new "Loop Engineering (June 2026 — the next layer above Harness)"
    subsection with anatomy, topologies, verification problem, 10-item checklist, full
    template, key sources.
  - Step 4 failure taxonomy: added 4 new loop-related failure modes (verification gaming,
    runaway loop/cost, loop context pollution, loop state loss on crash).
  - Reference Files list: noted horse-tack etymology + Loop Engineering in techniques.md.

- `references/model-guide.md`:
  - TOC: updated entry #19 to "Context Engineering, Harness Engineering & Loop Engineering".
  - H2 heading: renamed accordingly.
  - Paradigm shift section: updated from 3-stage to 4-stage (added Loop Engineering).
  - Added horse-tack etymology note after Harness Engineering Checklist.
  - Added new "Loop Engineering (June 2026 — the next layer above Harness)" subsection with
    definition, anatomy, harness-vs-loop disambiguation, topologies, verification problem,
    10-item checklist, tools, sources.

- `references/techniques.md`:
  - TOC: added Technique #31: Loop Engineering.
  - Section #22 (Harness Engineering): added "— a.k.a. 'Horse Engineering'" to title;
    added horse-tack etymology note and Karpathy disambiguation.
  - Added new Section #31: Loop Engineering — full deep dive with definition quotes from
    Addy Osmani, LangChain, Cobus Greyling; loop anatomy; 7 topologies; verification
    problem with failure modes and mitigations; 10-item checklist; full template; full
    example (CI Failure Auto-Fixer); tools; 7 key sources; when-NOT-to-use guidance.

- `references/templates-library.md`:
  - Added new "Loop Engineering — Scheduled Agent Loop Template (June 2026)" with full
    template (loop name, trigger, verifiable goal, agent harness, verification, stopping
    conditions, cost guardrails, observability, permission tiers).
  - Added Loop Engineering anti-patterns (6) and best practices (7) lists.
  - Added "Loop Engineering — Event-Driven Triage Loop Template" as second worked example
    for event-driven (vs scheduled) loops.

- `references/update-methodology.md`: this update's log.

**Rate-limit handling (per user request):**
- All 11 web searches run sequentially with 8–12 second delays between calls.
- Zero rate-limit hits (429 errors) during this update.
- Previous update hit 4 rate-limit errors when running 6 searches in parallel; sequential
  with delays is the correct pattern for this API.

**What did NOT change (still accurate from prior updates):**
- All model release dates, MCP spec, A2A, OWASP, EU AI Act — verified accurate.
- Harness engineering concept itself — only added etymology note and disambiguation,
  did not change existing checklist.

### End-of-June 2026 Update #1

**Trigger:** User requested a deep research sweep for missing/trending tools and asked to
upgrade the skill with anything new since the late-June 2026 update.

**Research method:** Attempted to deploy 6 parallel general-purpose research subagents
(frontier LLMs, agentic+MCP+A2A, image/video/audio gen, PromptOps/eval, security, coding
agents+memory+RAG). All 6 subagent calls failed with backend timeout
(`context deadline exceeded`). Pivoted to running ~60 targeted `web_search` queries
directly via the z-ai CLI, saved all results as JSON in `/home/z/my-project/research/`,
and generated a consolidated summary at `/home/z/my-project/research/SUMMARY.txt`.
Cross-validated all model release dates and discontinued-product claims against multiple
sources (vendor official pages, BBC, Snyk, official docs).

**Major findings applied:**

- **Image gen — added:**
  - **Ideogram 4.0** (June 3, 2026 — first **open-weight** Ideogram, 9.3B single-stream
    DiT, native 2048px, aspect ratios to 6:1). The skill previously listed Ideogram 3.0
    (closed); updated to 4.0 throughout.
  - **Stable Diffusion 4 Ultra** (Stability AI, 2026) — confirmed released with DiT
    architecture; flagged Stability's filmmaking pivot as vendor-pivot-risk.
  - **Imagen 4 deprecation** — Google is shutting down Imagen 4 standard/ultra/fast
    endpoints on **August 17, 2026**. Migrated all references to **Nano Banana Pro
    (Gemini 3 Pro Image)** as Google's current image family.
  - **Recraft V4.1** date confirmed (Feb 2026).

- **Video gen — added/tightened:**
  - **Luma Ray 3.14** (Jan 2026) — production-grade, native 1080p.
  - **Pika 2.2** — 4K rendering, character consistency.
  - **Grok Imagine Video 1.5** — clarified launch date **May 31, 2026**; #1 on
    Image-to-Video Arena leaderboard.
  - **Kling 3.0** — tightened release date to **Feb 4, 2026**; noted Kuaishou is spinning
    off Kling AI at reported $20B valuation (May 11, 2026).
  - **Seedance 2.0** — added audio-video joint generation as core strength.
  - **Veo 4 NOT released** — explicitly noted as teased at Google I/O 2026 but no official
    release date as of May 22, 2026.

- **Audio/voice/music gen — MAJOR NEW SECTION:**
  - Added new `## Audio / Voice / Music Generation` section to SKILL.md Step 7 with full
    4-layer stack (TTS, STT, Music, Character Animation).
  - **TTS:** Eleven v3 (70+ languages, expressive), Cartesia Sonic 3.5 (May 2026, sub-90ms
    latency, #1 for naturalness), Hume EVI 3 (any voice without fine-tuning), Sesame CSM
    (1B open-weights).
  - **STT:** ElevenLabs Scribe v2 (90+ languages, keyterm prompting), Deepgram Nova-3,
    Whisper v4, AssemblyAI Universal-2.
  - **Music:** Suno 5.5 (Mar 26, 2026, custom models), Lyria 3 / Lyria 3 Pro (Google),
    Stable Audio 2.5, Udio, ElevenLabs Music.
  - **Talking avatars:** Hedra (Feb 2026 public), HeyGen, D-ID.
  - Added **Technique #30: Voice/Music Stage-Direction Prompting** to techniques.md with
    TTS and music prompt templates + audio anti-slop rule.

- **Models — additions/corrections:**
  - Added **Qwen 3.7 Plus** (June 1, 2026, GA, low-cost multimodal agent model with vision
    + video understanding) alongside Qwen3.7-Max.
  - Updated **Grok 4.3** entry — confirmed $1.25/M input pricing, configurable reasoning
    (`none`/`auto`/`high`), native video input.
  - **MCP Tool Search** (Jan 14, 2026) — updated to reflect it's now available in **Claude
    Code** (not just the Claude Developer Platform).

- **Agentic frameworks — added 6 new frameworks:**
  - **PydanticAI** (type-safe Python, FastAPI-style)
  - **Smolagents** (HuggingFace minimalist code-thinking agents)
  - **DeerFlow 2.0** (ByteDance, June 23, 2026 — super-agent harness with sandbox+memory+skills+sub-agents)
  - **Vercel AI SDK** (TypeScript, Next.js-native, streaming-first)
  - **LlamaIndex AgentWorkflow** (event-driven multi-agent)
  - **Goose** (Block, AAIF-hosted open-source local-first agent)
  - Noted LangGraph leads 2026 adoption (~27K monthly searches vs CrewAI's ~15K).

- **MCP — additions:**
  - Spec section expanded: explicit mentions of **MCP Apps** (sandboxed HTML iframes),
    **Tasks extension**, **elicitations** (mid-flow user input).
  - **Ecosystem scale:** added Glama + PulseMCP (20,050+ servers), top servers (Playwright #1,
    GitHub, Figma, etc.).
  - Noted the **six SEPs** making MCP stateless-first.

- **Memory/RAG — added:**
  - **HippoRAG 2** — neurobiologically-inspired long-term memory, 10–30× cheaper than
    GraphRAG, 6× faster.
  - **MemGraphRAG** — memory-based multi-agent graph RAG (2026 paper).
  - **Graphiti** (Zep's graph engine).
  - Expanded vector DB stack: Pinecone serverless, Weaviate Embeddings 3.0, Qdrant, Chroma,
    Milvus 2.5+, pgvector, Astra DB, MongoDB Atlas Vector, Supabase Vector.
  - Added advanced RAG architectures: GraphRAG, LightRAG, nano-graphrag, Corrective RAG
    (CRAG), Adaptive RAG, Self-RAG, RAPTOR, HyDE.

- **PromptOps — added:**
  - **Giskard** — open-source LLM eval + vulnerability scanning.
  - **Langfuse** — open-source observability + prompt management.
  - **Inspect AI** (UK AISI) — security-focused eval.
  - DeepEval's **50+ built-in metrics** explicitly enumerated (G-Eval, hallucination
    detection, answer relevancy, contextual recall, faithfulness).
  - Promptfoo red-team **"most distinctive 2026 feature"** — auto-generates adversarial inputs.
  - Added the **2026 layered eval pipeline trend**: fast property checks (CI) → quality
    metrics (pre-release) → red-team (pre-deploy).
  - Added **prompt management platforms**: Vellum, Humanloop, Orq.ai, Portkey, Langfuse
    Prompts, Microsoft Promptflow, Pezzo.

- **Coding agents — added:**
  - **OpenCode, Kilo Code, Gemini CLI** — new free open-source coding agents.
  - Added coding agent pricing table (Copilot $10/mo, Cursor/Claude Code $20/mo, OS free).
  - Added Roo Code (Cline fork), Continue.dev, Goose (Block).

- **Security — additions:**
  - Expanded **Defense / Guardrail Platforms** section: Llama Guard 4, Llama Prompt Guard 2,
    NeMo Guardrails, Guardrails AI, AWS Bedrock Guardrails, Azure AI Content Safety,
    OpenAI Moderation API, Anthropic Constitutional Classifier (Jan 22, 2026), Lakera Guard,
    Rebuff.ai, LLM Guard (Protect AI), Cloudflare AI Gateway.
  - Added **Giskard** and **Inspect AI** to red-team tools.
  - Added new MCP attack classes: **Elicitation abuse** (phishing via new elicitations
    extension) and **MCP Apps sandbox escape attempts**.
  - Noted MCP server ecosystem scale (~20,050+ servers) as supply-chain risk.

**Files updated:**
- `SKILL.md` — frontmatter description + audio-generation tag; updated model defaults,
  MCP spec section, MCP Tool Search section, frameworks list, memory systems, image/video
  gen tables (with Imagen 4 deprecation callout), new Audio/Voice/Music gen section,
  PromptOps frameworks table with layered eval pipeline pattern, Reference Files list.
- `references/model-guide.md` — TOC updated; Google image family section rewritten
  (Imagen 4 deprecation + Nano Banana Pro migration); Stable Diffusion 4 Ultra section
  (was "unconfirmed" → now released); Ideogram 4.0 section (was 3.0); video leaderboard
  with 4 new entries + Kuaishou spin-off note; audio section completely rewritten into
  4-layer stack; coding agents expanded with open-source options + pricing; Qwen section
  updated with Qwen 3.7 Plus; MCP section updated with MCP Apps/Tasks/Elicitations +
  ecosystem scale.
- `references/techniques.md` — TOC updated; RAG section expanded with advanced
  architectures and vector DB stack; MCP Tool Search updated (Claude Code availability);
  DSPy section expanded with new optimizers + prompt management platforms + layered eval
  pipeline pattern; **new Technique #30: Voice/Music Stage-Direction Prompting** with
  templates and audio anti-slop rule; Deprecated table updated with Imagen 4, Ideogram
  3.0, SD 3.5 dominance.
- `references/templates-library.md` — added new **End-of-June 2026 Additions** section
  with 5 new templates: Ideogram 4.0 (text-in-image), Eleven v3 (voice direction),
  Suno 5.5 (full song), Hedra (talking avatar), Veo 3.1 (cinematic video with audio).
- `references/security.md` — added **Defense / Guardrail Platforms** table; added
  Giskard and Inspect AI to red-team tools; expanded MCP attack classes with Elicitation
  abuse and MCP Apps sandbox escape; added MCP server ecosystem scale note; updated
  Promptfoo red-team to highlight auto-adversarial-input generation.
- `references/update-methodology.md` — this update's log.

**What did NOT change (still accurate from late-June 2026 update):**
- Claude Opus 4.8 (May 28, 2026), GPT-5.6 (June 26, 2026), Gemini 3.5 Flash (May 19, 2026),
  GLM-5.2 (June 13, 2026), DeepSeek V4 (April 24, 2026), Grok 4.3 (April 30, 2026),
  Qwen3.7-Max (May 19, 2026), Meta Muse Spark (April 9, 2026) — all dates and specs
  verified accurate.
- Fable 5/Mythos 5 suspension (June 12, 2026) — confirmed via BBC, Anthropic, Snyk.
- Sora discontinuation (April 26, 2026 web/app; Sept 24, 2026 API) — confirmed.
- MCP 2026-07-28 RC (locked May 21, finalizes July 28, 2026) — confirmed.
- A2A protocol (150+ orgs, April 9, 2026 production-ready) — confirmed.
- Lethal Trifecta / Rule of Two (Simon Willison + Meta AI) — confirmed.
- OWASP Top 10 for Agentic Applications (ASI, Dec 9, 2025) — confirmed.
- EU AI Act full enforcement August 2, 2026 — confirmed.
- Reasoning-effort dials across vendors — confirmed.

### Late-June 2026 Update

**Research method:** 4 parallel general-purpose subagents researching:
1. Latest frontier LLM models (Anthropic, OpenAI, Google, xAI, DeepSeek, GLM, Qwen, Meta, open-weights)
2. Agentic + MCP + context engineering developments
3. Image/video/audio generation prompting
4. Prompt security, OWASP, NIST, EU AI Act, jailbreak research

**Major findings applied:**

- **Models:** Added Claude Opus 4.8 (May 28, 2026), GPT-5.6 Sol/Terra/Luna (June 26, 2026,
  limited preview), Gemini 3.5 Flash (May 19, 2026) + Gemini 3.5 Pro (announced, slipped
  to July 2026), Grok 4.3 (April 30, 2026 — corrected: Grok 5 has NOT been released),
  DeepSeek V4 (April 24, 2026 — corrected: R2 has NOT been released), GLM-5.2 (June 13,
  2026), Qwen3.7-Max (May 19, 2026), MiniMax M3 (late June 2026), Meta Muse Spark
  (April 9, 2026 — replaces Llama as Meta flagship).
- **Discontinued:** Sora web/app closed April 26, 2026; Sora API sunsets Sept 24, 2026.
  Removed from video leaderboard; migrated prompting guidance to Veo 3.1, Kling 3.0,
  Seedance 2.0, Grok Imagine 1.5.
- **New paradigm:** Harness engineering (Karpathy, Feb 2026) — extends context engineering
  to include tools, memory, runtime. Added as Step 2.5 alongside context engineering.
  Karpathy joined Anthropic May 2026.
- **MCP governance:** Donated to Linux Foundation's Agentic AI Foundation (AAIF) in Dec 2025,
  co-founded by Anthropic, OpenAI, Block. 2026-07-28 spec RC (stateless rewrite, locked May 21,
  finalizes July 28, 2026).
- **A2A protocol:** Google Cloud's agent-to-agent protocol (150+ organizations in year one).
  Added as companion to MCP.
- **MCP Tool Search:** Jan 14, 2026 — dynamic tool loading, the single most important 2026
  tool-calling change. Added as new technique.
- **Agent Skills:** Anthropic's 2026 capability packaging. Added as new technique and template.
- **Image gen shifts:** Midjourney V8.1 (default June 10, 2026) — personalization-first
  prompting, longer prompts work, `--cref`/`--oref` removed. gpt-image-2 (April 21, 2026) —
  5-slot template, anti-slop rule. Google Nano Banana / Imagen 4. FLUX.2 family. Recraft V4.1.
- **Video gen:** Veo 3.1 (Jan 2026, 4K + native audio), Gemini Omni (May 2026, any-input →
  video world model), Kling 3.0 (native 4K 60fps), Seedance 2.0, Runway Gen-4.5, Grok
  Imagine 1.5, Wan 2.6.
- **Multi-agent patterns:** Consolidated to 5 dominant production patterns (fan-out, pipeline,
  debate, supervisor, swarm). Added two 2026 academic surveys.
- **Subagent orchestration:** Claude Opus 4.8 dynamic workflows, GPT-5.6 ultra mode, Kimi
  K2.6 Agent Swarm.
- **Reasoning-effort dials:** Standardized across vendors (low/high/extra/max for Claude,
  max + ultra for GPT, adaptive for Gemini, configurable for Grok, maximum mode for DeepSeek).
- **Security:** Added OWASP Top 10 for Agentic Applications (ASI, Dec 9, 2025) alongside
  LLM Top 10 (2025). Added Lethal Trifecta / Rule of Two (Simon Willison / Meta AI).
  Added Spotlighting (Microsoft). Added MCP attack classes (Tool Poisoning, Rug Pull,
  Confused Deputy, MCP-ITP). Added EU AI Act enforcement date (Aug 2, 2026). Added
  OpenAI IH-Challenge dataset (March 2026). Added Anthropic's new Constitution (Jan 22, 2026).
  Added red-teaming tools (Garak, PyRIT, Promptfoo red-team, Arthur Bench, MCP-SafetyBench,
  NRT-Bench).
- **Benchmarks:** Added τ-bench (tau-bench), SWE-Bench Pro, Terminal-Bench, AppWorld,
  MCP-SafetyBench, NRT-Bench — the 2026 theme is "benchmarks closer to production reality
  produce dramatically lower scores."
- **Context engineering:** Added Anthropic's Write/Select/Compress/Isolate quartet (Sept 2025,
  still canonical in 2026). Added memory systems (Mem0, Letta, Zep, Cognee, Redis LangCache).
- **Production case studies:** Shopify River (Slack-native coding agent), Salesforce Agentforce
  Help Agent (June 25, 2026), Klarna cautionary tale.

**Files updated:**
- `SKILL.md` — rewrote with all H1/H2 2026 updates
- `references/model-guide.md` — rewrote with late-June 2026 model landscape
- `references/techniques.md` — rewrote with new techniques (Harness Engineering, Agent Skills,
  MCP Tool Search, A2A, Subagent Orchestration, Anti-Slop, Personalization-Driven)
- `references/templates-library.md` — appended "Late-June 2026 Additions" section with 13 new
  templates
- `references/security.md` — rewrote with OWASP ASI, Lethal Trifecta, MCP attack classes,
  EU AI Act, IH-Challenge, new red-teaming tools
- `references/update-methodology.md` — this file

### June 2026 Update (original)

**Trigger:** Skill referenced outdated model versions (Claude 4.6, GPT-5.4, Gemini 2.5).
**Method:** 5-10 parallel web searches across model/technique/tool categories.
**Major findings applied:**
- Updated models to Claude Opus 4.7, GPT-5.5, Gemini 3.1 Pro
- Added "Be honest" sycophancy fix → literal/contract-style shift
- Added video generation (Sora 2 Pro, Veo 3.1, Kling 3.0, Seedance 2.0)
- Added MCP under Agentic AI Foundation
- Added PromptOps section with Promptfoo, DeepEval, RAGAS, LangSmith
- Added agent runbook pattern

## Pitfalls

- **Don't trust single sources** — cross-validate model release dates and prices with
  multiple web searches
- **Don't add every new model** — focus on frontier/frequently-used models (Claude, GPT,
  Gemini, DeepSeek, GLM, Grok); skip niche ones
- **Don't remove legacy content** — keep sections like Claude 4.7 in a "migration guide"
  pattern so users upgrading existing workflows can find the changes
- **Don't overspecify dates** — use "mid-[year]" or "H1 [year]" instead of specific months;
  the skill lives longer than any one update session
- **Verify "released" claims for next-gen models** — Veo 4, Kling 4, Runway Gen-5, Grok 5,
  Llama 5, DeepSeek R2 were all rumored/announced but NOT released as of late June 2026.
  Always check the vendor's official model page before writing "X is released."
- **Check for discontinued products** — Sora was discontinued April 2026 but listicles still
  reference it. Update leaderboards to remove discontinued products.
- **Watch for CRLF** — files extracted from zip archives may have CRLF line endings, which
  cause the Edit tool to fail silently. Run `sed -i 's/\r$//'` on all files before editing.
- **Note government intervention** — Anthropic's Fable 5/Mythos 5 was suspended June 12,
  2026 by US export-control directive; GPT-5.6 launched June 26 into a government-coordinated
  limited preview. Flag these constraints in the skill so users don't assume the most capable
  tier is reliably available.

## Maintenance Cadence

Recommended cadence: **every 2-3 months**, or immediately after:
- A major model release (new Opus, GPT, Gemini, etc.)
- A paradigm-shifting paper/blog (like Karpathy's "agentic engineering")
- A new OWASP/NIST/regulatory publication
- A major vendor product discontinuation (like Sora)

The prompt engineering landscape moves fast — stale skills lose triggering accuracy and
provide wrong guidance. The description field in particular needs current model names so
GLM correctly triggers this skill when users mention them.
