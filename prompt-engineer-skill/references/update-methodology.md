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
