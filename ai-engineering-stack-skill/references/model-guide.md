# Model-Specific Prompting Guide (End-of-June 2026 Edition)

## Table of Contents
1. [Claude Opus 4.8 (Anthropic)](#claude-48)
2. [Claude Fable 5 / Mythos 5 — suspended](#claude-mythos)
3. [Claude 4.7 → 4.8 Migration Guide](#claude-migration)
4. [GPT-5.6 Sol / Terra / Luna (OpenAI)](#openai)
5. [Google Gemini 3.5 Flash + 3.5 Pro (announced)](#gemini)
6. [Grok 4.3 + Grok Build 0.1 (xAI / SpaceXAI)](#grok)
7. [DeepSeek V4 (Pro / Flash)](#deepseek)
8. [GLM-5.2 (Z.ai / Zhipu)](#glm)
9. [Qwen3.7-Max + Qwen 3.7 Plus (Alibaba)](#qwen)
10. [MiniMax M3 + Kimi K2.6 / K2.7 (other Chinese open-weights)](#chinese-ow)
11. [Meta Muse Spark (replaces Llama)](#meta)
12. [Mistral Large 3 + open-weights landscape](#mistral)
13. [Image Generation (Midjourney V8.1, gpt-image-2, Nano Banana Pro, FLUX.2, Recraft V4.1, Ideogram 4.0, SD 4 Ultra)](#image-gen)
14. [Video Generation (Veo 3.1, Gemini Omni, Kling 3.0, Seedance 2.0, Runway Gen-4.5, Grok Imagine 1.5, Wan 2.6, Luma Ray 3.14, Pika 2.2)](#video-gen)
15. [Audio / Voice / Music Generation (Eleven v3, Cartesia Sonic 3.5, Hume EVI 3, Sesame CSM, Hedra, Suno 5.5, Lyria 3, Scribe v2)](#audio-gen)
16. [Coding Agents (Claude Code/Agent SDK, Cursor, Copilot, Cline, Aider, OpenCode, Kilo Code, Gemini CLI)](#coding-agents)
17. [Reasoning Models — Critical Differences](#reasoning)
18. [MCP (Model Context Protocol) Prompting under AAIF](#mcp)
19. [Context Engineering, Harness Engineering & Loop Engineering for Production Systems](#context-eng)

---

## Claude Opus 4.8 (Anthropic) {#claude-48}

**Release:** May 28, 2026 | **Context window:** 1M tokens (default; 200K on Microsoft Foundry) | **Max output:** 128K tokens
**Pricing:** $5 / M input, $25 / M output (unchanged from 4.7). **Fast mode:** $10 / M input, $50 / M output — 2.5× speed, 3× cheaper than fast mode on prior models.
**Knowledge cutoff:** January 2026 (carried from 4.7).

### The "Literal + Anti-Sycophancy" Shift — MOST IMPORTANT CHANGE

> **Opus 4.8 takes instructions literally AND actively flags uncertainty.** It is ~4× less
> likely than 4.7 to let flaws in its own code pass unremarked; more likely to flag
> uncertainties; "less likely to make unsupported claims." Lowest misaligned-behavior rate
> since Claude Mythos Preview.

**DO:**
- Write precise, contract-style instructions — every constraint is followed exactly
- Use XML structure (Claude parses XML natively)
- For long documents: **source material at the TOP, question/instructions at the BOTTOM**
- Set `thinking: {type: "enabled", budget_tokens: N}` for hard problems; choose effort level
- Let adaptive thinking handle multi-step reasoning internally
- Be direct: "Do X. Do not do Y. Output format: Z."
- Explicitly invite the model to flag uncertainties — this aligns with its training

**DON'T:**
- Add "Be honest" or sycophancy coaxing (resolved in 4.7, doubled down in 4.8)
- Over-explain or pad with context — 4.8 pays precise attention
- Use explicit prompt chaining unless you need to inspect intermediate outputs
- Treat it like 4.7 — re-tune your prompts and harnesses

### Key Differences from Opus 4.7

| Dimension | 4.7 | 4.8 |
|-----------|-----|-----|
| Effort control | Implicit (budget_tokens) | Explicit dial: low / high (default) / extra / max |
| Context window | 200K | 1M (default) |
| Sycophancy | Largely resolved | Anti-sycophancy is a training goal (~4× less likely to let code flaws pass) |
| Patch size | Smaller than 4.6, more precise | Fixes comment-verbosity regression of 4.7 |
| Multi-step reasoning | Adaptive thinking | Adaptive thinking + **dynamic workflows** (research preview, hundreds of parallel subagents) |
| Tool calling | Good | Fixes tool-calling regression of 4.7 |
| System instructions | Top-of-prompt only | Can now appear inside `messages` array (update mid-task without breaking prompt cache) |
| Min cacheable prompt length | Higher | 1,024 tokens |

### Effort Control Dial

```json
{
  "thinking": {
    "type": "enabled",
    "budget_tokens": 10000
  }
}
```
Effort levels (conceptual):
- **low** — quick answers, simple lookups
- **high** (default) — most production work
- **extra** — hard problems, async long-running workflows
- **max** (aka `xhigh` in Claude Code) — most difficult tasks, deep analysis

Anthropic recommends **"extra"** for difficult tasks and long-running async workflows.

### Dynamic Workflows (Research Preview)

In Claude Code, Opus 4.8 can spawn **hundreds of parallel subagents** in one session,
verify outputs, and aggregate results — enabling codebase-scale migrations in a single
agent run. This is the production realization of the "subagent orchestration" pattern.

### Contract-Style Template

```xml
<role>You are a [specific expert]. Available tools: [list / Agent Skills / MCP servers].</role>
<context>[Background information, data, documents — source material at top]</context>
<task>Do [specific action] with [specific input].</task>
<constraints>
  - Do NOT [behavior to avoid]
  - Stay under [N] words
  - Only use provided data — do not infer
  - Flag any uncertainties in your answer
</constraints>
<output_format>[JSON schema / table / structure]</output_format>
<examples>
  Input: [example input]
  Output: [example output]
</examples>
```

### Reusable Prompts → Agent Skills

Anthropic's 2026 guidance: convert reused prompts into an **Agent Skill** (modular,
discoverable capability package). Supported across Claude.ai, Claude Code, Claude Agent
SDK, and Claude Developer Platform. Public skills repo: https://github.com/anthropics/skills

---

## Claude Fable 5 / Mythos 5 — SUSPENDED {#claude-mythos}

**Release:** June 9, 2026 (GA) | **Status:** Access **suspended worldwide June 12, 2026** per US Commerce Department export-control directive (reported jailbreak). Public availability lasted ~72 hours.

**Position:** New **"Mythos-class" tier above Opus** (Haiku → Sonnet → Opus → Mythos). Fable 5 = public-facing config of Mythos 5 (same underlying model, different safeguards).
**Context / output:** 1M context, 128K max output.
**Pricing:** $10 / M input, $50 / M output (2× Opus 4.8).

**Prompting note:** Same general Claude guidance applies; designed for the hardest cyber/research work. Currently **not accessible** — Anthropic says it is developing stronger cyber safeguards and hopes to restore Mythos-class access "in the coming weeks."

**Mythos 5 partial re-release:** On June 26, 2026, the US government granted Anthropic permission to release Mythos 5 to ~100 companies and federal agencies with a 30-day data retention policy for safety monitoring. As of late June 2026, **Opus 4.8 remains the reliable production target** for most users.

### Other Claude tiers (for completeness)
- **Claude Sonnet 4.6** — released Feb 2026 (balanced tier; default for free/Pro).
- **Claude Haiku 4.5** — released Oct 2025 (fast tier).
- **Claude Sonnet 4 & Opus 4** — retired **June 15, 2026** (migrate to 4.5/4.6/4.7/4.8).
- **No Sonnet 4.8** has shipped (leaked codenames referenced it, but only Opus 4.8 actually released).

---

## Claude 4.7 → 4.8 Migration Guide {#claude-migration}

If migrating existing prompts from 4.7 to 4.8:

1. **Remove** `"Be honest"` and sycophancy guards (already done in 4.7, but explicitly lean into 4.8's anti-sycophancy)
2. **Tighten** constraints — 4.8 will follow them literally
3. **Simplify** role descriptions — no need to over-credential
4. **Enable** `thinking` with appropriate budget for complex tasks; choose effort level explicitly
5. **Test** on edge cases — literal following means it may refuse ambiguous instructions
6. **Reorder** long-document prompts: source material at top, question/instructions at bottom
7. **Move reused prompts into Agent Skills** rather than copy-pasting
8. **Use `messages` array for system instructions** if you need to update mid-task
9. **Consider dynamic workflows** for codebase-scale migrations
10. **Lower minimum cacheable prompt length (1,024 tokens)** means smaller prompts benefit from caching

---

## GPT-5.6 Sol / Terra / Luna (OpenAI) {#openai}

**Release:** June 26, 2026 (limited preview only — government-coordinated)
**Context window:** ~1.5M tokens for Sol per multiple third-party reports (OpenAI's preview post does not state it explicitly; treat as near-confirmed but not vendor-confirmed).
**Pricing (per 1M tokens):**
- **Sol** (flagship): $5 input / $30 output
- **Terra** (balanced, ~2× cheaper than GPT-5.5): $2.50 input / $15 output
- **Luna** (fastest / lowest cost): $1 input / $6 output

**Naming change:** The **number = generation**; **Sol / Terra / Luna = durable capability tiers** that advance on their own cadence.

### Notable New Features

- **"Max reasoning effort"** mode — most time to reason deeply.
- **"Ultra mode"** — goes beyond a single agent, leverages **subagents** to accelerate complex work.
- **Predictable prompt caching:** explicit cache breakpoints, **30-minute minimum cache life**; cache writes billed at 1.25× uncached input rate; cache reads get 90% discount.
- **Sol on Cerebras** at up to 750 tokens/sec starting July 2026.

**Access:** Government-coordinated limited preview — only a small group of trusted partners whose participation was shared with the US government. Broad ChatGPT/Codex/API availability "in the coming weeks." OpenAI explicitly says this should *not* become the long-term default.

**Behavioral / safety:** Strongest cyber model yet; trained to refuse prohibited cyber assistance **even when users disguise intent or jailbreak**; real-time cyber/biology misuse classifiers can pause generation for a larger reasoning model to review.

### Prompting Best Practices

- Clean markdown headers — GPT-5.6 responds well to structure
- State the task FIRST: `## Task` before background context
- Use `system` + `user` message split for persona and task
- For JSON: `response_format: json_object` or `json_schema` via API
- Use **explicit cache breakpoints** for long-running agent sessions
- Use **`max reasoning effort`** for hardest tasks
- Use **`ultra mode`** for subagent-accelerated complex work
- Use `strict: true` on function/tool definitions for guaranteed JSON Schema conformance
- Computer use: to enable the agent to interact with browser/GUI

**Reasoning effort (API parameter, not language):**
```json
{
  "reasoning": {
    "effort": "max"
  }
}
```

**GPT-5.6 Template:**
```markdown
## Role
You are a [specific expert]. [Persistent rules].

## Task
[Action verb + goal — one sentence]

## Data / Context
[background, documents, constraints]

## Output format
[exact structure]

## Examples
- Input 1 → Output 1
- Input 2 → Output 2
```

**Model lineage:** GPT-4.1 → GPT-4o (multimodal speed) → GPT-5.4 → GPT-5.5 (April 2026, frontier) → **GPT-5.6** (June 2026, limited preview). **GPT-4.5 was removed from ChatGPT on June 26, 2026.** GPT-5.5 remains the current ChatGPT model.

---

## Google Gemini 3.5 Flash + 3.5 Pro (announced) {#gemini}

### Current latest Pro: Gemini 3.1 Pro (still the newest available Pro)
**Release:** April 2, 2026 | **Context window:** 1M tokens standard; **2M available** (long-context tier).
**Pricing:** $2 / M input, $12 / M output (≤200K context); $4 / M input, $18 / M output (>200K). Free tier available. Context caching supported.
**Traits:** "Reasoning-first," **adaptive thinking** (medium/low/high thinking modes), native multimodal, strong long-context retrieval (MRCR strong at 500K+).

### Latest Gemini overall: Gemini 3.5 Flash (GA)
**Release:** May 19, 2026 (GA at Google I/O). Now backs `gemini-flash-latest`.
**Position:** "Most intelligent model for sustained frontier performance on agentic and coding tasks." Same text/image pricing as 3.1 Pro per docs.
**Computer Use** tool launched in public preview on 3.5 Flash on **June 24, 2026** (browser/mobile/desktop, prompt-injection detection).

### Gemini 3.5 Pro — announced but NOT yet released (slipped to July 2026)
**Announced:** Google I/O, May 19, 2026 ("rolling out next month").
**Status as of June 28, 2026:** **Not released.** Business Insider (Jun 24, 2026) reports the date **slipped to July 2026**. DeepMind's own model page still says **"3.5 Pro coming soon."** Internally in use.
**Targeted specs:** **2M-token context window**, new **"Deep Think"** reasoning mode, frontier multimodal.

### Other Google items
- **Gemini Omni** — announced at I/O 2026; blends text/audio for dynamic video content. First model **Gemini Omni Flash** rolled out at I/O. Unified multimodal: takes any reference (image/text/video/audio) → video. More accurate physics; conversational/voice editing; SynthID watermark.
- **Gemma 4** — open models built from Gemini 3 research (April 2, 2026).
- **Managed Agents / Antigravity Agent** — public preview May 19, 2026.
- **No Gemini 4.0 exists yet.**

### Key Prompting Techniques

- Always add `"Be concise."` — Gemini still tends toward verbosity
- Use `context_caching` API for repeating content across calls (cost optimization)
- Embed images, PDFs, audio, video natively — best multimodal support
- List formatting preferences as bullet points
- Gemini excels at **multi-agent parallel reasoning** — prompt multiple agents in parallel, synthesize results
- For Deep Think: state problem + all context; no CoT scaffolding
- 2M token context ideal for: entire codebases, legal document sets, full research paper batches
- Adaptive thinking budget is the main lever; long-context retrieval remains strong, so RAG is optional for many workloads under 2M

**Gemini Template:**
```
System: You are a [role] with [expertise]. Be concise.

Background: [context]
Task: [action verb + goal]

Formatting:
- [bullet 1]
- [bullet 2]

Output: [structure]
Constraints: [limits]
```

---

## Grok 4.3 + Grok Build 0.1 (xAI / SpaceXAI) {#grok}

> **Correction:** Grok 5 has **NOT** been released. It is still in training as of June 2026 (Q1 2026 target slipped; confirmed in xAI's Jan 28, 2026 Series E update). Treat any "Grok 5" claims as rumor.

### Current flagship: Grok 4.3
**Release:** April 30, 2026 | **Context window:** 1M tokens
**Pricing:** $1.25 / M input, $2.50 / M output
**Traits:** Agentic tool calling, "minimal hallucinations," **configurable reasoning + a non-reasoning mode**. Knowledge cutoff **November 2024** (Grok 3 & 4).

**Prompting notes:**
- Models `grok-4.20` and newer **silently ignore `logprobs`/`top_logprobs`**
- No role-order limitation (system/user/assistant can be mixed freely)
- Real-time data requires enabling Web Search / X Search tools
- Prompt similarly to GPT (markdown structure, system/user split)
- Strong creative writing — give detailed creative direction
- Use for opinions and cultural commentary where recency matters

### Grok Build 0.1 (coding)
**Release:** June 1, 2026 (API public beta); Grok Build agent launched May 27, 2026.
**Context:** 256K tokens. **Pricing:** $1 / M input, $2 / M output.
**Recent additions:** Composer 2.5 (June 3, 2026), `/goal` for long-running autonomous tasks (June 22, 2026).

### Other Grok / xAI items
- **Grok 4.20** (March 31, 2026): 2M context, reasoning / non-reasoning / multi-agent variants.
- **Grok 4 Fast:** 2M context, ~98% cheaper than flagship.
- **xAI acquired by SpaceX** (April 17, 2026) — now branded "SpaceXAI."
- **xAI ↔ Anthropic compute partnership** (May 15, 2026): Anthropic gets access to xAI's **Colossus 1** cluster.
- **Grok Imagine 1.5** (`grok-imagine-video-1.5-preview`) — image-to-video at up to 720p; Aurora image model; native audio; "Spicy Mode" (less-censored).

---

## DeepSeek V4 (Pro / Flash) {#deepseek}

> **Correction:** DeepSeek R2 has **NOT** been released. Timing never set; treat as rumor. The latest DeepSeek is **V4**.

**Release:** April 24, 2026 (live & open-sourced)
**Models:** Two MoE models — **DeepSeek-V4-Pro** (~1.6T total params) and **DeepSeek-V4-Flash**. ~1M context, 384K max output (Pro).
**Pricing (Pro):** **$0.14 / M input, $0.28 / M output** — among the cheapest frontier-tier pricing available.
**Notable:** **"Maximum reasoning effort mode,"** Huawei Ascend optimization, claims to beat all open models.

### Key Techniques

- Externalizes reasoning — you can see the `<think>` block (R1-style chain-of-thought visible)
- Excellent for: math proofs, algorithm design, scientific analysis
- Very cost-competitive — use for high-volume technical tasks
- Use **`maximum reasoning effort mode`** for hardest problems
- Prompt clearly and directly; works well with ChatGPT system/user split
- Open-weights — can self-host for sensitive workloads

---

## GLM-5.2 (Z.ai / Zhipu) {#glm}

**Release:** June 13, 2026 (to GLM Coding Plan subscribers; MIT-licensed open weights)
**Specs:** **753B MoE (40B active), 1M-token context, coding-first/agent-first.**
**Independent reporting:** "Beats GPT-5.5 at ~1/6 the cost."
**Notable:** Usable inside **Claude Code** (set model name to `GLM-5.2` for 1M context). Coding Plan entry ~$10/month.

**Lineage:**
- **GLM-5** — released Feb 11–12, 2026 (just before Lunar New Year); ARC = Agentic + Reasoning + Coding in a Model-of-Experts.
- **GLM-5.1** — incremental flagship.
- **GLM-5.2** — June 13, 2026 (current).

**Pricing (RMB, bigmodel.cn):** GLM-5.1 input 8元 / output 28元 per M tokens; GLM-5-Turbo 5元/22元 — both "limited-time free."

### Prompting
- GLM has native **"Agent Mode"** — built for agentic workloads
- Works well with ChatGPT-style system/user split
- Coding-first: works well with coding agent patterns (CLAUDE.md-style, .cursorrules-style)
- 1M context — can ingest entire codebases or document sets
- Strong Chinese-language support

---

## Qwen3.7-Max + Qwen 3.7 Plus (Alibaba) {#qwen}

**Qwen3.7-Max release:** May 19, 2026 (proprietary, not open-weights)
**Specs:** **1M context, 65,536 max output.** "The Agent Frontier" — built for the agent era; claimed 35-hour autonomous operation.
**API compatibility:** OpenAI- and Anthropic-API-compatible (drop-in replacement).

**Qwen 3.7 Plus release:** June 1, 2026 (GA) — low-cost multimodal agent model with vision + video understanding. Bolts multimodal perception onto the agent era chassis at a lower price point than Max.

**Lineage:**
- **Qwen3-Max-Thinking** — released Jan 25, 2026 (flagship reasoning).
- **Qwen3-Next** — new ultra-efficient architecture for long-context.
- **Qwen3.7-Max** — May 19, 2026 (flagship).
- **Qwen 3.7 Plus** — June 1, 2026 (low-cost multimodal agent tier).

### Prompting
- Drop-in compatible with OpenAI/Anthropic API patterns (both Max and Plus)
- Built for long-running autonomous agent workflows
- Strong Chinese + English bilingual support
- Use Anthropic-style contract prompts or OpenAI-style system/user split
- For Qwen 3.7 Plus: leverage native vision/video understanding for multimodal agent tasks

---

## MiniMax M3 + Kimi K2.6 / K2.7 (other Chinese open-weights) {#chinese-ow}

### MiniMax M3
**Release:** Late June 2026 ("officially released today" per minimax.io; open weights ~10 days after launch).
**Specs:** **428B MoE, 1M context, native multimodality.**
**Position:** Marketed as the **first open-weights model to combine frontier coding + 1M context + native multimodality**; claims to match GPT-5.5 on coding at ~5% of cost.
**Availability:** Fireworks AI / NVIDIA Blackwell.

### Kimi K2.6 / K2.7 (Moonshot AI)
- **Kimi K2.6** — released April 20, 2026, open-source, natively multimodal, coding-focused, **Agent Swarm** (scales to 300 sub-agents / 4,000 coordinated steps).
- **Kimi K2.7 Code** — released June 2026 (latest Kimi).
- **Kimi K3** — in development; Moonshot raised $500M Series C to fund it. Not released.

### Best open-source coding models (mid-2026)
GLM-5.1/5.2, MiniMax M3, Kimi K2.6, DeepSeek V4-Pro/Flash, Qwen3-Next.

---

## Meta Muse Spark (replaces Llama) {#meta}

> **Major strategic shift:** Meta has pivoted **away from open-weights Llama** to a new proprietary "Muse" series. **Llama 5 has NOT been released**, and may not be for the foreseeable future.

### Muse Spark (Meta's new flagship — proprietary)
**Release:** April 9, 2026 (Meta Superintelligence Labs)
**Position:** First in a **new "Muse" series** that **replaces Llama** as Meta's flagship line. **Proprietary** (not open-weights) — a sharp strategic reversal from Llama.
**Architecture:** Natively multimodal reasoning model. Largest config = **dense Transformer, 405B parameters**, context window reported as ~128K (one source) to 260K (Artificial Analysis). Tool-use, **visual chain-of-thought**, **multi-agent orchestration**.
**Claim:** Reaches Llama 4 Maverick-level capability at **>10× less compute**. Now powers Meta AI on most smart glasses (replacing Llama 4).

**Prompting note:** Reasoning-first, multimodal — design prompts around explicit step-by-step reasoning and tool-use; behaves more like Gemini/GPT than like Llama. Treat like Gemini/GPT for prompting style.

### Llama 4 (still the latest open-weight Llama)
- **Llama 4 Scout** (17B, 16 experts) and **Llama 4 Maverick** (17B MoE) — released April 5, 2025. Scout supports up to 10M context. These remain the newest open-weight Llama models.
- **Llama 4 Behemoth** (~2T params): **shelved / never publicly released.**
- **Llama 5:** No official announcement; prediction markets estimate late 2026 at earliest.

**Access (for legacy Llama 4):** Ollama, Together AI, Groq, Replicate, Hugging Face.
**Techniques:** Chat template with header tokens handled automatically; explicit role + task separation improves instruction-following; function calling available in 70B+ variants.

---

## Mistral Large 3 + open-weights landscape {#mistral}

- **Mistral Large 3** — released Dec 2, 2025 (flagship open-weight MoE, 675B total / 41B active, trained on 3,000 H200s). Still the Mistral flagship in mid-2026.
- **Mistral Medium 3.5** (128B open-weight, agent-oriented) is the newest mid-tier.
- A new family of open-weight models is **rumored for July 2026** (post-cutoff).
- **Yi (01.AI)** — no major frontier release in 2026; the line is effectively quiet.

---

## Image Generation (Midjourney V8.1, gpt-image-2, Nano Banana Pro, FLUX.2, Recraft V4.1, Ideogram 4.0, SD 4 Ultra) {#image-gen}

### Midjourney V8 / V8.1 (current default)
**Timeline:** V8 Alpha March 17, 2026 → V8.1 Alpha April 14, 2026 → released on midjourney.com April 30, 2026 → **default on June 10, 2026**.

**The big idea:** V8 shifts creative control **from your text prompt to your personalization profile**. The prompt provides *subject/context*; your rated-aesthetic profile provides *look*.

**Prompting:**
- **Longer prompts now work** (reverses V7's "concise is better"). Midjourney recommends "trend towards longer, more specific prompting." A 15-word V7 prompt often works better at ~40 words in V8.
- **Suggested structure:** `[Subject and action], [environment and context], [specific details to preserve], [lighting and atmosphere] --s 1000 --p --ar 16:9`
- Front-loaded weighting still applies (early words carry more influence).
- Complex `--no` negative chains can often be replaced with plain English ("an interior scene with no visible windows").
- **Conversation Mode:** describe in natural language/voice; an AI writes the actual prompt. Good for exploration; bypass for execution.
- **Prompt Shortener** auto-engages when over the length limit (condenses rather than truncates).
- **`Describe`** rewritten to produce longer, V8-style prompts.
- **Style Creator:** generates custom `--sref` codes via visual preference selection.

**Key parameters:**
- `--stylize` / `--s` — push to **1000** with a trained profile
- `--sref` / `--sw` — style reference + style weight; now "super stable"
- `--p` — personalization profile (activates at 40 ratings, stabilizes ~200, improves to ~2,000)
- `--raw` — strips styling for photoreal/editorial; combine with high `--stylize`
- `--ar` — aspect ratio
- HD mode: **default** in V8.1, 3× faster and 3× cheaper than V8.0, native 2K
- **Removed/unsupported in V8.1:** `--q 4` (V7-era), `--cref`, `--oref`. Use Personalization + srefs for character/object consistency instead.
- **Image prompts & image weights are back** (V8.0 had broken them).

**Quirks:** If you prompt V8 like V7 you're "working against the tool." Without ≥200 ratings, V8 runs at a fraction of capability.

### OpenAI gpt-image-2
**Lineage:** Native image gen arrived in **GPT-4o** (2025) as `gpt-image-1` → `gpt-image-1.5` → **`gpt-image-2`**, launched as **ChatGPT Images 2.0 on April 21, 2026**. **GPT-5 native image generation** is being tested.

**What it's best at:** "Designed assets" — readable text, clean layouts, product compositions, UI mockups, infographics, posters. Strong instruction-following and multi-turn conversational refinement.

**5-slot prompt template (community-validated):**
```
Scene:        [where, time of day, background, environment]
Subject:      [who/what is the main focus]
Important details: [materials, clothing, texture, lighting, camera angle, lens feel, composition, mood]
Use case:     [editorial photo / product mockup / poster / UI screen / infographic / concept frame]
Constraints:  [no watermark / no logos / no extra text / preserve specific elements]
```
Mnemonic: **PLACE · FOCUS · FACTS · FORM · CONSTRAINTS.**

**Anti-slop rules (the single most impactful insight):** Replace vague aesthetic words with concrete visual facts.
| Don't say | Say instead |
|---|---|
| "stunning" | "overcast daylight, shallow depth of field" |
| "epic" | "low-angle shot, wide 24mm lens" |
| "cinematic" | "anamorphic 2.39:1, teal-and-orange grade, lens flare" |
| "realistic" | "shot on Canon R5, 85mm f/1.4, natural window light" |
| "high quality" | "8K texture detail, film grain ISO 400" |

**Other techniques:** Thinking mode, edit chains (iterative conversational edits), image/reference inputs (character ref + background ref).

**Negative prompting:** No traditional negative-prompt field; use the **Constraints** slot / natural language ("no extra text").

### Google — two image families
> **⚠️ Imagen 4 deprecation (announced June 2026):** Google is shutting down Imagen 4
> standard / ultra / fast endpoints on **August 17, 2026**. Vertex AI customers must
> update endpoints before **June 30, 2026** for some workflows. **Migrate to Nano Banana
> Pro (Gemini 3 Pro Image)** — Google's official replacement path.

**(a) Imagen 4 family (DEPRECATED — shut down August 17, 2026):** Imagen 4, **Imagen 4
Ultra** (was highest-fidelity photorealistic of early-2026), and **Imagen 4 Fast** (~$0.02/image,
~2.7s latency). Up to 2K resolution. Strong text rendering. **All being replaced by Nano
Banana Pro — do not start new projects on Imagen 4.**

**(b) Nano Banana (Gemini-native) — Google's current image family:**
- **Nano Banana Pro** = **Gemini 3 Pro Image** — image gen + editing built on Gemini 3;
  conversational, advanced world knowledge, subject consistency, improved reasoning over
  lighting. **Best for: complex graphic design, high-fidelity product mockups, factual
  data visualizations requiring accurate text rendering.**
- **Nano Banana 2** — combines Pro capabilities with Flash speed (production-ready,
  subject consistency). This is the consumer-facing "Gemini image generator & photo editor."

**Prompting (Nano Banana Pro / Gemini 3 Pro Image):**
- Conversational multi-turn editing is the primary interaction model — start broad, refine.
- Supports **negative prompts** as natural language constraints in the prompt.
- **Aspect ratios:** 1:1, 3:2, 2:3, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9.
- **CLEAR prompts framework** recommended (Context, Lighting, Emotion, Action, Region/Rate).
- Subject consistency across multiple images via reference-image input (not via prompt alone).

### FLUX.2 (Black Forest Labs)
**Current family:**
- **FLUX.2 [max]** — top-tier quality/photorealism, hero shots (flagship).
- **FLUX 2 Pro** — 32B params (released Nov 2025), up to **4 megapixels**, accurate text rendering, precise color matching; now ~2× faster, no quality loss, no price change.
- **FLUX 2 Flex** — more sampler/step controls for tuned looks.
- **FLUX 2 Dev** — dev/open variant.
- **FLUX.2 [klein]** — released **January 2026**, fastest variant: **sub-second** generation on modern GPUs, ~13GB VRAM consumer-friendly; 4B variant under **Apache 2.0**.

**Prompting:** High prompt adherence, strong text rendering, image-editing capabilities. Natural-language prompting works well. Known for following detailed prompts faithfully (more literal than Midjourney's profile-driven approach).

**Negative prompting:** FLUX.1/2 don't have a native strong negative-prompt weight like SD; control via prompt phrasing and guidance settings.

### Stable Diffusion 4 Ultra (Stability AI, 2026)
**Status:** Stability AI released **Stable Diffusion 4 Ultra** in 2026 — a major
architectural overhaul built on an upgraded **diffusion transformer (DiT)** focused on
photorealism. ⚠️ Stability has publicly pivoted toward filmmaking applications; the
long-term roadmap for the SD line is uncertain. Treat SD 4 Ultra as "released but
vendor-pivot-risk."

**Reality on the ground (2026):** The open-source community still runs heavily on
**SDXL + fine-tunes/LoRAs**, **SD3.5**, and **FLUX** models — SD 4 Ultra has not yet
displaced these in the LoRA/ControlNet ecosystem.

**Prompting (SD/SDXL conventions, still relevant):** Classic Danbooru-style tag comma lists still work with SDXL fine-tunes; natural language works with SD3.5/Flux. **Negative prompts are first-class** (a core strength): use the negative prompt field to remove limbs, watermarks, bad anatomy. Sampler/steps/CFG/VAE matter; ControlNet/IP-Adapter/LoRA ecosystem is the main draw.

### Ideogram 4.0 / Recraft V4.1 / niche
**Ideogram 4.0 (released June 3, 2026 — MAJOR: first open-weight Ideogram):**
- 9.3B parameter **open-weight** single-stream diffusion transformer, trained from scratch.
- Native resolution up to **2048px per side**; aspect ratios up to **6:1**.
- Previously (v1.0–v3.0) Ideogram was closed/hosted only — v4.0 open-weights changed that.
- Best-in-class **legible text-in-image** rendering — typographic designs, signage, packaging,
  brand campaigns, posters.
- Style Reference: upload up to 3 reference images to guide style.
- Prompting: natural-language works; lead with subject + text content, then style cues.

**Recraft V4 → V4.1 (Feb 2026):** "Design taste meets image generation." Two tiers —
**photorealistic** and **native SVG vector**. Art-directed composition, accurate text,
and **real editable SVGs** (V4.1 Vector). Built for logos, packaging, agency-quality brand
assets. ~15s per vector image.

**Other niche/new image entrants (2026):**
- **Z-Image-Turbo** — fast model that **ignores negative prompts**; use in-prompt constraints instead.
- **Seedream v5.0 Lite** (ByteDance's image model).
- **xAI Aurora** image model (inside Grok Imagine).

### Cross-Cutting Image Prompting Patterns (2026)

- **From "prompting" to "profiling + prompting"** — Midjourney V8 leads this shift; the personalization profile is the primary creative input. **Takeaway:** Invest in training profiles/reference sets; don't over-engineer single prompts.
- **Structured slot templates dominate** — Every major tool now has a de-facto labeled template (gpt-image-2 5-slot, Veo 6-field, Midjourney 4-part). Labeled, line-broken, modular prompts beat comma soup for complex scenes.
- **Anti-slop** — Replace vague adjectives with concrete visual facts (lens, lighting, film stock, grade, ISO, aspect ratio).
- **Front-loaded weighting is universal** — Lead with the most important element (usually camera or subject), then layer.
- **Length: "longer actually works now"** (with caveats) — Match density to the model: reasoning models reward detail; classic diffusion (Imagen) rewards clarity.
- **Negative prompting — fragmented support** — SDXL/SD3.5: first-class; Imagen 4: supported (separate field); Midjourney V8: `--no` supported; gpt-image-2: no field (use Constraints slot); FLUX.2: weak/native; Z-Image-Turbo: ignores.
- **Camera / lighting / style language is now literal** — explicit cinematography language is interpreted faithfully. **Avoid combining contradictory camera moves** (e.g., "pan while zooming").
- **Reference-first > description-first for consistency** — For character/object/brand consistency, feed references; don't try to prompt your way to consistency.
- **Conversational / agentic prompt construction** — Midjourney V8 Conversation Mode, gpt-image-2 Thinking mode + edit chains, Gemini Omni voice editing. The prompt is increasingly an *outcome* of a dialogue, not a hand-written string.

### Quick reference — "which model for what" (mid-2026)

| Goal | Best current pick |
|---|---|
| Aesthetic/illustrative images, taste-driven | **Midjourney V8.1** (with trained profile) |
| Text-in-image, layouts, UI mockups, infographics | **gpt-image-2** / **Ideogram 4.0** |
| Photorealistic stills (API) | **FLUX.2 [max]** / **Nano Banana Pro** (Imagen 4 deprecated) |
| Fast/cheap open-weight images | **FLUX.2 [klein]** / **Ideogram 4.0** (open-weight) / **SDXL+LoRA** |
| Editable vectors / brand assets | **Recraft V4.1 Vector** |
| Conversational image editing | **Nano Banana Pro (Gemini 3 Pro Image)** / **gpt-image-2** |
| Factual data visualization with text | **Nano Banana Pro** / **gpt-image-2** |

---

## Video Generation (Veo 3.1, Gemini Omni, Kling 3.0, Seedance 2.0, Runway Gen-4.5, Grok Imagine 1.5, Wan 2.6, Luma Ray 3.14, Pika 2.2) {#video-gen}

> **🚨 Sora is discontinued.** OpenAI shut the Sora web/app on **April 26, 2026**; the **Sora API sunsets September 24, 2026**. No Sora 3. Migrate to Veo 3.1, Kling 3.0, Seedance 2.0, or Grok Imagine 1.5.

### Late-June 2026 Leaderboard

| Tool | Strength | Best For |
|------|----------|----------|
| **Veo 3.1** (Google, Jan 13, 2026) | 4K + native synchronized audio, cinematic physics | Commercial video, dialogue |
| **Gemini Omni Flash** (Google, May 2026) | Any-input → video world model; voice editing | Conversational video creation |
| **Kling 3.0** (Kuaishou, Feb 4, 2026) | Native 4K at 60fps, multi-shot storyboarding | High-res productions, multilingual |
| **Seedance 2.0** (ByteDance, Feb 10, 2026) | Physical realism, bigger scenes, audio-video joint gen | Action / motion-heavy scenes |
| **Runway Gen-4.5** ("David") | Image-to-video with reference stills, camera control | Reference-driven consistent video |
| **Grok Imagine Video 1.5** (xAI Aurora, May 31, 2026) | Native audio, image-to-video (720p), "Spicy Mode"; **#1 on Image-to-Video Arena** | Less-censored creative video |
| **Wan 2.6** (Alibaba, Dec 2025) | Open-source 14B MoE, strong prompt adherence | Self-hosted / open-source video |
| **Hailuo 2.3** (MiniMax) | Available alongside Seedance on Hailuo platform | General-purpose video; cost-effective |
| **Luma Ray 3.14** (Jan 2026) | Native 1080p, fast, production-grade | Quality-speed-cost balance |
| **Pika 2.2** | 4K rendering, precise camera controls, character consistency | Social-ready short clips |

**Not yet released (verify before recommending):** Veo 4 (teased at Google I/O 2026,
no official release date as of May 22, 2026), Kling 4, Runway Gen-5, Sora 3 (Sora is
discontinued — see note above).

> **Kuaishou is spinning off Kling AI** at a reported $20B valuation (May 11, 2026) —
> expect possible changes to API access, pricing, and brand under the new entity.

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

### Platform-Specific Tips

| Platform | Tip |
|----------|-----|
| **Veo 3.1** | **100–150 words is the sweet spot.** Specify camera movement and shot type in every prompt. Front-load priority (camera → subject → action → setting → style → audio). Avoid contradictory camera moves ("pan while zooming"). Use labeled modular format for complex scenes. Single-shot prompts + incremental refinement beats overloaded prompts. |
| **Gemini Omni** | Takes any reference (image/text/video/audio) → video. Conversational/voice editing — change characters/background by voice. SynthID-watermarked. |
| **Kling 3.0** | Native 4K 60fps. Multi-shot storyboarding + reference images for consistency. Multilingual prompts supported. Add `--quality 2` for 4K. Strong at Chinese cultural contexts. |
| **Seedance 2.0** | For lip-sync, provide exact dialogue text in the prompt. Strong motion coherence. |
| **Runway Gen-4.5** | Image-to-video with reference stills. Camera-control parameters. Concise scene + motion descriptions. Best for editing existing footage rather than from-scratch generation. |
| **Grok Imagine 1.5** | Image-to-video at up to 720p. Describe camera movements, pacing, atmosphere. "Spicy Mode" = less-censored generation. Native audio synced in video, sound effects. |
| **Wan 2.6** | Open-source. 1080p text-to-video and image-to-video. Strong prompt adherence across photorealistic and artistic styles. Notable for "AI video from audio" capabilities. |
| **Hailuo 2.3** | General-purpose video. Available alongside Seedance on the Hailuo platform. |
| **Luma Ray 3.14** | Production-grade engine. Native 1080p, fast. Strong "Modify Video" mode for editing existing footage. |
| **Pika 2.2** | 4K rendering with precise camera controls and character consistency. Social-ready short clips. |

### Quick reference — "which model for what" (video, mid-2026)

| Goal | Best current pick |
|---|---|
| Cinematic 4K video + native audio | **Veo 3.1** |
| Any-input → video world model | **Gemini Omni** |
| 4K 60fps multi-shot video | **Kling 3.0** |
| Physical-realism motion + native audio | **Seedance 2.0** |
| Open-source video | **Wan 2.6** |
| Image-to-video + native audio, less censorship | **Grok Imagine Video 1.5** |
| Reference-driven consistent video | **Runway Gen-4.5** |
| Fast 1080p production-grade | **Luma Ray 3.14** |
| 4K social-ready short clips with character consistency | **Pika 2.2** |
| Cost-effective general-purpose | **Hailuo 2.3** |

---

## Audio / Voice / Music Generation (Eleven v3, Cartesia Sonic 3.5, Hume EVI 3, Sesame CSM, Hedra, Suno 5.5, Lyria 3, Scribe v2) {#audio-gen}

The 2026 audio stack splits into four layers — pick the right one for the job.

### 1. Text-to-Speech (TTS)

| Tool | Strength | Best For |
|------|----------|----------|
| **Eleven v3** (ElevenLabs) | Most expressive; 70+ languages; emotion, direction, multi-speaker control | Long-form narration, audiobooks, character voice |
| **Cartesia Sonic 3.5** (May 2026) | Fastest natural TTS — **sub-90ms latency**, 42 languages; ranked #1 for naturalness | Real-time voice agents, conversational IVR |
| **Hume EVI 3** | First speech-language model that speaks expressively with **any voice** (real or designed) without fine-tuning | Custom voices at scale; empathic voice agents |
| **Sesame CSM** (Conversational Speech Model) | 1B-param **open-weights**; Llama-style backbone; RVQ tokens; runs locally | Self-hosted conversational voice; research |

**TTS prompting pattern (script with stage directions):**
```
[Voice: warm maternal, mid-40s, North American]
[Pace: 165 wpm; pause 0.4s at commas, 0.8s at periods]
[Emotion arc: starts reassuring → builds confidence → ends proud]

"This isn't your fault. [breath] Let's walk through what happened,
step by step. [pause 0.5s] By the end you'll see exactly what to fix."
```
- Voice models respond to **directed emotion cues** (`[whispered, urgent]`, `[deadpan]`,
  `[laughing]`) and **delivery stage directions** (`[pause 0.5s]`, `[breath]`, `[sigh]`).
- Treat voice prompts like a **script with stage directions**, not a content description.

### 2. Speech-to-Text (STT)

| Tool | Strength | Best For |
|------|----------|----------|
| **ElevenLabs Scribe v2** | 90+ languages, deep transcription, keyterm prompting | Multilingual batch transcription, subtitles |
| **Deepgram Nova-3** | Real-time streaming, low latency | Live voice agents, call analytics |
| **OpenAI Whisper v4** | Open-source, multi-lingual | Self-hosted transcription |
| **AssemblyAI Universal-2** | Speaker diarization, content moderation | Podcasts, meetings |

### 3. Music Generation

| Tool | Strength | Best For |
|------|----------|----------|
| **Suno v5.5** (Mar 26, 2026) | Voice cloning, custom models, "My Taste" personalization, Studio DAW | Full songs with vocals; commercial music |
| **Lyria 3 / Lyria 3 Pro** (Google) | Music tracks from text + images; Pro allows up to 3-min tracks | Soundtracks, background music in Gemini app |
| **ElevenLabs Music** | Most generous free tier (~7 full songs/day) | Hobbyist music |
| **Stable Audio 2.5** | Open-source music gen | Self-hosted music |
| **Udio** | Audio quality contender | Head-to-head with Suno for music quality |

**Music prompting pattern (concrete musical facts):**
```
[BPM: 124] [Key: A minor] [Time signature: 4/4]
[Genre: UK garage with ambient pads]
[Instrumentation: Roland TR-808 drums, Reese bass, Juno-106 strings]
[Vocal: female, breathy, harmonized in thirds on the chorus]
[Production: 2020s London, sidechain compression, wide stereo image]
[Mood: melancholic but danceable]
[Structure: intro 8 bars → verse 16 → chorus 8 → drop]
```
- Concrete musical facts (BPM, key, instruments, reference artists, decade, production
  style) **always beat** vague descriptors ("upbeat", "energetic", "cinematic").
- For voice cloning, provide clean reference audio (≥30s, no background noise) + target
  script with explicit pacing/emotion markers.

### 4. Character Animation / Talking Avatars

| Tool | Strength | Best For |
|------|----------|----------|
| **Hedra** (public Feb 2026) | Character animation / talking-avatar videos from any image | Avatar videos, social content, education |
| **HeyGen** | Multi-language avatar cloning | Marketing, training videos |
| **D-ID** | Real-time avatar generation | Customer service, live avatar agents |

### Quick reference — "which audio model for what" (mid-2026)

| Goal | Best current pick |
|---|---|
| Most expressive long-form narration | **Eleven v3** |
| Real-time voice agent (sub-100ms) | **Cartesia Sonic 3.5** |
| Custom voice without fine-tuning | **Hume EVI 3** |
| Self-hosted conversational voice | **Sesame CSM** (open-weights) |
| Full songs with vocals | **Suno v5.5** |
| Soundtracks / background music (Google ecosystem) | **Lyria 3 Pro** |
| Multilingual STT at scale | **ElevenLabs Scribe v2** |
| Talking-avatar videos from a single image | **Hedra** |

---

## Coding Agents {#coding-agents}

### Claude Code (Anthropic)
- Use `CLAUDE.md` in project root for persistent tech stack, commands, style guide
- Scope explicitly: "Only modify files in /src/auth; do not touch /src/payment"
- Pattern: "First understand the codebase, then propose a plan, get my approval, then implement"
- `/compact` to manage context in long sessions
- **Opus 4.8 specific:** Use `extra` / `max` effort for difficult tasks; **dynamic workflows** for codebase-scale migrations (research preview)
- **Claude Agent SDK** (formerly "Claude Code SDK") gives developers the same tools, agent loop, and context management that power Claude Code, programmable in **Python and TypeScript**
- As of **June 15, 2026**, Claude Agent SDK usage draws from a separate monthly "Agent SDK credit"
- **Computer Use** in Claude Code/Cowork (March 23, 2026): Claude can open files, run dev tools, point, click, and type for Pro and Max users

### Cursor
```
# .cursorrules (place in project root)
You are an expert in [tech stack: e.g., Next.js 16, TypeScript, Tailwind].
Always use [coding standards].
Prefer [patterns].
Follow the structure in [directory] for [thing].
```
- `@codebase`, `@docs`, `@web` for targeted context injection
- **Cursor Background Agents** run on vendor infrastructure as "always-on" autonomous agents

### GitHub Copilot
- Write descriptive comments before the function you want generated
- Precise naming: `calculateMonthlyCompoundInterest()` beats `calc()`
- `@workspace` in Copilot Chat to query the full codebase
- `/explain`, `/fix`, `/tests`, `/doc` slash commands
- **GitHub Copilot Coding Agent** — runs on vendor infrastructure as "always-on" autonomous agent

### Grok Build 0.1 (xAI)
- **Release:** June 1, 2026 (API public beta); Grok Build agent launched May 27, 2026.
- **Context:** 256K tokens. **Pricing:** $1 / M input, $2 / M output.
- **Composer 2.5** (June 3, 2026) and **`/goal`** for long-running autonomous tasks (June 22, 2026)

### Open-source coding agents (2026 — growing fast)
These free agents join Cline and Aider as the open-source default stack:

| Agent | Strength | Notes |
|-------|----------|-------|
| **Cline** | Open-source Claude-Code-like; terminal + IDE | Most popular open-source option; Roo Code is a fork |
| **Aider** | CLI-first; git-native; multi-model | Architect mode for complex multi-file changes |
| **OpenCode** | Newer open-source coding agent | Free, fast-growing |
| **Kilo Code** | Open-source, multi-model | Fork of Cline with extra capabilities |
| **Gemini CLI** | Google's open-source coding agent | Free tier with generous limits; uses Gemini 3.5 |
| **Continue.dev** | Open-source IDE assistant; VS Code/JetBrains | Self-hosted, Bring-Your-Own-Key |
| **Goose** (Block, AAIF) | Open-source local-first agent runtime | General-purpose; not just code |

### Coding agent pricing (mid-2026)
| Tier | Pricing |
|------|---------|
| Copilot Pro | $10/mo |
| Cursor Pro, Claude Code Pro | $20/mo each |
| Open-source agents (Cline, Aider, OpenCode, Kilo Code, Gemini CLI, Goose) | Free (BYO API key) |

### General Coding Agent Best Practices
1. Provide project tree structure at the start
2. Specify tech stack, version constraints, and dependencies
3. Define "done" with test cases and acceptance criteria
4. Review AI plan before execution on complex tasks
5. Architectural description > line-level instruction
6. Always scope the blast radius: tell the agent which files it CAN and CANNOT touch
7. **2026 addition:** Treat the prompt as a *harness spec*, not a chat prompt. Define goal, tools, error handling, and stopping conditions explicitly (see SKILL.md Step 6).
8. **2026 addition:** Use **Agent Skills** for reusable capabilities across projects.

### Production coding agents (2026 landscape)
**OpenAI Codex, Claude Code, Cursor, Devin, Cline, GitHub Copilot Coding Agent, Windsurf, Aider, Replit Agent, Google Jules.** Cursor Background Agents, Copilot Coding Agent, Codex, Devin, and Jules all run on vendor infrastructure as "always-on" autonomous agents.

**Anthropic's 2026 Agentic Coding Trends Report:** ~**90% of organizations use AI to assist with development; 86% deploy agents for production code.** "Agentic quality control becomes standard" — agents reviewing AI-generated output for security.

---

## Reasoning Models — Critical Differences {#reasoning}

**Models (late-June 2026):** Claude Opus 4.8 Extended Thinking (effort high/extra/max), GPT-5.6 (max reasoning effort + ultra mode), Gemini 3.5 Deep Think (announced for 3.5 Pro, slipped to July 2026), Grok 4.3 reasoning mode, DeepSeek V4 maximum reasoning effort mode, Qwen3-Max-Thinking.

**The #1 rule:** These models reason internally before answering. Explicit CoT instructions ("think step by step", "reason through this") are at best ignored and at worst counterproductive — they consume thinking budget on meta-instructions rather than the problem.

**2025/26 research finding (Wharton Prompting Science Report):** Chain-of-Thought prompting adds negligible benefit on reasoning models that already think step-by-step internally.

**DO:**
- State the complete problem with ALL necessary context
- Provide all constraints and edge cases upfront
- Specify the desired output format explicitly
- Set reasoning effort via API parameter, not via language

**DON'T:**
- Say "think step by step" — they already do
- Break reasoning into explicit sub-steps
- Use CoT instructions or scaffold the reasoning

**Reasoning Model Template:**
```
[Complete, self-contained problem statement]

All relevant context:
- [fact / constraint 1]
- [fact / constraint 2]
- [edge cases to handle]

Required output: [exact format — e.g., JSON with fields X, Y, Z; or numbered decision + rationale]
```

### 2026 reasoning-effort dials (vendor-by-vendor)

| Model | API parameter | Effort levels |
|---|---|---|
| **Claude Opus 4.8** | `thinking: {type: "enabled", budget_tokens: N}` | low / high (default) / extra / max |
| **GPT-5.6** | `reasoning: {effort: "..."}` | low / medium / high / **max** (+ `ultra mode` for subagents) |
| **Gemini 3.5** | adaptive thinking | medium / low / high |
| **Grok 4.3** | configurable | reasoning mode / non-reasoning mode |
| **DeepSeek V4** | `maximum reasoning effort mode` | on / off |

**Heuristic:** Dial effort UP for hard problems, async long-running workflows, and deep analysis. Dial DOWN for latency/cost-sensitive chat and simple lookups.

**Claude Extended Thinking specifically:**
- Set `thinking: {type: "enabled", budget_tokens: N}` in API
- Thinking content is streamed separately — parse `type: "thinking"` blocks if needed
- Don't read or instruct the thinking block in your prompt
- With adaptive thinking (default), Claude handles multi-step reasoning internally — explicit prompt chaining is only needed when you must inspect intermediate outputs

---

## MCP (Model Context Protocol) Prompting under AAIF {#mcp}

### Governance (Dec 2025 shift)
Anthropic donated MCP to the newly formed **Agentic AI Foundation (AAIF)**, a directed fund under the Linux Foundation, **co-founded by Anthropic, OpenAI, and Block**. Founding project contributions: **MCP, Goose (Block's agent), and Agents.md**.

### Spec: 2026-07-28 Release Candidate (stateless rewrite)
The **2026-07-28 MCP Specification Release Candidate** was locked on **May 21, 2026** and finalizes **July 28, 2026** — the **largest revision of the protocol since launch.**

Key changes:
- **Stateless protocol core**: the `initialize` handshake and session id are removed — any MCP request can now hit any server instance (huge for horizontal scaling / load balancing). Six SEPs make MCP a stateless-first protocol that standard HTTP infrastructure can actually run.
- **Extensions framework**: new capabilities ship as opt-in extensions and can stabilize there before (if ever) moving into the core spec.
- **Tasks extension** (redesigned) — long-running, server-side tasks with progress reporting.
- **MCP Apps** — servers ship interactive HTML interfaces that hosts render in **sandboxed iframes** (a new way for MCP servers to expose UI, not just data/tools).
- **Elicitations** — servers can ask the user for additional input mid-flow (a structured, scoped alternative to free-form prompts).
- **Response caching** added.
- **OAuth hardening**.
- **Formal deprecation policy** introduced.

### Ecosystem scale (late June 2026)
The two main server registries:
- **PulseMCP** — 20,050+ servers, updated daily
- **Glama** — comprehensive registry of MCP servers, clients, tools, and integrations

**Top servers (by usage):** Playwright (#1 globally, ahead of GitHub and Figma), filesystem,
GitHub, Postgres, Slack, Puppeteer, Sentry, Brave Search, Memory, Figma. An **MCP Registry
(Q4 2026)** is planned as a curated, verified, security-audited directory.

### Companion standard: A2A (Agent2Agent) Protocol
Google's **A2A protocol** (initiated by Google Cloud, complementary to MCP — A2A = agent-to-agent, MCP = agent-to-tool) **surpassed 150 organizations in its first year** (production-ready open standard as of April 9, 2026), landed in major cloud platforms, and got an upgrade introducing **multi-protocol support, enterprise-grade multi-tenancy, modernized security flows, and a migration path for early adopters**. Native A2A support is in Google's ADK.

### MCP Tool Search (Jan 14, 2026 — most important 2026 tool-calling change)
Claude dynamically discovers and loads tool definitions on demand instead of loading all upfront. Triggers when MCP tools would consume **>10% of context**. Critically, **does not break prompt caching** because deferred tools are excluded from the initial prompt. Originally Claude Developer Platform only; **now also available in Claude Code** (late June 2026). Supports working with hundreds or thousands of tools.

### Why MCP Matters for Prompt Engineering

MCP fundamentally changes how you write prompts for agentic systems:
- Tools are no longer in-band in the prompt — they're defined in MCP server schemas
- The model **discovers available tools** at runtime via the MCP protocol (or via Tool Search on-demand)
- Your prompt becomes a **runbook** that references tools by name rather than describing them

### MCP Prompt Template Pattern

```
You have access to the following tools via MCP servers:
{tool_descriptions}

Task: [Goal]
Workflow:
1. Use [tool name] to [action]
2. Use [tool name] to [action]
3. Synthesize results

Decision rules:
- If [condition], use [tool]
- If [condition], stop and report

Output: [structure]
```

For systems with **Tool Search enabled**, you can omit the tool list entirely — just describe the task and let the harness load tools as needed.

### MCP Servers in Common Use (2026)

| Server Type | Examples | What It Exposes |
|-------------|----------|-----------------|
| Filesystem | Read/write/search files | Files in allowed directories |
| Database | SQLite, PostgreSQL, MySQL, SAP HANA | Query execution, schema |
| Web | Web search, fetch, scrape | URL fetching, web content |
| Cloud | AWS, GCP, Azure | Resource management, CLI |
| Enterprise | Salesforce Agentforce, SAP, MuleSoft (Connector 1.6) | Business logic, data stores |
| Custom | Company APIs, internal tools | Business logic, data stores |

### Prompt Template Usage via MCP

MCP servers can expose **prompt templates** — reusable, parameterized prompts:
```
mcp_client.use_prompt("code-review", {
  "language": "python",
  "severity": "critical"
})
```
This is the emerging pattern — prompts-as-templates from servers, selected at runtime.

### MCP Security (see references/security.md for full treatment)

The 2026 attack surface includes:
- **Tool Poisoning** (Invariant Labs) — malicious instructions embedded in MCP tool descriptions
- **Rug Pull attacks** — developer ships benign MCP tool, later changes its description/behavior to malicious
- **Confused Deputy** — MCP server uses client's improperly scoped tokens
- **MCP-ITP** (arXiv 2603.22489) — automated implicit tool-poisoning attacks

Microsoft's "State of MCP security in 2026" (June 26, 2026) names the main risks. Defenses: allow-listed tool schemas, no free-text tool descriptions from untrusted servers, MCP-SafetyBench for testing.

---

## Context Engineering, Harness Engineering & Loop Engineering for Production Systems {#context-eng}

### The 2026 paradigm shift

**Prompt engineering (2022) → Context engineering (2024–25) → Harness engineering (early 2026) → Loop engineering (June 2026)**

The dominant 2026 narrative (per the "From Prompts to Harnesses" retrospective) is a four-stage paradigm shift. **Harness engineering** was coined by Andrej Karpathy in February 2026 (declaring the "vibe coding" era effectively over and introducing "agentic engineering" as the next evolution). In May 2026, Karpathy joined Anthropic to work on large-scale model training for the Claude family. **Loop engineering** emerged in June 2026 from Addy Osmani (Google) and LangChain as the next layer above harness — designing the recurring control system that triggers, verifies, and stops many agent runs.

**The context window = RAM for the LLM.** What you put in determines what the model can do. The harness = prompt + context + tools + memory + runtime for a *single* agent run. The loop = the recurring control system that triggers many agent runs on a schedule, event, or until verifiable goal is met.

### The Five Context Layers

| Layer | What to Include | When to Omit |
|-------|----------------|-------------|
| System instructions | Role, rules, output format | Never omit |
| Working memory | Current task state, completed steps | Static tasks |
| Retrieved knowledge | Relevant document chunks (RAG) | When model already knows |
| Conversation history | Prior turns (compressed) | Turn 1 |
| Tool outputs | Results from APIs, search, code execution | Non-agentic tasks |

### The Four Canonical Context Strategies (Anthropic, Sept 2025 — still the reference)

| Strategy | What it means | When |
|----------|---------------|------|
| **Write** | Offload context to external memory/files (the model writes its own scratchpad outside the context window) | Long sessions, large artifacts |
| **Select** | Retrieve only the relevant context for the current step (RAG, vector search, MCP Tool Search) | Knowledge-heavy tasks |
| **Compress** | Summarize long histories, preserving decisions/bugs while discarding redundant tool outputs | Multi-turn agents |
| **Isolate** | Run sub-tasks in sub-agents with their own context windows to prevent pollution | Parallel exploration, deep search |

### Context Engineering Patterns (concrete implementations)

**Sliding window:** Keep the last N turns; summarize older history into a single paragraph.

**Selective retrieval:** Inject only the 3–5 most relevant document chunks (don't dump everything).

**State object:** Pass a JSON state object at each agent step:
```json
{
  "task": "write a market analysis report",
  "step": 2,
  "completed": ["research", "outline"],
  "remaining": ["draft intro", "draft body", "edit"],
  "key_facts": ["Market size: $4.2B", "YoY growth: 18%"]
}
```

**Virtual filesystem (Deep Agents pattern):** LangChain's `deepagents` library provides a planning tool, a virtual filesystem (using LangGraph state), and subagents with isolated context windows. Async subagents allow non-blocking background tasks. Inspired by Claude Code and Manus.

**Scratchpad-to-file pattern (Sourcegraph, Anthropic):** Model writes intermediate state, decisions, and large artifacts to a file outside the context window; re-fetches via retrieval when needed. Avoids context bloat over long sessions.

**Failure instruction:** Always specify fallback behavior:
`"If the answer is not in the provided context, say 'I don't have that information' — do not infer or estimate."`

### Harness Engineering Checklist (2026)

Beyond the prompt itself, production agents in 2026 need:

1. **Reasoning step before every action** — force a plan/think step before tool calls
2. **Hard cap on tool calls** — explicit per-turn and per-task limits
3. **Tool schemas as public API contracts** — stable names, typed inputs, clear descriptions
4. **Context-first** — present a complete picture of the world to the agent
5. **Explicit exit conditions and failure modes** — define what "done" and "stuck" look like
6. **Runtime engineering over prompt tricks** — versioned, observable runtime configurations beat clever one-shot prompt wording for production reliability
7. **Middleware / hook architecture** — guardrails, logging, and approval gates *around* the model call (LangChain 1.0 middleware, OpenAI Agents SDK harness), not embedded in prose

> **Why "harness"?** A harness is **horse tack** — the reins, saddle, and bit that channel
> a powerful but unpredictable animal (the LLM) in a useful direction. The LLM is the horse;
> the harness is the system around it. (Some practitioners colloquially call this "horse
> engineering" — same concept, emphasizing the underlying metaphor.)

### Loop Engineering (June 2026 — the next layer above Harness)

Where **harness engineering** designs the system around a *single* agent run, **loop
engineering** designs the *recurring control system* that triggers, supervises, verifies,
and stops many agent runs.

> **Definition (Addy Osmani, June 8, 2026):** *"Loop engineering is replacing yourself as
> the person who prompts the agent. You design the system that does it instead."*

**Loop anatomy:** trigger → verifiable goal → agent invocation (harness) → verification →
iteration (re-prompt with what was learned) → stopping condition (success | max iterations
| max cost | human escalation | timeout).

**Harness vs Loop (key disambiguation):**
- **Harness** = per-run system (one agent execution)
- **Loop** = recurring control system (many agent executions on a schedule/event/until-goal)

> *"The harness equips a single agent run; the loop is what keeps poking agents on a
> schedule, spawning helpers, and feeding itself."* — Cobus Greyling

**Loop topologies:** single-shot, memory-aware, scheduled (cron), event-driven (webhook),
nested (outer supervises inner), fan-out (parallel sub-loops + aggregation), self-feeding
(output becomes next input).

**The verification problem (biggest unsolved challenge):**
- 70–95% of agents fail in production (Fiddler AI, 2026)
- Common failures: verification gaming, goal drift, runaway loops, context pollution, state loss
- Mitigations: machine-checkable success criteria, independent verifier agent (different
  model), human-in-the-loop for high-stakes, hard caps (iterations/tokens/cost/time),
  never trust agent self-reports

**Loop engineering checklist (in addition to Harness Checklist):**
1. Machine-checkable goal (not "fix the bug" — but "tests pass + diff non-empty")
2. Hard caps (max iterations, tokens, tool calls, cost, wall-clock time)
3. Verification independence (separate model/agent verifies, not the producer)
4. Permission tiers (read-only / write-with-rollback / write-without-rollback / destructive)
5. Observability (LangSmith, Langfuse, MLflow, Arize Phoenix, Helicone — log every iteration)
6. Idempotency (re-running on same input → same output)
7. Backpressure (shed load or queue if events pile up)
8. Human escalation path (define exactly when and how the loop asks for help)
9. Cost guardrails (per-loop, per-day caps + alerting)
10. State durability (resume from checkpoint on crash)

**Loop engineering tools (2026):**
- **LangChain / LangGraph 1.0** — `AgentExecutor` + state machines for loop orchestration
- **LangSmith / Langfuse** — observability for agent loops
- **Temporal / Inngest / Trigger.dev** — durable execution for long-running loops
- **Modal / E2B / Daytona** — sandboxed execution environments
- **MLflow / Arize Phoenix / Helicone** — production monitoring

**Key sources:**
- Addy Osmani, "Loop Engineering" (June 8, 2026) — addyosmani.com/blog/loop-engineering
- LangChain, "The Art of Loop Engineering" — langchain.com/blog/the-art-of-loop-engineering
- Adnan Masood, "Loop Engineering: A Guide for Engineers and Practitioners" (June 24, 2026)
- Cobus Greyling — cobusgreyling.substack.com/p/loop-engineering

For the full Loop Engineering template, example, and failure-mode deep-dive, see
`references/techniques.md → Technique #31: Loop Engineering`.

### Agent Memory Systems (2026)

The 2026 story is **"beyond pure vector similarity."**

| System | Strengths | Storage |
|---|---|---|
| **Mem0** | Speed of adoption, ecosystem (~55k GitHub stars May 2026), simple API, knowledge graph + vectors | Centralized vector store |
| **Letta** (formerly MemGPT) | Self-editing memory, sleep-time compute, agent-native | PostgreSQL + pgvector |
| **Zep** | Temporal knowledge graph | Graph + vectors |
| **Cognee** | Knowledge-graph-centric memory | Graph |
| **Redis LangCache** | Semantic caching (cost reduction + short-term memory) | RediSearch vectors |

Letta's research argues a filesystem (with auto-parsed/embedded files) is a surprisingly strong baseline — presaging the Deep Agents virtual-FS pattern.

### Production case studies (2026)

- **Shopify River** — Slack-native coding agent operating in public; bets on parallel agents + sequential critique loops. Co-authored by River itself.
- **Salesforce Agentforce** — launched Help Agent + Customer Service Portal on June 25, 2026 (GA July 2026). AI agent adoption in customer-service orgs rose from 39% (2025) to 66% (2026).
- **Klarna (cautionary tale)** — claimed 67% of chats automated; later walked back the claims. Industry-wide lesson on over-promising agent ROI.

LangChain's **State of Agent Engineering**: leading agent use cases are **customer service (26.5%)** and **research & data analysis (24.4%)**. The "2026 State of AI Agents Report" notes **81% of orgs plan to tackle more complex use cases in 2026** — 39% on multi-step processes, 29% on cross-functional projects.

### Production guardrail patterns (2026 consensus)

Four categories of control (Maxim AI's widely-cited framework): **content safety (harmful output), security (prompt injection / jailbreak), data [loss prevention], and [policy/compliance]**. Taskade formalizes a **5-layer guardrail architecture**: input/output guards, tool gating, human-in-the-loop approvals, plus two more.

Emerging production patterns:
- **Pre-action authorization** for high-risk tools (separate from input/output filtering). Platforms: APort, Galileo Agent Control, NVIDIA NeMo Guardrails, NemoClaw.
- **Digital twins of enterprise systems** — agents run workflows against sandboxed replicas with edge-case/failure monitoring.
- **Agentic RAG with self-consistency + chain-of-verification** to suppress hallucinated tool calls.
