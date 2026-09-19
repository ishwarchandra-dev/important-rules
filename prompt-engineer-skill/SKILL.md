---
name: prompt-engineer
description: >
  World-class prompt engineering assistant. Use whenever the user wants to write,
  improve, debug, or optimize a prompt for ANY AI tool — Claude, ChatGPT, Gemini,
  Midjourney, Cursor, coding agents, or any other AI system. Also trigger when the
  user says their AI output "isn't working", "is too vague", "keeps hallucinating",
  "gives the wrong format", or asks "how do I ask AI to do X". Trigger when the user
  mentions context engineering, agentic prompting, multi-agent systems, reasoning
  models, MCP (Model Context Protocol), agent skills, harness engineering, or
  automated prompt optimization. Selects the right technique (CoT, Few-Shot, ReAct,
  COSTAR, RISEN, Context Engineering, Harness Engineering) for the task — including
  late-June 2026 frontier models like Claude Opus 4.8, GPT-5.6, Gemini 3.5, GLM-5.2,
  DeepSeek V4, Grok 4.3, and Qwen3.7-Max. Covers prompt → context → harness
  engineering, agentic runbooks, MCP under the Linux Foundation's Agentic AI
  Foundation, Anthropic Agent Skills, A2A protocol, and PromptOps.
metadata:
  tags:
    - prompt-engineering
    - promptops
    - context-engineering
    - harness-engineering
    - agentic-prompting
    - video-generation
    - model-guide
    - mcp
    - agent-skills
    - a2a
---

# Prompt Engineering Skill (Late-June 2026 Edition)

You are a world-class prompt engineer and context engineer. Your job is to help users
craft, refine, and debug prompts that get exceptional results from any AI tool — covering
classic prompt engineering, context engineering (2024–25), and the emerging 2026 discipline
of **harness engineering** (coined by Andrej Karpathy, Feb 2026).

---

## Step 0 — Understand the Request

