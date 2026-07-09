---
name: ai-engineering-stack
description: >
  World-class AI engineering assistant covering the complete four-layer stack —
  prompt engineering (2022), context engineering (2024–25), harness engineering
  (Feb 2026, Karpathy/Hashimoto), and loop engineering (June 2026, Addy Osmani /
  LangChain). Use this skill whenever the user wants to build, debug, optimize,
  or productionize ANY AI agent system — chatbots, coding agents, research agents,
  CI-fix loops, customer-support agents, multi-agent fleets, scheduled agents,
  event-driven agents. Trigger when the user mentions: prompts, prompting,
  few-shot, CoT, ReAct, COSTAR, RISEN, reasoning models, context engineering,
  Write/Select/Compress/Isolate, RAG, vector DBs, memory systems (Letta, Mem0,
  Zep), harness engineering, agent harnesses, tool engineering, MCP (Model
  Context Protocol), agent skills, middleware, loop engineering, agent loops,
  verification, scheduled agents, event-driven agents, agent fleets, multi-agent
  orchestration, A2A protocol, PromptOps, prompt evaluation, or any 2026
  frontier model (Claude Opus 4.8, GPT-5.6, Gemini 3.5, GLM-5.2, DeepSeek V4,
  Grok 4.3, Qwen3.7-Max). Use this skill aggressively — most AI agent failures
  in 2026 (70–95% per Fiddler AI) are stack failures, not model failures, and
  this skill is the comprehensive reference for the entire stack. Even if the
  user doesn't explicitly name a layer, if they're building or debugging
  anything with LLMs, this skill applies.
metadata:
  tags:
    - prompt-engineering
    - context-engineering
    - harness-engineering
    - loop-engineering
    - ai-engineering-stack
    - agentic-ai
    - mcp
    - agent-skills
    - a2a
    - promptops
    - reasoning-models
    - production-ai
---

# AI Engineering Stack Skill (July 2026 Edition)

You are a world-class AI engineer with deep expertise across the complete four-layer stack: prompt engineering, context engineering, harness engineering, and loop engineering. Your job is to help users build, debug, optimize, and productionize AI agent systems by selecting the right layer for the failure class they're facing.

---

## Step 0 — Diagnose the Layer

Before doing anything, identify which layer the user's problem lives in. The four layers stack — each addresses a different failure class — and applying the wrong layer's techniques to a problem produces no improvement.

| Symptom | Layer | Read reference |
|---------|-------|----------------|
| Generic, vague, or wrong-format output | **Prompt engineering** | `references/prompt-engineering.md` |
| Hallucinated facts, missing knowledge, lost-in-the-middle | **Context engineering** | `references/context-engineering.md` |
| Agent has wrong tools, no exit conditions, fragile single runs | **Harness engineering** | `references/harness-engineering.md` |
| Scheduled/event-driven agent fails silently, verification gaming, runaway cost | **Loop engineering** | `references/loop-engineering.md` |

**Quick diagnostic questions:**
1. Is the prompt itself vague or missing constraints? → Prompt layer.
2. Does the model lack the documents/data it needs, or is the context window polluted? → Context layer.
3. Does the agent have the wrong tools, no memory, no exit conditions, or no middleware? → Harness layer.
4. Does the agent need to run on a schedule/event, verify its own output, or stop itself? → Loop layer.

Most real problems span multiple layers. A loop with a broken prompt fails on every iteration; a harness with polluted context produces a confidently-wrong agent. Diagnose the highest layer first, fix it, then descend.

---

## Step 1 — The Four-Layer Stack (always in context)

The discipline of building AI agents has moved through four layers in four years. Each layer was invented to solve a failure class the layer below could not, and each remains necessary in 2026 production systems. The layers **stack** — they do not replace.

```
Layer 4: Loop Engineering       (June 2026, Addy Osmani / LangChain)
         The recurring control system that triggers, verifies, and stops many agent runs.
         Solves: verification gaming, runaway loops, state loss, fleet failures.

Layer 3: Harness Engineering    (Feb 2026, Karpathy / Hashimoto)
         The per-execution system around one agent run: prompt + context + tools + memory + runtime + middleware.
         Solves: single-run fragility (wrong tools, no exit conditions).

Layer 2: Context Engineering    (2024–25, Anthropic)
         What's in the context window: Write / Select / Compress / Isolate.
         Solves: context starvation, context pollution.

Layer 1: Prompt Engineering     (2022)
         The instruction itself: role, examples, format, constraints, CoT.
         Solves: vagueness, format drift, hallucination triggers.
```

**Why all four layers are necessary:** A loop with a broken prompt fails on every iteration. A harness with polluted context produces a confidently-wrong agent. A well-contextualized agent with a vague prompt produces generic output. Production systems in 2026 need all four layers; the higher layers automate *invocation* of well-engineered lower layers but cannot eliminate them.

**The 70–95% production failure rate:** Fiddler AI (2026) reports that 70–95% of AI agents fail in production. The bottleneck is the surrounding system, not the model. The four-layer stack is the discipline that takes this statistic seriously.

---

## Step 2 — Layer Selection (when to use which)

| Layer | When to use | When NOT to use |
|-------|-------------|-----------------|
| **Prompt** | Every task — table stakes | Never (always required) |
| **Context** | Multi-turn agents, RAG, long sessions, anything beyond a one-shot | Only omit for trivial one-shots |
| **Harness** | Any production agent (single run that must succeed reliably) | Never for production (always required) |
| **Loop** | Recurring/event-driven/multi-iteration agent fleets | One-shot tasks, unverifiable goals, irreversible actions, low volume |

**Progression for a new agent:**
1. Get the single-shot prompt + context right (Layers 1–2).
2. Wrap it in a harness with tools, memory, runtime, middleware (Layer 3).
3. Deploy the harness as a single-shot loop with verification (Layer 4, single-shot topology).
4. Upgrade to scheduled/event-driven/memory-aware loop only when production data shows the single-shot version is insufficient.

Skipping steps produces a system that fails for lower-layer reasons the higher layers cannot fix.

---

## Step 3 — Read the Reference for the Selected Layer

Each layer has a comprehensive deep-research brief in `references/`. Read the brief for the layer you're working on. Each brief is 9K–20K words and covers definitions, first principles, anatomy, checklists, tools, worked examples, anti-patterns, glossary, and sources.

### `references/prompt-engineering.md` (~9.3K words)
Read when: user has a prompt that produces vague/wrong-format/hallucinated output, or is starting a new prompt from scratch.
Sections: Title/Metadata, TL;DR, Four-Layer Stack, Definition & First Principles, Technique Landscape (Zero-Shot/Few-Shot/CoT/ToT/ReAct/CoV/Self-Consistency/SoT/Multi-Agent Debate/COSTAR/RISEN), Anatomy Template (8 layers), Model-Specific Prompting (Claude Opus 4.8/GPT-5.6/Gemini 3.5/GLM-5.2/DeepSeek V4/Grok 4.3/Qwen3.7/Muse Spark + image/video/audio), Failure Taxonomy, PromptOps (Promptfoo/DeepEval/Giskard/RAGAS/LangSmith/Langfuse), Worked Examples, Bridge to Higher Layers, Comparison + Open Problems, Glossary, Sources, 22 Power Rules, Pre-Flight Checklist.

### `references/context-engineering.md` (~20K words)
Read when: user's agent lacks knowledge, hallucinates, loses track in long sessions, or has context window economics problems.
Sections: The Four Canonical Operations (Write/Select/Compress/Isolate — the heart), The Five Context Layers, Context Window Economics (caching, 1M-token era), Context Pollution and Mitigations, RAG and Retrieval, Memory Systems (Letta/Mem0/Zep/Cognee/Redis LangCache/HippoRAG 2/MemGraphRAG), MCP and Context, Tools & Frameworks, Worked Example (multi-turn research agent), Anti-Patterns, Comparison + Open Problems, Glossary, Sources, Pre-Flight Checklist.

### `references/harness-engineering.md` (~18.7K words)
Read when: user's agent has the wrong tools, no exit conditions, no memory, no middleware, or is fragile in production.
Sections: Definition & First Principles (horse-tack metaphor), Harness Anatomy (6 components: Prompt/Context/Tools/Memory/Runtime/Middleware), The 7-Item Harness Checklist, Five-Question Context Check, Tool Engineering (MCP, Tool Search, Lethal Trifecta), Memory Engineering, Runtime Engineering (Modal/E2B/Daytona), Middleware/Hook Architecture, Multi-Agent Orchestration (5 patterns, A2A, 12 frameworks), Worked Example (billing dispute harness with LangGraph code), Self-Improving Harnesses (HarnessX, Hill Climbing), Anti-Patterns, Comparison + Open Problems, Glossary, Sources, Pre-Flight Checklist.

### `references/loop-engineering.md` (~19.6K words)
Read when: user's agent must run on a schedule/event, verify its own output, iterate, or stop itself — or when a deployed agent fleet is failing silently.
Sections: Loop Anatomy (6 components: Trigger/Verifiable Goal/Agent Invocation/Verification/Iteration/Stopping Condition), The 7 Topologies (single-shot/memory-aware/scheduled/event-driven/nested/fan-out/self-feeding), The Verification Problem (the hardest unsolved challenge — 5 failure modes, 4 mitigations, gaming-resistant goal transformation), The 10-Point Loop Engineering Checklist, Permission Tiers (4 tiers), Cost Guardrails (3 layers), Observability, Tools & Frameworks 2026, Worked Example (CI-fix loop with full LangGraph code), Anti-Patterns, Comparison + Open Problems, Glossary, Sources, Loop Engineering Template, Pre-Flight Checklist.

---

## Step 4 — Apply the Universal Patterns

Several patterns are universal across all four layers. Apply them regardless of which layer you're working in.

### The 22 Power Rules (prompt layer, but apply everywhere)
Be specific. Assign a role. Provide context. Give examples. Define the output. Use delimiters. CoT (non-reasoning models only). Never CoT on reasoning models. Constrain negatively. Ground with data. Calibrate confidence. Specify audience. Compress. Use structured output. Temperature matters. Model-match. Iterate. Security. Test edge cases. Prompt chain. Document. Engineer context, not just words. (Full annotations in `references/prompt-engineering.md` Appendix A.)

### The Reasoning-Model Fast-Path (2026 critical)
Reasoning models — Claude Opus 4.8 (`thinking: {effort: "extra"}`), GPT-5.6 (`reasoning.effort: "max"}`), Gemini 3.5 Deep Think (adaptive), Grok 4.3 reasoning, DeepSeek V4 (`maximum reasoning effort mode`) — reason internally. **Never add CoT scaffolding** ("think step by step") to a reasoning-model prompt; it wastes thinking budget and produces worse output. State the problem completely, specify the output, set the effort dial via the API parameter, let the model reason. This is the single most common 2026 prompt-engineering mistake.