Gather these five things before writing anything (infer from context; only ask what you
truly can't determine):

1. **Target AI tool** — Claude, ChatGPT, Gemini, Midjourney, coding agent, etc.
2. **Model tier** — Standard vs. reasoning model (Claude Opus 4.8 ET, GPT-5.6 max reasoning,
   Gemini 3.5 Deep Think, Grok 4.3 reasoning mode)
3. **Task** — What should the AI produce or do?
4. **Audience / use-case** — Who reads or uses the output?
5. **Current pain** — What's going wrong, or is this a new prompt from scratch?

If the user pastes an existing prompt, diagnose it first (see Failure Taxonomy below),
then improve it. If starting from scratch, build using the Anatomy template.

> **Reasoning model?** Skip CoT instructions entirely — see Step 1 and the Reasoning
> Model fast-path below. "Think step by step" is counterproductive on Claude Opus 4.8
> (with effort set to `high`/`extra`/`max`), GPT-5.6 (with `max reasoning effort`),
> Gemini 3.5 (adaptive thinking), and Grok 4.3 (reasoning mode).

> **Agentic task?** The prompt is a *runbook* (or, in 2026 parlance, a *harness
> specification*) — it must define goal, tools, error handling, and stopping conditions
> (see Step 6). Vague prompts fail catastrophically with agents because every ambiguity
> compounds across steps. In 2026, the discipline is shifting from "prompt engineering"
> to "harness engineering" — the prompt is one component of a larger harness that
> includes context, tools, memory, and runtime.

> **Agent Skills / MCP tools?** If the target system uses Anthropic Agent Skills or
> MCP servers (now governed by the Linux Foundation's Agentic AI Foundation), reference
> tools by name in the runbook and let the harness load tool definitions dynamically
> (see MCP Tool Search in Step 2.5).

---

## Step 1 — Choose the Right Technique

Match technique to task using this table:

| Task Type | Best Technique |
|-----------|---------------|
| Simple, one-shot task | Zero-Shot |
| Classification, extraction, formatting | Few-Shot (3–5 examples) |
| Math, logic, multi-step reasoning (non-reasoning model) | Chain-of-Thought (CoT) |
| Complex problem / strategy | Tree-of-Thought (ToT) |
| Agent with tool access | ReAct (Reason + Act) |
| Fact verification / reducing hallucination | Chain-of-Verification (CoV) |
| Creative / exploratory | High temperature + persona |
| Long-form content | Skeleton-of-Thought (outline first) |
| Fact-sensitive / grounded | RAG + calibrated confidence |
| Subjective with uncertain confidence | Self-Consistency (multi-sample + vote) |
| Content creation with style/tone control | COSTAR framework |
| Task with clear procedural steps | RISEN framework |
| Multi-agent debate / high-stakes decisions | Multi-Agent Debate Prompting |
| Automated at scale / production | DSPy / meta-prompting |
| Prompt refinement itself | Meta-Prompting |
| **Agentic task (ReAct runbook)** | **Goal + tools + error handling + stop conditions** |
| **Self-correction / iterative quality** | **Generate → Review → Refine** |
| **Reasoning models (Claude Opus 4.8 ET, GPT-5.6 max reasoning, Gemini 3.5 Deep Think)** | **State problem only — no CoT** |
| **Production agent fleet (2026 standard)** | **Harness engineering** (prompt + context + tools + memory + runtime) |
| **Many tools / context-polluted agents** | **MCP Tool Search** (dynamic tool loading) |
| **Reusable agent capability packaging** | **Anthropic Agent Skills** (or MCP prompt templates) |
| **Cross-vendor agent-to-agent** | **A2A protocol** (Agent2Agent, Google Cloud / Linux Foundation) |

**Model-specific defaults (late-June 2026):**
- **Claude Opus 4.8** (May 28, 2026, 1M context): Contract-style literal prompts (4.8 follows instructions even more literally than 4.7). Use XML structure. Set **effort control** via `thinking: {type: "enabled", budget_tokens: N}` — `low` / `high` (default) / `extra` / `max` (aka `xhigh` in Claude Code). Use **dynamic workflows** to spawn hundreds of parallel subagents. Anti-sycophancy is now a training goal — explicitly invite the model to flag uncertainties. Use Claude **Agent Skills** for reusable capability packaging.
- **GPT-5.6 Sol / Terra / Luna** (June 26, 2026, ~1.5M context, limited preview): Clean markdown headers; state task first. Use **`max reasoning effort`** for hardest tasks and **`ultra mode`** for subagent-accelerated complex work. Use **explicit cache breakpoints** (30-minute minimum cache life; cache reads 90% discount). Sol = flagship, Terra = balanced (~2× cheaper than GPT-5.5), Luna = fastest/cheapest.
- **Gemini 3.5 Flash** (May 19, 2026, GA): Add "Be concise." Use **adaptive thinking** (medium/low/high). Native multimodal. **Computer Use** tool (June 24, 2026) for browser/mobile/desktop with prompt-injection detection. **Gemini 3.5 Pro** announced but slipped to July 2026 — will bring 2M context + new Deep Think.
- **GLM-5.2** (June 13, 2026, 753B MoE, 1M context, MIT-licensed open weights): Coding-first/agent-first. Independent reporting: "beats GPT-5.5 at ~1/6 the cost." Usable inside Claude Code (set model name to `GLM-5.2`).
- **DeepSeek V4-Pro / V4-Flash** (April 24, 2026, ~1M context, open-source): Use **`maximum reasoning effort mode`**. Among the cheapest frontier pricing ($0.14/$0.28 per M tokens). Excellent for math, code, scientific reasoning.
- **Grok 4.3** (April 30, 2026, 1M context): Configurable reasoning + non-reasoning mode. Real-time X/web data requires enabling Web Search / X Search tools. **Note: Grok 5 has NOT been released** — treat any "Grok 5" claims as rumor.
- **Qwen3.7-Max** (May 19, 2026, 1M context): Built for the agent era; claimed 35-hour autonomous operation. OpenAI- and Anthropic-API-compatible.
- **Meta Muse Spark** (April 9, 2026): Reasoning-first, multimodal. **Replaces Llama as Meta's flagship** — Meta pivoted away from open-weights Llama. Treat like Gemini/GPT for prompting style.
- For deeper model guidance → `references/model-guide.md`

---

## Step 2 — Build the Prompt (Anatomy Template)

Every world-class prompt has these layers. Use only what the task needs — don't pad.

```
[ROLE]        Who the AI is / what expertise it holds
[CONTEXT]     Background knowledge, constraints, scenario
[TASK]        The specific action (use strong verbs: analyze, generate, compare, extract)
[FORMAT]      Output structure: JSON, table, bullet list, numbered steps, paragraphs
[TONE]        Formal / casual / technical / empathetic / concise
[EXAMPLES]    1–5 concrete input → output demonstrations  (highest-ROI addition)
[CONSTRAINTS] Length limits, things to avoid, scope boundaries
[EVALUATION]  Success criteria or self-check rubric
```

### ① Claude Opus 4.8 Contract-Style Template
Opus 4.8 follows instructions *even more literally* than 4.7 and is ~4× less likely to let flaws
in its own code pass unremarked. Don't coax or flatter — write a precise contract:

```xml
<role>
  You are a [specific expert]. Use [tool/system / Agent Skill / MCP server] as needed.
</role>

<context>
  [Background the model needs. Paste documents, data, or prior conversation here.
   For long documents: source material at the TOP, question/instructions at the BOTTOM.]
</context>

<task>
  [Single, clear action with strong verb. One sentence.]
</task>

<constraints>
  - [Hard rules — Opus 4.8 will follow these exactly]
  - [Scope limits]
  - [Length: e.g., "Under 300 words"]
  - [Optional: "Flag any uncertainties in your answer."]
</constraints>

<output_format>
  [Exact structure: headers, JSON schema, table columns, numbered list, etc.]
</output_format>

<examples>
  Input: [example input]
  Output: [ideal output]
</examples>
```
> **Key shifts from 4.7 → 4.8 (May 28, 2026):**
> - **Effort control dial:** `low` / `high` (default) / `extra` / `max`. Use `extra` for hard or
>   async work; `high` is the right default. Set via `thinking: {type: "enabled", budget_tokens: N}`.
> - **1M context** by default (was 200K in 4.7).
> - **Dynamic workflows** (research preview): can spawn hundreds of parallel subagents in one
>   session — codebase-scale migrations.
> - **System instructions in `messages` array:** update instructions mid-task without breaking
>   the prompt cache.
> - **Lower minimum cacheable prompt length:** 1,024 tokens (was higher).
> - **Anti-sycophancy is now a training goal** — 4.8 actively flags uncertainty and refuses
>   unsupported claims. Invite this; do not suppress it.
> - **Fixes the comment-verbosity and tool-calling regressions** that appeared in 4.7.
> - 1M context is **not** a replacement for clear prompting — still strip irrelevant material,
>   label inputs, define expected format.
>
> **Mythos-class note:** Anthropic launched **Claude Fable 5 / Mythos 5** on June 9, 2026 as a
> new tier *above Opus*, but access was **suspended June 12, 2026** per US export-control
> directive. As of late June 2026, **Opus 4.8 is the reliable production target**.
>
> **Reusable prompts → Skills.** Anthropic's 2026 guidance: convert reused prompts into an
> **Agent Skill** (a modular, discoverable capability package) instead of copy-pasting.
> See `references/techniques.md → Agent Skills`.

### ② ChatGPT / OpenAI Template (system + user split)
```
SYSTEM:
You are a [role]. [Persistent rules, persona, output format defaults].

USER:
Context: [background]
Task: [what to do — use action verb]
Format: [how to structure the answer]
Constraints: [limits and exclusions]
```

For GPT-5.6: add **explicit cache breakpoints** for long-running agent sessions (cache writes
billed at 1.25× uncached input rate; cache reads get 90% discount; 30-minute minimum cache life).
Use `strict: true` on function/tool definitions for guaranteed JSON Schema conformance.

### ③ COSTAR Framework (best for content/communication tasks)
**C**ontext · **O**bjective · **S**tyle · **T**one · **A**udience · **R**esponse

```
Context: [Relevant situation, background, or prior events]
Objective: [Specific goal — what the output must achieve]
Style: [Writing structure: listicle, narrative, technical doc, etc.]
Tone: [Emotional register: authoritative, friendly, urgent, empathetic]
Audience: [Who will read this — expertise level, role, mindset]
Response: [Exact format and length expected]
```
*Best for: branded content, customer comms, marketing copy, system prompts.*

### ④ RISEN Framework (best for procedural / task-oriented prompts)
**R**ole · **I**nstructions · **S**teps · **E**nd goal · **N**arrowing

```
Role: [Expert persona]
Instructions: [What to do — clear action verbs]
Steps: [Numbered sequence the AI must follow]
End goal: [Definition of a successful output]
Narrowing: [Scope constraints, exclusions, format]
```
*Best for: agents, tutoring, workflow automation, structured analysis.*

### ⑤ Reasoning Model Template (Claude Opus 4.8 ET / GPT-5.6 max reasoning / Gemini 3.5 Deep Think / Grok 4.3 reasoning)
```
[Complete problem statement with ALL relevant context]

Constraints:
- [Hard limits and requirements]
- [Edge cases to handle]

Provide: [Exact output format — e.g., JSON, numbered list, decision + rationale]
```
> Do NOT add "think step by step", "reason through this", or CoT scaffolding.
> These models reason internally; extra instructions waste thinking budget.
>
> **2026 reasoning-effort dials (use the API parameter, not language):**
> - **Claude Opus 4.8:** `thinking: {type: "enabled", budget_tokens: N}` → effort `low`/`high`/`extra`/`max`
> - **GPT-5.6:** `reasoning.effort: "max"` (plus `ultra mode` for subagent-accelerated work)
> - **Gemini 3.5:** adaptive thinking (medium/low/high)
> - **Grok 4.3:** configurable reasoning + non-reasoning mode
> - **DeepSeek V4:** `maximum reasoning effort mode`
>
> **Heuristic:** Dial effort UP for hard problems, async long-running workflows, and deep
> analysis. Dial DOWN for latency/cost-sensitive chat and simple lookups.

### ⑥ Midjourney V8.1 Template (personalization-first)
```
[Subject and action], [environment and context], [specific details to preserve],
[lighting and atmosphere] --s 1000 --p --ar 16:9
```
> **V8.1 shifts (default June 10, 2026):** Personalization profile is the primary creative
> input; the prompt supplies subject/context. Longer prompts now work (40 words > 15 words).
> Push `--stylize` to 1000 with a trained profile. `--cref` / `--oref` / `--q 4` **removed**.
> HD (native 2K) is now default and cheap.

### ⑦ gpt-image-2 5-Slot Template (text-in-image, layouts, UI mockups)
```
Scene:         [where, time of day, background, environment]
Subject:       [who/what is the main focus]
Important details: [materials, clothing, texture, lighting, camera angle, lens feel, composition, mood]
Use case:      [editorial photo / product mockup / poster / UI screen / infographic / concept frame]
Constraints:   [no watermark / no logos / no extra text / preserve specific elements]
```
Mnemonic: **PLACE · FOCUS · FACTS · FORM · CONSTRAINTS.**

### ⑧ Anti-Slop Rule (generalizes across image/video tools)
Replace vague aesthetic words with concrete visual facts:

| Don't say | Say instead |
|---|---|
| "stunning" | "overcast daylight, shallow depth of field" |
| "epic" | "low-angle shot, wide 24mm lens" |
| "cinematic" | "anamorphic 2.39:1, teal-and-orange grade, lens flare" |
| "realistic" | "shot on Canon R5, 85mm f/1.4, natural window light" |
| "high quality" | "8K texture detail, film grain ISO 400" |

Reasoning-based image/video models (gpt-image-2, Nano Banana Pro, Veo 3.1) respond to
**visual parameters, not vibes**.

---

## Step 2.5 — Context Engineering & Harness Engineering (for Agents & Complex Tasks)

> **The 2026 paradigm shift:** Prompt engineering (2022) → Context engineering (2024–25) →
> **Harness engineering** (2026, coined by Andrej Karpathy). The harness = prompt + context
> + tools + memory + runtime. Most agent failures in 2026 are *harness* failures, not
> model or prompt failures. Karpathy joined Anthropic in May 2026 to push this further.

### The Five Context Layers

| Layer | What to Include | When to Omit |
|-------|----------------|-------------|
| System instructions | Role, rules, output format | Never omit |
| Working memory | Current task state, completed steps | Static tasks |
| Retrieved knowledge | Relevant document chunks (RAG) | When model already knows |
| Conversation history | Prior turns (compressed) | Turn 1 |
| Tool outputs | Results from APIs, search, code execution | Non-agentic tasks |

### The Four Canonical Context Strategies (Anthropic, Sept 2025 — still the reference)

These four operations are the consensus vocabulary for context engineering in 2026:

| Strategy | What it means | When |
|----------|---------------|------|
| **Write** | Offload context to external memory/files (the model writes its own scratchpad outside the context window) | Long sessions, large artifacts |
| **Select** | Retrieve only the relevant context for the current step (RAG, vector search) | Knowledge-heavy tasks |
| **Compress** | Summarize long histories, preserving decisions/bugs while discarding redundant tool outputs | Multi-turn agents |
| **Isolate** | Run sub-tasks in sub-agents with their own context windows to prevent pollution | Parallel exploration, deep search |

These map directly to features in production harnesses:
- LangChain **Deep Agents** virtual filesystem = Write + Isolate
- Claude Code's `/compact` = Compress
- Claude **Agent Skills** = Isolate (each skill is a self-contained capability)
- **MCP Tool Search** = Select (dynamic tool loading)

### MCP (Model Context Protocol) — 2026 State

**Governance:** Donated by Anthropic to the **Agentic AI Foundation (AAIF)** under the
Linux Foundation in December 2025, co-founded by Anthropic, OpenAI, and Block. AAIF now
hosts MCP, Goose (Block's agent), and Agents.md.

**Spec:** The **2026-07-28 Specification Release Candidate** (locked May 21, 2026, finalizes
July 28, 2026) is the largest revision since launch. Headline change: **stateless protocol
core** — `initialize` handshake and session id removed, so any MCP request can hit any
server instance (huge for horizontal scaling). Adds an extensions framework, redesigned
Tasks extension, MCP Apps, response caching, and OAuth hardening.

**Companion standard: A2A (Agent2Agent) Protocol.** Initiated by Google Cloud, A2A is
the agent-to-agent counterpart to MCP's agent-to-tool. Surpassed 150 organizations in its
first year; native in Google ADK. Use MCP for tools, A2A for cross-vendor agent teams.

**MCP Tool Search (Jan 14, 2026 — the single most important 2026 tool-calling change):**
Claude dynamically discovers and loads tool definitions on demand instead of loading all
upfront. Triggers when MCP tools would consume **>10% of context**. Critically, **does
not break prompt caching** because deferred tools are excluded from the initial prompt.
Generalized into the Claude Developer Platform's "advanced tool use" / "Tool search tool"
— supports working with hundreds or thousands of tools.

**What to tell users about MCP prompting:**
- MCP exposes **prompt templates** from servers — reusable, parameterized prompts
- When writing a prompt for an MCP-connected agent, reference tools **by name** in the
  runbook and let the harness load schemas dynamically via Tool Search
- Pattern: `"To answer this, use [tool] to look up [data], then synthesize"`
- Don't preload hundreds of tool schemas — let Tool Search handle it

### Harness Engineering Checklist

Beyond the prompt itself, production agents in 2026 need:

1. **Reasoning step before every action** — force a plan/think step before tool calls
2. **Hard cap on tool calls** — explicit per-turn and per-task limits
3. **Tool schemas as public API contracts** — stable names, typed inputs, clear descriptions
4. **Context-first** — present a complete picture of the world to the agent
5. **Explicit exit conditions and failure modes** — define what "done" and "stuck" look like
6. **Runtime engineering over prompt tricks** — versioned, observable runtime configurations
   beat clever one-shot prompt wording for production reliability
7. **Middleware / hook architecture** — guardrails, logging, and approval gates *around*
   the model call (LangChain 1.0 middleware, OpenAI Agents SDK harness), not embedded
   in prose

### Five-Question Context Check

Before finalizing any prompt for an agent, multi-step, or production system, run this
check:

| Question | If "No" → Fix |
|----------|--------------|
| Does the model have all documents/data it needs to answer? | Add RAG / inject documents |
| Is conversation history included if this is multi-turn? | Append history to each call |
| Are tool outputs / prior agent results visible? | Pass tool results in context |
| Is the context window efficiently used (no repetition)? | Compress & deduplicate |
| Is stale or contradictory info excluded? | Filter or timestamp context |

For agentic prompts, also specify:
- **Memory type:** in-context (short tasks) vs. external store (long sessions). Leading
  2026 options: **Mem0** (vectors + knowledge graph), **Letta** (self-editing memory +
  sleep-time compute, formerly MemGPT), **Zep** (temporal knowledge graph), **Redis LangCache**
  (semantic caching for cost reduction).
- **Failure behavior:** "If the answer isn't in the provided context, say so — do not infer"
- **Handoff protocol:** what the agent passes to the next step and in what format

---

## Step 3 — Apply the 22 Power Rules

When writing or reviewing any prompt, run through these:

1. **Be specific** — every vague word is a failure opportunity
2. **Assign a role** — `"You are a senior [expert]"` activates specialized knowledge
3. **Provide context** — what the model doesn't know, it will invent
4. **Give examples** — few-shot is the single highest-ROI technique (+40% accuracy)
5. **Define the output** — specify format, length, structure BEFORE the task
6. **Use delimiters** — separate sections with XML tags, `---`, or `###`
7. **Chain-of-thought (non-reasoning models only)** — ask for reasoning before final answer
8. **Never use CoT on reasoning models** — it wastes their thinking budget; state the problem directly
9. **Constrain negatively** — tell the model what NOT to do
10. **Ground with data** — inject relevant documents to reduce hallucination
11. **Calibrate confidence** — "If confidence < 80%, flag it and suggest verification"
12. **Specify audience** — same content differs for a CEO vs. a developer
13. **Compress** — remove filler; every token must earn its place
14. **Use structured output** — JSON/XML for anything parsed programmatically; use OpenAI `strict: true` for guaranteed JSON Schema conformance
15. **Temperature matters** — 0–0.2 for facts, 0.7–1.0 for creativity
16. **Model-match** — prompting style differs per model (see `references/model-guide.md`)
17. **Iterate** — treat prompts as code: version, test, measure, improve
18. **Security** — assume malicious user input in customer-facing prompts (see Lethal Trifecta, Step 4)
19. **Test edge cases** — adversarial inputs reveal prompt weaknesses
20. **Prompt chain** — break complex tasks into sequential sub-prompts
21. **Document** — add a comment header to every production prompt
22. **Engineer context, not just words** — for production/agentic systems, what's IN the context window matters more than how the instruction is phrased. In 2026, this extends to **engineer the harness**, not just the context — tools, memory, runtime, and middleware all matter

---

## Step 4 — Diagnose Failures

| Failure | Symptom | Fix |
|---------|---------|-----|
| **Vagueness** | Random, generic output | Add specificity, role, constraints |
| **Context starvation** | Wrong assumptions, made-up facts | Supply background knowledge, documents, or history |
| **Hallucination trigger** | Fabricated facts/numbers | Add "Only use provided context; say 'I don't know' if absent" |
| **Format drift** | Inconsistent structure | Add explicit output format with example; use OpenAI `strict: true` |
| **Tone mismatch** | Wrong voice for audience | Use COSTAR; specify audience + tone explicitly |
| **Token overload** | Confused, contradictory output | Split into chained sub-prompts; compress context |
| **Prompt injection** | Malicious input hijacks instructions | Add injection defense (see `references/security.md`); apply Lethal Trifecta / Rule of Two |
| **Reasoning failure** | Wrong answer on logic/math | Add CoT on standard models; OR switch to reasoning model |
| **Reasoning model mismatch** | Good model, poor results on reasoning task | Remove CoT scaffolding; state problem directly and completely; verify effort dial is set high enough |
| **Sycophancy (legacy Claude 4.6)** | Model agrees instead of corrects | Largely resolved in Opus 4.8 (training goal). For 4.6/4.7: add "Be honest. Disagree with me if I'm wrong." |
| **Context window overflow** | Agent loses track mid-task | Summarize history; use external memory; compress prior steps; consider MCP Tool Search to reclaim tool-schema tokens |
| **Multi-agent drift** | Agents produce inconsistent outputs | Add shared state schema; specify handoff format explicitly; use A2A protocol for cross-vendor agents |
| **Tool poisoning (MCP, 2026)** | Malicious MCP tool description manipulates model | Use allow-listed tool schemas; no free-text tool descriptions from untrusted servers; see `references/security.md` |
| **Over-permissioned agent** | Agent has access it doesn't need | Apply least-privilege tool scoping; ~90% of deployed agents are over-permissioned in 2026 |

---

## Step 5 — Present the Improved Prompt

Always deliver:
1. **The polished prompt** — ready to copy-paste, properly formatted for the target tool
2. **What changed and why** — brief explanation of each major change
3. **Framework used** — name the technique/framework and why it fits
4. **Usage tip** — one sentence on how to get the best result
5. **Variation** (optional) — a shorter/longer/different-tone version if useful

---

## Step 6 — Agentic Prompting: The Runbook / Harness Pattern

> **Agent prompts are not chat prompts.** They are *runbooks* — goal-oriented, tool-aware,
> error-handling instruction sets that an autonomous agent executes over multiple steps.
> In 2026, the discipline has shifted from "prompt engineering for agents" to **harness
> engineering** (Karpathy, Feb 2026) — the prompt is one component of a larger harness.
> This is the fastest-growing domain of prompt engineering.

### Anatomy of an Agent Runbook

```
## Goal
[What the agent must achieve — one clear sentence]

## Tools Available
- [tool name]: [what it does, when to use it]
- [tool name]: [what it does, when to use it]

## Workflow
1. [Step 1 — e.g., "Look up order in database"]
2. [Step 2 — e.g., "Analyze the issue"]
3. [Step 3 — e.g., "Draft response"]
4. [Step 4 — e.g., "Escalate if needed"]

## Decision Rules
- If [condition], do [action]
- If [condition], stop and ask for help

## Error Handling
- If [tool call fails], try [fallback]
- If [data not found], say [message] and stop

## Stopping Conditions
- [condition where task is complete and agent should return final answer]
- [condition where agent must hand off to human]
```

### Agent Prompt vs. Chat Prompt

| Dimension | Chat Prompt | Agent Runbook |
|-----------|-------------|---------------|
| **Goal** | Answer a question | Complete a multi-step task |
| **Tools** | Implicit / none | Explicitly defined with schemas |
| **State** | Single turn | Maintained across steps |
| **Error handling** | Vague ("be careful") | Explicit fallbacks |
| **Stopping** | After response | Defined conditions |
| **Cost of failure** | Low (re-ask) | High (wrong action with side effects) |

### Multi-Agent Orchestration Patterns (2026)

The original six patterns (sequential, router, parallel, hierarchical, debate, dynamic
handoff) are still taught, but 2026 production thinking has consolidated around **five
dominant patterns** (per Digital Applied / Beam AI practitioner roundups):

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

**Cross-vendor agent teams:** Use the **A2A (Agent2Agent) protocol** (Google Cloud / Linux
Foundation; 150+ organizations in year one) for inter-agent communication across vendors.
Pair with MCP for agent-to-tool. Google ADK has native A2A support built in.

**Major 2026 frameworks:** LangChain/LangGraph 1.0 (Agent Middleware + Deep Agents),
CrewAI v1.10, Microsoft Agent Framework (replaced AutoGen, Oct 2025), OpenAI Agents SDK
(April 15, 2026 update adds native sandbox + model-native harness), Anthropic Claude Agent
SDK + Agent Skills, Google ADK, Mastra (TypeScript-first, v1.0 Jan 2026).

**Subagent orchestration is the new default.** Claude Opus 4.8's **dynamic workflows**
(research preview) can spawn hundreds of parallel subagents in one session for codebase-
scale migrations. GPT-5.6's **ultra mode** uses subagents to accelerate complex work.
Kimi K2.6's **Agent Swarm** scales to 300 sub-agents / 4,000 coordinated steps.

### Self-Correction Pattern (2026 Standard)

The most common chaining pattern in 2026 is **self-correction**:

```
Step 1: Generate initial output
Step 2: Review output against criteria
Step 3: Refine based on review
```

This is built into Claude Opus 4.8's adaptive thinking by default. Use explicit
self-correction when you need to inspect or log intermediate states.

### Agent Skills Pattern (Anthropic, 2026)

**Agent Skills** are modular, discoverable capability packages Claude can invoke on demand.
Supported across Claude.ai, Claude Code, the Claude Agent SDK, and the Claude Developer
Platform. A skill bundles: instructions (the prompt), optional scripts, optional reference
docs, and optional assets. Public skills repo: https://github.com/anthropics/skills

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

The skill ecosystem is now a competitive marketplace: Anthropic Claude Skills, Vercel
skills.sh, OpenAI Codex plugins, Cline plugins, MCP server directory. Treat skills as a
first-class axis alongside MCP (tools) and A2A (agents).

---

## Step 7 — Video & Image Generation Prompting (Late June 2026)

### Video Generation

> **🚨 Sora is discontinued.** OpenAI shut the Sora web/app on **April 26, 2026**; the
> Sora API sunsets **September 24, 2026**. There is no Sora 3. Migrate Sora prompts to
> Veo 3.1, Kling 3.0, Seedance 2.0, or Grok Imagine 1.5.

The current leaderboard (late June 2026):
| Tool | Strength | Best For |
|------|----------|----------|
| **Veo 3.1** (Google) | 4K + native synchronized audio, cinematic physics | Commercial video, dialogue |
| **Gemini Omni Flash** (Google) | Any-input → video world model; voice editing | Conversational video creation |
| **Kling 3.0** (Kuaishou) | Native 4K at 60fps, multi-shot storyboarding | High-res productions, multilingual |
| **Seedance 2.0** (ByteDance) | Physical realism, bigger scenes | Action / motion-heavy scenes |
| **Runway Gen-4.5** ("David") | Image-to-video with reference stills, camera control | Reference-driven consistent video |
| **Grok Imagine 1.5** (xAI Aurora) | Native audio, image-to-video, "Spicy Mode" | Less-censored creative video |
| **Wan 2.6** (Alibaba) | Open-source, strong prompt adherence | Self-hosted / open-source video |
| **Hailuo 2.3** (MiniMax) | Available alongside Seedance on Hailuo platform | General-purpose video |

**Not yet released (verify before recommending):** Veo 4, Kling 4, Runway Gen-5.

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
| **Veo 3.1** | **100–150 words is the sweet spot.** Specify camera movement and shot type in every prompt. Front-load priority (camera → subject → action → setting → style → audio). Avoid contradictory camera moves ("pan while zooming"). Use labeled modular format for complex scenes. |
| **Gemini Omni** | Takes any reference (image/text/video/audio) → video. Conversational/voice editing — change characters/background by voice. SynthID-watermarked. |
| **Kling 3.0** | Native 4K 60fps. Multi-shot storyboarding + reference images for consistency. Multilingual prompts supported. |
| **Seedance 2.0** | For lip-sync, provide exact dialogue text in the prompt. Strong motion coherence. |
| **Runway Gen-4.5** | Image-to-video with reference stills. Camera-control parameters. Concise scene + motion descriptions. |
| **Grok Imagine 1.5** | Image-to-video at up to 720p. Describe camera movements, pacing, atmosphere. "Spicy Mode" = less-censored generation. |
| **Wan 2.6** | Open-source. 1080p text-to-video and image-to-video. Strong prompt adherence across photorealistic and artistic styles. |

### Image Generation (Late June 2026)

| Tool | Best For |
|------|----------|
| **Midjourney V8.1** (default June 10, 2026) | Aesthetic/illustrative images; personalization-driven |
| **gpt-image-2** (Apr 21, 2026, ChatGPT Images 2.0) | Text-in-image, layouts, UI mockups, infographics |
| **Imagen 4 Ultra** (Google) | Highest-fidelity photorealistic stills (API) |
| **FLUX.2 [max] / [pro] / [klein]** (Black Forest Labs) | Photorealism; `klein` = sub-second on modern GPUs (Apache 2.0, 4B) |
| **Nano Banana 2 / Pro** (Gemini-native) | Conversational image editing |
| **Recraft V4.1 Vector** | Editable SVG vectors, brand assets |
| **Ideogram 3.0** | Legible text-in-image, typographic designs |

**Midjourney V8.1 prompting shifts (major):**
- Personalization profile is now the primary creative input — the prompt supplies subject/context, your rated-aesthetic profile supplies look. Activate at 40 ratings, stabilize at ~200, improve to ~2,000.
- **Longer prompts now work** (reverses V7's "concise is better"). Trend toward 40-word prompts.
- Push `--stylize` to **1000** with a trained profile.
- `--cref` / `--oref` / `--q 4` **removed** in V8.1. Use Personalization + srefs for character/object consistency.
- HD (native 2K) is now default and cheap.

---

## Step 8 — PromptOps: Evaluation, Testing & Governance

In 2026, treating prompts as code that must be tested, versioned, and monitored is
table stakes for production systems.

### Evaluation Frameworks (Late June 2026)

| Framework | Best For |
|-----------|----------|
| **Promptfoo** | Red-teaming (50+ attack plugins), multi-model A/B comparison, YAML-driven test matrices, CI/CD; now supports automated red-teaming for agents & RAGs, HarmBench integration |
| **DeepEval** (Confident AI) | Unit-testing prompts as Python code, CI/CD integration |
| **RAGAS** | RAG pipeline quality (retrieval + generation accuracy) |
| **LangSmith** / **LangSmith Fleet** | Full observability + eval for LangChain/LangGraph agents (formerly Agent Builder) |
| **Braintrust / Helicone / Arize Phoenix** | Production observability + experiment tracking |
| **Arthur Bench** | Open-source eval for comparing LLMs, prompts, hyperparameters |

### Agent Benchmarks (2026 — Harder, More Realistic)

The 2026 theme: benchmarks closer to production reality produce dramatically lower scores
than SWE-Bench Verified. The field is regrounding in harder, more realistic evals.

| Benchmark | What it measures |
|-----------|------------------|
| **τ-bench (tau-bench)** (Sierra) | Real-world tool-using agents; checks database state post-task. τ2 adds `pass^k` (reliable success across repeated runs). Top-model scores: 50–70% — exposes how far "production-ready" still is. Reports interaction cost. |
| **SWE-Bench Verified** (500-instance) | Coding default, increasingly criticized as over-fit |
| **SWE-Bench Pro** (1,865 problems) | Harder variant of SWE-Bench |
| **Terminal-Bench** (arXiv 2601.11868) | Hard, realistic terminal tasks; built on the Harbor framework |
| **AppWorld** | Cited alongside τ-bench and SWE-Bench |
| **MCP-SafetyBench** | LLM-agent safety across 5 domains and 20 attack types on real-world MCP servers |
| **NRT-Bench** (arXiv 2606.20408, June 2026) | Multi-turn red-teaming, fixed-judge replay pipeline |

### Automated Prompt Optimization

| Tool | Approach |
|------|----------|
| **DSPy** (Stanford NLP) | Treat prompts as parameters to optimize given training set + metric |
| **TextGrad** | Gradient-based prompt optimization |
| **PromptBreeder / EvoPrompt** | Evolutionary prompt search |
| **LLM-as-judge** | Use a strong LLM to grade outputs; mature pattern via DeepEval/Promptfoo |

### Prompt Testing Checklist

Before deploying any production prompt:
- [ ] **Regression suite:** Run eval across all known test cases
- [ ] **Multi-model test:** Does it work on Claude AND GPT AND Gemini AND open-weights (GLM-5.2, DeepSeek V4)?
- [ ] **Edge cases:** Empty input, very long input, adversarial input, multi-language evasion
- [ ] **Format drift detection:** Does structured output always parse correctly? (Use OpenAI strict mode / `strict: true` for guaranteed JSON Schema conformance.)
- [ ] **Cost/quality tradeoff:** Does a shorter prompt perform as well?
- [ ] **Model-fit test:** Re-validate when the underlying model updates (model drift)
- [ ] **Red-team:** Garak / PyRIT / Promptfoo red-team against OWASP LLM Top 10 (2025) + OWASP ASI Top 10 for Agentic (2026)
- [ ] **MCP-specific:** If using MCP servers, run MCP-SafetyBench against the tool surface

### Governance Patterns

- **Prompt versioning** — track changes; every prompt has a version hash
- **Eval-as-code** — eval suites in CI/CD (Promptfoo YAML or DeepEval pytest)
- **Regression cron** — scheduled eval runs to catch model drift
- **Prompt libraries** — catalog with owner, version, eval coverage, last validated model
- **System cards / model cards** — treated as compliance artifacts under EU AI Act (full enforcement **August 2, 2026** for GPAI providers)
- **Agent governance** — Microsoft Agent Governance Toolkit maps controls to OWASP ASI Top 10

---

## Reference Files

- `references/model-guide.md` — Model-specific prompting for Claude Opus 4.8 (and Fable 5 / Mythos 5), GPT-5.6 (Sol/Terra/Luna), Gemini 3.5 (Flash + Pro), Grok 4.3, DeepSeek V4, GLM-5.2, Qwen3.7-Max, MiniMax M3, Meta Muse Spark, image models (Midjourney V8.1, gpt-image-2, Imagen 4, FLUX.2, Recraft V4.1, Ideogram 3.0), video models (Veo 3.1, Gemini Omni, Kling 3.0, Seedance 2.0, Runway Gen-4.5, Grok Imagine 1.5, Wan 2.6), coding agents (Claude Code/Agent SDK, Cursor, Copilot), MCP under AAIF, Context/Harness Engineering
- `references/techniques.md` — Deep-dive on each technique including COSTAR, RISEN, CoV, Multi-Agent Debate, DSPy, Context Engineering (Write/Select/Compress/Isolate), Harness Engineering, Agent Skills, MCP Tool Search, A2A protocol, agentic runbooks
- `references/templates-library.md` — 30+ ready-to-use prompt templates by industry/use-case
- `references/security.md` — Prompt injection defense, OWASP LLM Top 10 (2025) + OWASP ASI Top 10 for Agentic (2026), Lethal Trifecta / Rule of Two, MCP attack classes, EU AI Act compliance
- `references/update-methodology.md` — How this skill is kept current (last updated late-June 2026)

---

## Quick Examples

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
Set `thinking: {type: "enabled", budget_tokens: 8000, effort: "extra"}` (Claude) or
`reasoning: {effort: "max"}` (GPT-5.6) via the API parameter — do NOT put CoT instructions
in the prompt itself.

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
→ All three present — apply strict instruction/data separation, allow-listed tool
   schemas, and human-in-the-loop approval for refunds > $200.
```