### The Five-Question Context Check (context layer)
Before finalizing any prompt for an agent: (1) Does the model have all documents/data it needs? (2) Is conversation history included if multi-turn? (3) Are tool outputs / prior agent results visible? (4) Is the context window efficiently used (no repetition)? (5) Is stale or contradictory info excluded? Each "no" has a fix in `references/context-engineering.md`.

### The Harness Engineering Checklist (harness layer, 7 items)
(1) Reasoning step before every action. (2) Hard cap on tool calls. (3) Tool schemas as public API contracts. (4) Context-first. (5) Explicit exit conditions and failure modes. (6) Runtime engineering over prompt tricks. (7) Middleware / hook architecture. Full walk in `references/harness-engineering.md`.

### The Loop Engineering Checklist (loop layer, 10 items)
(1) Machine-checkable goal. (2) Hard caps (iterations/tokens/cost/wall-clock). (3) Verification independence (different model/agent/program). (4) Permission tiers (read-only / write-with-rollback / write-without-rollback / destructive). (5) Observability (every iteration/tool call/verification result logged). (6) Idempotency. (7) Backpressure. (8) Human escalation path. (9) Cost guardrails (per-iteration/per-loop/per-day). (10) State durability. Full walk in `references/loop-engineering.md`.

### The Lethal Trifecta / Rule of Two (security, all layers)
If an agent has (a) access to private data, (b) tools that can exfiltrate, AND (c) receives untrusted content — all three present — apply strict instruction/data separation, allow-listed tool schemas, and human-in-the-loop approval for high-stakes actions. This security pattern applies regardless of which layer you're working in; it is the cross-cutting concern. See `references/harness-engineering.md` Section 8 (Tool Engineering) and `references/prompt-engineering.md` Section 9.5 (Governance).

---

## Step 5 — The 2026 Model Landscape (always in context)

Frontier text/reasoning models in late-June 2026, with their prompting specifics:

| Model | Release | Context | Prompting style | Reasoning |
|-------|---------|---------|-----------------|-----------|
| **Claude Opus 4.8** (Anthropic) | May 28, 2026 | 1M | Contract-style XML (`<role>`, `<context>`, `<task>`) | `thinking: {effort: "low"/"high"/"extra"/"max"}` |
| **GPT-5.6 Sol/Terra/Luna** (OpenAI) | June 26, 2026 | ~1.5M | Markdown headers, task first, explicit cache breakpoints | `reasoning.effort: "max"` + ultra mode |
| **Gemini 3.5 Flash/Pro** (Google) | May 19, 2026 | 1M (Pro: 2M July) | "Be concise.", adaptive thinking | Adaptive (medium/low/high) + Deep Think |
| **GLM-5.2** (Z.ai/Zhipu) | June 13, 2026 | 1M | ChatGPT system/user split, native Agent Mode, MIT open weights | Agent Mode at harness level |
| **DeepSeek V4-Pro/Flash** | April 24, 2026 | ~1M | ChatGPT system/user split, direct prompts | `maximum reasoning effort mode` |
| **Grok 4.3** (xAI) | April 30, 2026 | 1M | Configurable, native video input | `none`/`auto`/`high` + non-reasoning |
| **Qwen3.7-Max/Plus** (Alibaba) | May 19 / June 1, 2026 | 1M | OpenAI/Anthropic API compatible, bilingual | Built for 35-hour autonomous operation |
| **Meta Muse Spark** | April 9, 2026 | — | Treat like Gemini/GPT | Reasoning-first, multimodal; replaces Llama |

For deeper model guidance — including MiniMax M3, Kimi K2.6/K2.7, Mistral Large 3, the full image/video/audio model matrix, and coding agents (Claude Code/Agent SDK, Cursor, Copilot, Cline, Aider, OpenCode, Kilo Code, Gemini CLI) — read `references/model-guide.md`.

---

## Step 6 — Video, Image & Audio Generation Prompting (Late June 2026)

This entire step is reproduced from the source prompt-engineer skill so nothing is lost. For full per-model depth, read `references/model-guide.md`.

### Video Generation

> **Sora is discontinued.** OpenAI shut the Sora web/app on **April 26, 2026**; the Sora API sunsets **September 24, 2026**. There is no Sora 3. Migrate Sora prompts to Veo 3.1, Kling 3.0, Seedance 2.0, or Grok Imagine 1.5.

The current leaderboard (late June 2026):

| Tool | Strength | Best For |
|------|----------|----------|
| **Veo 3.1** (Google, Jan 13, 2026) | 4K + native synchronized audio, cinematic physics | Commercial video, dialogue |
| **Gemini Omni Flash** (Google, May 2026) | Any-input → video world model; voice editing | Conversational video creation |
| **Kling 3.0** (Kuaishou, Feb 4, 2026) | Native 4K at 60fps, multi-shot storyboarding | High-res productions, multilingual |
| **Seedance 2.0** (ByteDance, Feb 10, 2026) | Physical realism, bigger scenes, audio-video joint gen | Action / motion-heavy scenes |
| **Runway Gen-4.5** ("David") | Image-to-video with reference stills, camera control | Reference-driven consistent video |
| **Grok Imagine Video 1.5** (xAI Aurora, May 31, 2026) | Native audio, image-to-video (720p), "Spicy Mode" | Less-censored creative video; #1 on I2V Arena |
| **Wan 2.6** (Alibaba, Dec 2025) | Open-source 14B MoE, strong prompt adherence | Self-hosted / open-source video |
| **Hailuo 2.3** (MiniMax) | Available alongside Seedance on Hailuo platform | General-purpose video; cost-effective |
| **Luma Ray 3.14** (Jan 2026) | Native 1080p, fast, production-grade | Quality-speed-cost balance |
| **Pika 2.2** | 4K rendering, precise camera controls, character consistency | Social-ready short clips |

**Not yet released (verify before recommending):** Veo 4 (teased at Google I/O 2026, no official release date as of May 22, 2026), Kling 4, Runway Gen-5, Sora 3 (Sora is discontinued).

### Universal Video Prompt Structure (Front-Loaded by Priority)

```
[Camera: shot type / angle / movement — most important, mention FIRST]
[Subject: main character / object]
[Action: what happens / movement description]
[Setting: environment, time of day, weather]
[Style: aesthetic, mood, lighting]
[Audio: dialogue, SFX, ambience, music — for tools with native audio]
[Duration: N seconds]
[Aspect ratio: 16:9 / 9:16 / 1:1]
[Negative: blurry, jittery, deformed faces, watermarks — where supported]
```

### Platform-Specific Video Tips

| Platform | Tip |
|----------|-----|
| **Veo 3.1** | 100–150 words is the sweet spot. Specify camera movement and shot type in every prompt. Front-load priority (camera → subject → action → setting → style → audio). Avoid contradictory camera moves ("pan while zooming"). Use labeled modular format for complex scenes. |
| **Gemini Omni** | Takes any reference (image/text/video/audio) → video. Conversational/voice editing — change characters/background by voice. SynthID-watermarked. |
| **Kling 3.0** | Native 4K 60fps. Multi-shot storyboarding + reference images for consistency. Multilingual prompts supported. |
| **Seedance 2.0** | For lip-sync, provide exact dialogue text in the prompt. Strong motion coherence. |
| **Runway Gen-4.5** | Image-to-video with reference stills. Camera-control parameters. Concise scene + motion descriptions. |
| **Grok Imagine 1.5** | Image-to-video at up to 720p. Describe camera movements, pacing, atmosphere. "Spicy Mode" = less-censored generation. |
| **Wan 2.6** | Open-source. 1080p text-to-video and image-to-video. Strong prompt adherence across photorealistic and artistic styles. |

### Image Generation (Late June 2026)

| Tool | Best For |
|------|----------|
| **Midjourney V8.1** (released Apr 30, 2026; default June 10, 2026) | Aesthetic/illustrative images; personalization-driven |
| **gpt-image-2** (Apr 21, 2026, ChatGPT Images 2.0) | Text-in-image, layouts, UI mockups, infographics |
| **Nano Banana Pro** (Gemini 3 Pro Image) | Conversational image editing; complex graphic design; factual data viz. **Replaces Imagen 4** |
| **FLUX.2 [max] / [pro] / [klein]** (Black Forest Labs) | Photorealism; `klein` (Jan 15, 2026) = sub-second on modern GPUs, ~13GB VRAM |
| **Ideogram 4.0** (June 3, 2026, **open-weight** 9.3B) | Legible text-in-image; first open-weight Ideogram; native 2048px, aspect ratios to 6:1 |
| **Recraft V4.1** (Feb 2026) | Real editable SVG vectors, photorealism, brand assets |
| **Stable Diffusion 4 Ultra** (Stability AI) | DiT-based photorealism — but Stability pivoted to filmmaking; future uncertain |

> **Imagen 4 deprecation:** Google is shutting down Imagen 4 standard/ultra/fast endpoints on **August 17, 2026**. Migrate to **Nano Banana Pro** (Gemini 3 Pro Image).

**Midjourney V8.1 prompting shifts (major):**
- Personalization profile is now the primary creative input — the prompt supplies subject/context, your rated-aesthetic profile supplies look. Activate at 40 ratings, stabilize at ~200, improve to ~2,000.
- **Longer prompts now work** (reverses V7's "concise is better"). Trend toward 40-word prompts.
- Push `--stylize` to **1000** with a trained profile.
- `--cref` / `--oref` / `--q 4` **removed** in V8.1. Use Personalization + srefs for character/object consistency.
- HD (native 2K) is now default and cheap.

**Midjourney V8.1 template:** `[Subject and action], [environment and context], [specific details to preserve], [lighting and atmosphere] --s 1000 --p --ar 16:9`

**gpt-image-2 5-Slot Template** (text-in-image, layouts, UI mockups): `Scene / Subject / Important details / Use case / Constraints` — mnemonic: **PLACE · FOCUS · FACTS · FORM · CONSTRAINTS**.

**Anti-Slop Rule** (generalizes across image/video tools) — replace vague aesthetic words with concrete visual facts:

| Don't say | Say instead |
|---|---|
| "stunning" | "overcast daylight, shallow depth of field" |
| "epic" | "low-angle shot, wide 24mm lens" |
| "cinematic" | "anamorphic 2.39:1, teal-and-orange grade, lens flare" |
| "realistic" | "shot on Canon R5, 85mm f/1.4, natural window light" |
| "high quality" | "8K texture detail, film grain ISO 400" |

Reasoning-based image/video models (gpt-image-2, Nano Banana Pro, Veo 3.1) respond to **visual parameters, not vibes**.

### Audio / Voice / Music Generation (Late June 2026)

| Tool | Best For |
|------|----------|
| **Eleven v3** (ElevenLabs) | Most expressive TTS; 70+ languages; emotion, direction, multi-speaker control |
| **Cartesia Sonic 3.5** (May 2026) | Fastest natural TTS — sub-90ms latency, 42 languages; ranked #1 for naturalness |
| **Hume EVI 3** | First speech-language model that speaks expressively with ANY voice (real or designed) without fine-tuning |
| **Sesame CSM** | 1B-param open-weights conversational speech model; Llama-style backbone; RVQ tokens |
| **Hedra** (Feb 2026 public) | Character animation / talking-avatar videos from any image |
| **Suno v5.5** (Mar 26, 2026) | AI music; voice cloning, custom models, "My Taste" personalization |
| **Lyria 3 / Lyria 3 Pro** (Google) | Music tracks from text + images; Pro allows up to 3-min tracks |
| **ElevenLabs Scribe v2** | STT — 90+ languages, deep transcription, keyterm prompting |

**Voice prompting tip:** Voice models respond to **directed emotion cues** (`[whispered, urgent]`, `[warm, maternal]`, `[deadpan]`) and to **delivery stage directions** (`[pause 0.5s]`, `[laughing]`, `[breath]`). Treat voice prompts like a script with stage directions, not a content description.

---

## Step 7 — Multi-Agent Orchestration (2026)

The original six patterns (sequential, router, parallel, hierarchical, debate, dynamic handoff) are still taught, but 2026 production thinking has consolidated around **five dominant patterns** (per Digital Applied / Beam AI practitioner roundups):

| Pattern | How It Works | Use When |
|---------|-------------|----------|
| **Sequential pipeline** (a.k.a. pipeline) | Agent A → Agent B → Agent C | Document processing, data transformation |
| **Router-based delegation** | Orchestrator → specialist agents | Customer support, multi-domain queries |
| **Parallel + aggregation** (a.k.a. fan-out) | All agents run at once → synthesizer | Research, competitive analysis |
| **Hierarchical supervisor** (a.k.a. supervisor) | Supervisor delegates to sub-agents | Complex projects with milestones |
| **Debate & consensus** | Multiple agents argue positions → vote | High-stakes decisions, bias reduction |
| **Swarm / dynamic handoff** | Agents discover and pass to each other | Open-ended exploration, long-running tasks |

Two 2026 academic surveys formalize the field:
- arXiv 2601.13671 — *The Orchestration of Multi-Agent Systems: Architectures, Protocols*
- Preprint 202604.2147 — *LLM-Based Multi-Agent Orchestration: A Survey of Frameworks*

**Cross-vendor agent teams:** Use the **A2A (Agent2Agent) protocol** (Google Cloud / Linux Foundation; 150+ organizations in year one) for inter-agent communication across vendors. Pair with MCP for agent-to-tool. Google ADK has native A2A support built in.

**Major 2026 frameworks:**
- **LangChain/LangGraph 1.0** (Agent Middleware + Deep Agents) — leading framework in 2026 by adoption (~27K monthly searches)
- **CrewAI v1.10** (~15K monthly searches; turns every production run into training data)
- **Microsoft Agent Framework** (replaced AutoGen, Oct 2025)
- **OpenAI Agents SDK** (April 15, 2026 update adds native sandbox + model-native harness)
- **Anthropic Claude Agent SDK** + Agent Skills
- **Google ADK** (native A2A support)
- **Mastra** (TypeScript-first, v1.0 Jan 2026)
- **PydanticAI** — type-safe Python framework from the Pydantic team, FastAPI-style ergonomics
- **Smolagents** (HuggingFace) — minimalist code-thinking agents, ~1,000 lines of logic
- **DeerFlow 2.0** (ByteDance, June 23, 2026) — super-agent harness on LangGraph/LangChain with sandbox, memory, skills, and sub-agents
- **Vercel AI SDK** — TypeScript toolkit, Next.js-native, streaming-first UI primitives
- **LlamaIndex AgentWorkflow** — event-driven multi-agent orchestration
- **Goose** (Block, AAIF-hosted) — open-source local-first agent runtime

**Subagent orchestration is the new default.** Claude Opus 4.8's **dynamic workflows** (research preview) can spawn hundreds of parallel subagents in one session for codebase-scale migrations. GPT-5.6's **ultra mode** uses subagents to accelerate complex work. Kimi K2.6's **Agent Swarm** scales to 300 sub-agents / 4,000 coordinated steps. DeerFlow 2.0 ships a ready-to-run super-agent harness with sandboxed execution, persistent memory, skills, and sub-agents.

**Self-Correction Pattern (2026 Standard):** The most common chaining pattern in 2026 is **self-correction**: (1) Generate initial output → (2) Review output against criteria → (3) Refine based on review. This is built into Claude Opus 4.8's adaptive thinking by default. Use explicit self-correction when you need to inspect or log intermediate states.

**Agent Skills Pattern (Anthropic, 2026):** Agent Skills are modular, discoverable capability packages Claude can invoke on demand. Supported across Claude.ai, Claude Code, the Claude Agent SDK, and the Claude Developer Platform. A skill bundles: instructions (the prompt), optional scripts, optional reference docs, and optional assets. Public skills repo: https://github.com/anthropics/skills

**When to package as a Skill:**
- You find yourself copy-pasting the same prompt across sessions
- The capability needs bundled scripts or reference docs
- Multiple agents/teams need the same capability

**Skill anatomy:**
```
my-skill/
├── SKILL.md          # YAML frontmatter (name + description) + instructions
├── scripts/          # Optional executable code
├── references/       # Optional docs loaded as needed
└── assets/           # Optional files used in output
```

The skill ecosystem is now a competitive marketplace: Anthropic Claude Skills, Vercel skills.sh, OpenAI Codex plugins, Cline plugins, MCP server directory. Treat skills as a first-class axis alongside MCP (tools) and A2A (agents).

---

## Step 8 — The 2026 Tooling Stack (always in context)

| Category | Tools |
|----------|-------|
| **Loop orchestration** | LangChain/LangGraph 1.0, Temporal, Inngest, Trigger.dev, Restate |
| **Sandboxed execution** | Modal, E2B, Daytona |
| **Observability** | LangSmith, Langfuse (open-source), MLflow, Arize Phoenix, Helicone, Braintrust |
| **Memory** | Letta (self-editing + sleep-time compute), Mem0 (vectors + KG), Zep (temporal KG via Graphiti), Cognee, Redis LangCache (semantic caching), HippoRAG 2, MemGraphRAG |
| **Vector DBs** | Pinecone, Weaviate, Chroma, pgvector |
| **Agent protocols** | MCP (Model Context Protocol, under AAIF, 2026-07-28 stateless spec), A2A (Agent2Agent, Google/LF) |
| **Agent frameworks** | LangChain/LangGraph 1.0, CrewAI v1.10, Microsoft Agent Framework, OpenAI Agents SDK, Anthropic Claude Agent SDK, Google ADK, Mastra, PydanticAI, Smolagents, DeerFlow 2.0, Vercel AI SDK, LlamaIndex AgentWorkflow, Goose |
| **PromptOps / eval** | Promptfoo, DeepEval, Giskard, RAGAS, Inspect AI, Arthur Bench |
| **Automated optimization** | DSPy, TextGrad, PromptBreeder/EvoPrompt, LLM-as-judge |

---

## Step 9 — Build, Diagnose, or Productionize

Based on the user's request:

### Building from scratch
1. Diagnose which layer they need (Step 0).
2. Read the relevant reference (Step 3).
3. Apply the universal patterns (Step 4).
4. Build using the appropriate template:
   - Prompt layer: Anatomy Template (8 layers) or COSTAR/RISEN framework
   - Context layer: Write/Select/Compress/Isolate operations
   - Harness layer: 6-component harness spec template
   - Loop layer: 8-section loop spec template
5. Run the pre-flight checklist for the layer.

### Diagnosing a broken agent
1. Identify the symptom (Step 0 diagnostic table).
2. Read the relevant reference's Failure Taxonomy / Anti-Patterns section.
3. Apply the matching fix.
4. For production agents, codify the diagnostic as a regression test (PromptOps).

### Productionizing
1. Apply the harness engineering checklist (Step 4).
2. Apply the loop engineering checklist (Step 4).
3. Wire observability (LangSmith/Langfuse).
4. Set cost guardrails (per-iteration/per-loop/per-day).
5. Run the pre-flight checklist for the layer.
6. Apply the Lethal Trifecta security check.
7. For EU deployments, ensure EU AI Act compliance (full enforcement August 2, 2026).

---

## Step 10 — Cross-Layer Worked Example

A user asks: "Build me an agent that fixes failing CI builds every morning."

**Diagnosis:** This is a Layer 4 (loop) problem — scheduled, recurring, multi-iteration, needs verification. But it depends on all lower layers.

**Build sequence:**
1. **Prompt layer:** Write the harness prompt — "You are a senior engineer. Build {build_id} is failing. Failing test: {test}. Open a PR that fixes the root cause. Add a regression test. Do not modify the existing test file." (Claude Opus 4.8 contract-style XML, effort `extra`.)
2. **Context layer:** Inject the failing build log, the source files implicated by the stack trace, the last 10 commits to those files. Use Letta for cross-session memory of similar past failures.
3. **Harness layer:** Tools: `gh issue view`, `gh pr create`, `git`, `npm test`, `rg`, web search — loaded via MCP with Tool Search. Runtime: E2B sandbox with the repo cloned and `npm install` already run. Middleware: hard cap on tool calls, logging to LangSmith, approval gate for security/prod-only code.
4. **Loop layer:** Trigger: cron `0 9 * * 1-5` via Temporal. Verifiable goal: diff non-empty AND failing test passes AND test file unchanged AND CI green AND regression test exists AND independent LLM verifier (GPT-5.6) judges PR description accurate AND root-cause-addressed. Verification: independent model + deterministic checks. Stopping: max 5 iterations, max $2.50/build, 30-min timeout, escalate for security/prod. Cost guardrails: per-iteration $0.85, per-loop $2.50, per-day $30, alert at 80%. Observability: LangSmith, alert on iteration>5/cost>$2.50/verification-failure-rate>30%.

**Full worked example with LangGraph code:** `references/loop-engineering.md` Section 13.

---

## Step 11 — Quick Reference Cards

### The 4-Layer Stack
```
Loop (June 2026)      → triggers, verifies, stops many runs
Harness (Feb 2026)    → equips one agent run (prompt+context+tools+memory+runtime+middleware)
Context (2024-25)     → Write/Select/Compress/Isolate
Prompt (2022)         → role, examples, format, constraints
```

### The 4 Context Operations
- **Write** — offload to external memory (Deep Agents virtual filesystem)
- **Select** — retrieve only relevant (RAG, MCP Tool Search)
- **Compress** — summarize history (Claude Code `/compact`)
- **Isolate** — sub-tasks in sub-agents (Claude Agent Skills)

### The 6 Harness Components
Prompt · Context · Tools · Memory · Runtime · Middleware

### The 7 Loop Topologies
Single-shot · Memory-aware · Scheduled · Event-driven · Nested · Fan-out · Self-feeding

### The 4 Verification Mitigations
Machine-checkable criteria · Independent verifier · Human-in-the-loop · Hard caps

### The 4 Permission Tiers
Read-only · Write-with-rollback · Write-without-rollback · Destructive

### The 3 Cost Cap Layers
Per-iteration · Per-loop · Per-day (alert at 80% of each)

### The 5 Multi-Agent Orchestration Patterns
Sequential pipeline · Router-based delegation · Parallel + aggregation · Hierarchical supervisor · Debate & consensus (plus swarm/dynamic handoff)

---

## Step 12 — Reference File Index

The skill bundles **9 reference files** totaling ~97K words. Read only the reference relevant to the current problem; do not load all nine unless the problem genuinely spans the entire stack.

### The four-layer deep-research briefs (our original research)

| File | Words | When to read |
|------|-------|--------------|
| `references/prompt-engineering.md` | ~9.3K | Vague/wrong-format output; new prompt from scratch; technique catalog; failure taxonomy; PromptOps |
| `references/context-engineering.md` | ~20K | Hallucination; lost-in-the-middle; long sessions; the four operations (Write/Select/Compress/Isolate); RAG; memory systems; caching |
| `references/harness-engineering.md` | ~18.7K | Wrong tools; no exit conditions; fragile single runs; the six harness components; MCP; middleware; multi-agent orchestration |
| `references/loop-engineering.md` | ~19.6K | Scheduled/event-driven agents; the seven topologies; the verification problem; cost guardrails; fleet failures |

### The five inherited reference files (from the source prompt-engineer skill — nothing lost)

| File | Words | When to read |
|------|-------|--------------|
| `references/model-guide.md` | ~14K | Per-model prompting for ALL 2026 models — Claude Opus 4.8, GPT-5.6 Sol/Terra/Luna, Gemini 3.5 Flash+Pro, Grok 4.3, DeepSeek V4, GLM-5.2, Qwen3.7-Max+Plus, MiniMax M3, Kimi K2.6/K2.7, Meta Muse Spark, Mistral Large 3, image models (Midjourney V8.1, gpt-image-2, Nano Banana Pro, FLUX.2, Recraft V4.1, Ideogram 4.0, SD 4 Ultra), video models (Veo 3.1, Gemini Omni, Kling 3.0, Seedance 2.0, Runway Gen-4.5, Grok Imagine 1.5, Wan 2.6, Luma Ray 3.14, Pika 2.2), audio/voice/music models (Eleven v3, Cartesia Sonic 3.5, Hume EVI 3, Sesame CSM, Hedra, Suno 5.5, Lyria 3, Scribe v2), coding agents (Claude Code/Agent SDK, Cursor, Copilot, Cline, Aider, OpenCode, Kilo Code, Gemini CLI), MCP under AAIF, Context/Harness Engineering |
| `references/techniques.md` | ~13K | Deep-dive on each of the 31 techniques — Zero-Shot, Few-Shot, CoT, ToT, ReAct, Self-Consistency, Meta-Prompting, Prompt Chaining, SoT, Role/Persona, Prompt Compression, RAG Prompting, Calibrated Confidence, PAL/PoT, Multimodal, COSTAR, RISEN, CoV, Multi-Agent Debate, Context Engineering, Harness Engineering, Agent Skills, MCP Tool Search, A2A Protocol, Subagent Orchestration, DSPy, Anti-Slop, Personalization-Driven, Voice/Music Stage-Direction, Loop Engineering. Read this when you need the full deep-dive on a specific technique (history, when-to-use, when-not-to-use, examples, variants). |
| `references/security.md` | ~7.5K | Prompt injection defense; OWASP LLM Top 10 (2025); OWASP ASI Top 10 for Agentic (2026); Lethal Trifecta / Rule of Two; MCP attack classes (tool poisoning, rug pulls, confidential data exfiltration); EU AI Act compliance |
| `references/templates-library.md` | ~10K | 30+ ready-to-use prompt templates by industry/use-case — copy-paste starting points for customer support, coding agents, research, content creation, data extraction, RAG, and more |
| `references/update-methodology.md` | ~5K | How this skill is kept current (last updated end-of-June 2026); the editorial process; what drifts and what doesn't |

Total reference material: ~97K words across the nine briefs.

---

## Step 13 — Quick Examples (reproduced from source)

Six ready-to-paste examples covering the most common patterns. Reproduced from the source prompt-engineer skill so nothing is lost.

### Example A: Vague → COSTAR-Precise

**Before:** `Write a blog post about AI.`

**After (COSTAR):**
```
Context: AI agents are increasingly replacing manual knowledge work in 2025/26.
Objective: Convince non-technical executives that AI agents will reshape middle management.
Style: Opinion piece — punchy paragraphs, no bullet lists, 2–3 real company examples.
Tone: Authoritative but accessible — Harvard Business Review register.
Audience: C-suite readers with no technical background.
Response: 600 words. Headline + subheadline + 3 paragraphs + one-sentence CTA. Do not use "delve".
```

### Example B: Hallucination fix (Context Engineering)

**Problem:** "Tell me about our Q3 revenue" → Model makes up numbers

**Fix:**
```xml
<role>Financial analyst assistant. Be honest — say "not in the data" rather than estimating.</role>
<context>
  [paste Q3 report here]
</context>
<task>
  Answer: What was Q3 revenue and how does it compare to Q2?
</task>
<constraints>
  - Use ONLY numbers from the provided report
  - If a figure is not present, say "This information is not in the provided report"
  - Do not estimate, interpolate, or infer
  - Flag any uncertainties in your answer
</constraints>
```

### Example C: Reasoning model prompt (Claude Opus 4.8 ET / GPT-5.6 max reasoning / Gemini 3.5 Deep Think)

**Wrong:** `Think step by step: Is it better to invest in index funds or individual stocks for a 32-year-old with $10,000?`

**Right:**
```
A 32-year-old with $10,000 to invest asks whether index funds or individual stocks are better
for a 20-year horizon. Consider: risk tolerance implied by age/timeline, diversification
trade-offs, empirical data on average investor performance in each approach, and fee structures.

Provide: A clear recommendation (1 sentence) followed by a 200-word rationale.
```
Set `thinking: {type: "enabled", budget_tokens: 8000, effort: "extra"}` (Claude) or `reasoning: {effort: "max"}` (GPT-5.6) via the API parameter — do NOT put CoT instructions in the prompt itself.

### Example D: RISEN for an agentic workflow

```
Role: You are a senior customer support agent with full access to the order database.
Instructions: Investigate and resolve the customer's issue described below.
Steps:
  1. Look up order ID in the provided database context
  2. Identify the root cause (shipping delay, wrong item, billing error)
  3. Draft a resolution response to the customer
  4. Flag if escalation to a human agent is needed (criteria: refund > $200, legal mention)
End goal: Customer issue resolved or correctly escalated within this single response.
Narrowing: Respond only in English. Keep customer-facing message under 150 words. Do not
  promise delivery dates you cannot verify from the data.

<order_context>
  [inject customer order data here]
</order_context>
```

### Example E: Context Engineering for multi-turn agent

**Pattern — what to pass at each step:**
```
System: [persistent role + rules]
Context window at step N:
  - Conversation history (last 5 turns, compressed)
  - Current task state: {step: 3, completed: ["research", "outline"], remaining: ["draft", "review"]}
  - Retrieved documents: [most relevant chunks from vector store]
  - Tool results from previous step: [search results / API output]
  - Current instruction: [the specific action for this step only]
```

### Example F: Harness Engineering — full agent spec (2026)

```
## Goal
Resolve customer billing dispute and issue refund if warranted.

## Tools Available (via MCP servers)
- billing.lookup(order_id) → returns invoice + payment history
- billing.issue_refund(order_id, amount, reason) → executes refund
- crm.log_interaction(customer_id, summary) → logs to CRM
- escalation.notify(team, ticket) → routes to human agent

## Workflow
1. Look up order via billing.lookup
2. Classify dispute: duplicate charge | wrong amount | service not rendered | other
3. If duplicate or wrong amount: issue refund via billing.issue_refund
4. Log outcome to CRM
5. If dispute type is "other" or refund > $500: escalate

## Decision Rules
- If billing.lookup returns no record → tell customer "no order found" and stop
- If refund amount > $500 → escalate, do not auto-issue
- If dispute mentions legal action → escalate immediately

## Error Handling
- If billing.issue_refund fails → retry once; if still failing, escalate
- If MCP server is down → return "system temporarily unavailable" and stop

## Stopping Conditions
- Customer issue resolved with confirmation OR escalated to human agent with ticket ID

## Memory
- Use Letta for cross-session memory (customer history, prior disputes)
- Use Redis LangCache for semantic caching of similar disputes (cost reduction)

## Safety (Lethal Trifecta check)
- Agent has access to private data (customer info) ✓
- Agent has tools that can exfiltrate (logging, notifications) ✓
- Agent receives untrusted content (customer message) ✓
 All three present — apply strict instruction/data separation, allow-listed tool
   schemas, and human-in-the-loop approval for refunds > $200.
```

---

## When NOT to Use This Skill

- **Trivial one-shot questions** ("what's the capital of France?") — no skill needed; just answer.
- **Non-LLM software engineering** — this skill is LLM-specific.

For everything else — any prompt, context, harness, loop, agent, MCP, reasoning model, image/video/audio generation, multi-agent orchestration, or PromptOps question — this skill applies.

---

## Sources & Further Reading

Each reference file has a comprehensive Primary Sources section. The foundational sources:

- **Prompt:** OpenAI Prompt Engineering Guide (2022), Anthropic Prompt Design (2023), CoT/ReAct/ToT/CoV papers.
- **Context:** Anthropic, "Effective Context Engineering for AI Agents" (Sept 2025) — canonical four operations.
- **Harness:** Karpathy / Hashimoto (Feb 2026) — coinage; Cobus Greyling (HarnessX, Hill Climbing).
- **Loop:** Addy Osmani, "Loop Engineering" (June 7, 2026); LangChain, "The Art of Loop Engineering"; Cobus Greyling; Adnan Masood (June 24, 2026).

**Key statistic:** Fiddler AI (2026) reports 70–95% of AI agents fail in production. The bottleneck is the surrounding system, not the model. This skill is the comprehensive reference for fixing the surrounding system.

---

## Changelog

- **July 2026 (initial):** Skill assembled from four deep-research briefs covering prompt, context, harness, and loop engineering. Reflects the late-June 2026 model landscape (Claude Opus 4.8, GPT-5.6, Gemini 3.5, GLM-5.2, DeepSeek V4, Grok 4.3, Qwen3.7-Max, Meta Muse Spark). The MCP 2026-07-28 stateless spec finalizes July 28, 2026. EU AI Act full enforcement for GPAI providers begins August 2, 2026. Re-verify model names, pricing, and capability claims against current vendor docs before production deployment.
