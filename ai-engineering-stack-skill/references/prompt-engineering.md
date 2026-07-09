# Prompt Engineering: A Deep Research Brief

*A practitioner-first, academically-rigorous survey of the foundational layer of the AI agent stack — the discipline of crafting instructions that produce exceptional output from any frontier model, with full coverage of the late-June 2026 model landscape and the techniques that survive the reasoning-model era.*

| Field | Value |
|---|---|
| **Paradigm coinage date** | 2022 (emerged with the first broadly-useful instruction-tuned LLMs — InstructGPT, GPT-3.5, the original ChatGPT) |
| **Coined by** | Practitioner community; crystallized by OpenAI's prompt engineering guide (late 2022) and Anthropic's prompt design documentation (early 2023). No single coiner; the discipline accreted around shared practice. |
| **Layer below** | None — prompt engineering is the foundational layer of the four-layer stack |
| **Layer above** | Context engineering (2024–25) |
| **Status as of July 2026** | Table stakes — every production system requires it. Not obsolete; the lower layers of the stack automate *invocation* of well-engineered prompts but never eliminate the need for the prompt itself. Transformed by reasoning models (Claude Opus 4.8, GPT-5.6 max reasoning, Gemini 3.5 Deep Think, Grok 4.3 reasoning, DeepSeek V4) which eliminate classic CoT scaffolding. |

---

## 2. TL;DR

Prompt engineering is the discipline of crafting the instruction given to an LLM so that the model's output matches the user's intent. It is the foundational layer of the four-layer AI engineering stack — prompt (2022) → context (2024–25) → harness (Feb 2026) → loop (June 2026) — and remains necessary in every production system; the higher layers automate the *invocation* of well-engineered prompts but cannot eliminate the prompt itself. The discipline's central insight is that LLMs are extraordinarily literal and context-sensitive: vague instructions produce vague output, missing context produces hallucinations, and unstated constraints produce surprising results. The fix is the same set of techniques that have been refined since 2022 — assign a role, give examples (few-shot), specify the output format, constrain negatively, ground with data — augmented in 2026 by reasoning-model-specific patterns that eliminate classic chain-of-thought scaffolding.

The technique landscape in 2026 has two tiers. The **classic techniques** — Zero-Shot, Few-Shot, Chain-of-Thought (CoT), Tree-of-Thought (ToT), ReAct, Chain-of-Verification (CoV), Skeleton-of-Thought, Self-Consistency, COSTAR, RISEN, Multi-Agent Debate, DSPy — remain the consensus vocabulary for standard models and for teaching. The **reasoning-model techniques** — Claude Opus 4.8 with `thinking: {type: "enabled", effort: "extra"}`, GPT-5.6 with `reasoning.effort: "max"`, Gemini 3.5 adaptive thinking, Grok 4.3 reasoning mode, DeepSeek V4 `maximum reasoning effort mode` — eliminate CoT scaffolding entirely; the model reasons internally and extra "think step by step" instructions waste thinking budget. The frontier of the discipline is in model-specific prompt shapes (Claude's contract-style XML, GPT's markdown headers, GLM-5.2's system/user split with native Agent Mode), Anthropic Agent Skills, MCP prompt templates, and PromptOps (Promptfoo, DeepEval, Giskard, RAGAS) that treat prompts as versioned, tested, monitored code.

Prompt engineering is not obsolete — it is the foundation every higher layer assumes. A loop with a broken prompt fails on every iteration; a harness with a vague prompt produces a confidently-wrong agent; a context-engineered window with a vague prompt produces generic output. The frontier of the discipline in 2026 is in reasoning-model prompting, model-specific shapes, and PromptOps; the foundations have not changed since 2022 and are unlikely to.

---

## 3. The Four-Layer Stack

Prompt engineering sits at the bottom of the four-layer AI engineering stack. The stack is cumulative: each layer was invented to solve a failure class the layer below could not, and each remains necessary in 2026 production systems. This section briefly locates prompt engineering in the stack; the full evolution is documented in the Loop Engineering brief at `/home/z/my-project/download/loop-engineering-deep-research.md` Section 3 and the Harness Engineering brief Section 3.

### 3.1 Where prompt engineering sits

Prompt engineering is the layer closest to the model. It asks: "given an LLM call with a fixed context window and a fixed set of tools, what instruction produces the output I want?" The techniques — role assignment, few-shot examples, output format specification, negative constraints, chain-of-thought scaffolding (for standard models) — operate entirely within the prompt itself. They do not modify the context, the tools, the runtime, or the loop; they assume all of those are fixed and ask how to phrase the instruction given those constraints.

### 3.2 What prompt engineering solves

Prompt engineering solves the failure class of **vagueness**. An LLM given a vague instruction ("write a blog post about AI") produces vague output (a meandering essay with no audience, no length, no tone) because the model has no information to disambiguate the request. The techniques — assign a role, give examples, specify the format, constrain negatively — supply the disambiguating information. The result is output that matches the user's intent on the first try, without iteration.

### 3.3 What prompt engineering does NOT solve

Prompt engineering does not solve **context starvation** (the model lacks the document it needs to answer — that's context engineering's Write/Select), **single-run fragility** (the agent has the wrong tools or no exit conditions — that's harness engineering), or **multi-run fleet failures** (verification gaming, runaway loops — that's loop engineering). A perfectly-engineered prompt fails if the context is wrong, the harness is broken, or the loop has no verification. This is why the four layers stack rather than replace.

### 3.4 Why prompt engineering is not obsolete

The naive reading of the four-layer stack — that each new layer obsoletes the one below — is wrong. The higher layers *automate the invocation* of well-engineered lower layers; they do not eliminate the lower layers. A loop invokes a harness, which assembles a context, which contains a prompt. If the prompt is vague, the loop's iterations will all produce vague output; if the harness's tools are wrong, the loop's iterations will all use the wrong tools; if the loop's verification is broken, the loop will silently fail. The four-layer stack is a dependency chain: each layer assumes the layers below are well-engineered. Prompt engineering is the foundation, and the foundation must be solid.

### 3.5 The 2026 transformation: reasoning models

The single biggest change to prompt engineering in 2026 is the maturation of **reasoning models** — Claude Opus 4.8 (May 28, 2026, 1M context), GPT-5.6 Sol/Terra/Luna (June 26, 2026, ~1.5M context), Gemini 3.5 Pro/Flash (May 19, 2026, 1M context, with Deep Think), Grok 4.3 (April 30, 2026, 1M context, configurable reasoning), DeepSeek V4 (April 24, 2026, ~1M context, `maximum reasoning effort mode`), GLM-5.2 (June 13, 2026, 1M context, MIT-licensed open weights, native Agent Mode), and Qwen3.7-Max (May 19, 2026, 1M context). These models reason internally — they allocate their own thinking budget via API parameters (`thinking: {effort: "extra"}` for Claude, `reasoning.effort: "max"}` for GPT, adaptive thinking for Gemini) rather than requiring external CoT scaffolding. The implication for prompt engineering is stark: classic CoT instructions ("think step by step", "reason through this carefully") are counterproductive on reasoning models — they waste thinking budget and produce worse output. The reasoning-model prompt is shorter, more direct, and states the problem completely, letting the model allocate its own reasoning.

### 3.6 The 2026 transformation: model-specific shapes

The second change is the divergence of prompt shapes across models. Claude Opus 4.8 follows contract-style XML (`<role>`, `<context>`, `<task>`, `<constraints>`) literally — more literally than 4.7, with ~4× less likely to let flaws in its own code pass unremarked. GPT-5.6 prefers clean markdown headers with the task stated first, plus explicit cache breakpoints for long-running agent sessions. GLM-5.2 works best with a ChatGPT-style system/user split and recognizes coding-agent idioms (.cursorrules-style, CLAUDE.md-style). Gemini 3.5 prefers concise instructions and benefits from "Be concise." prepended. The implication: a prompt that works on one frontier model may underperform on another. Production prompt engineering in 2026 is model-specific, with the prompt shape tuned to the target model.

---

## 4. Definition & First Principles

### 4.1 Definition

Prompt engineering is the discipline of crafting the instruction given to an LLM so that the model's output matches the user's intent. It includes the choice of technique (Zero-Shot, Few-Shot, CoT, ReAct, COSTAR, RISEN, etc.), the structure of the prompt (role, context, task, format, tone, examples, constraints, evaluation), the choice of model and its reasoning configuration, and the iterative refinement of the prompt against observed output. The output of prompt engineering is a prompt — a piece of text (or structured instruction) that, given to a specific model with a specific configuration, reliably produces output that meets a defined success criterion.

### 4.2 The five first principles

**First principle: every vague word is a failure opportunity.** LLMs are extraordinarily literal. A word like "good" or "professional" or "comprehensive" means whatever the model's training distribution suggests it means, which is rarely what the user meant. The fix is specificity: replace "write a good blog post" with "write a 600-word HBR-register opinion piece for a C-suite reader, no bullet lists, three real company examples, ending with a one-sentence CTA." The more specific the prompt, the smaller the space of plausible outputs, and the more likely the first attempt matches intent.

**Second principle: assign a role to activate specialized knowledge.** The instruction "You are a senior financial analyst with 15 years of experience in fintech valuations" activates the model's fintech-valuation knowledge in a way that the bare instruction "value this fintech" does not. The role is not a decorative framing device; it is a retrieval cue that biases the model toward the relevant region of its training distribution. The role should be specific (not "you are an expert" but "you are a senior engineer familiar with TypeScript/React monorepos") and relevant to the task.

**Third principle: examples are the single highest-ROI addition.** Few-shot — providing 3–5 concrete input → output demonstrations — typically improves accuracy by ~40% across task types. The model uses the examples to infer the format, the tone, the level of detail, and the implicit constraints that the user did not state. Examples are more effective than instructions for conveying format ("here are three examples of the JSON I want" beats "return JSON in this format: ..."). The examples should cover the variation in the input space — at least one easy case, one hard case, one edge case.

**Fourth principle: define the output before the task.** Specify the format, length, structure, and tone before stating the task. A prompt that says "write a blog post about X" produces a different shape of output than one that says "600-word HBR-register opinion piece, three paragraphs, no bullet lists, ending with a CTA. Topic: X." The output specification constrains the model's generation in a way that the task statement alone does not.

**Fifth principle: constrain negatively.** Tell the model what NOT to do. "Do not use the word 'delve'. Do not use bullet lists. Do not promise delivery dates you cannot verify." Negative constraints close off common failure modes that positive instructions alone cannot. LLMs have default behaviors (the "delve" tic, the bullet-list default, the over-confident prediction) that negative constraints explicitly suppress.

### 4.3 The reasoning-model caveat

The five first principles hold for all models, but the *techniques* derived from them diverge for reasoning models. Specifically, the CoT technique ("think step by step", "reason through this carefully before answering") — derived from the third principle's emphasis on examples and the first principle's emphasis on specificity — is **counterproductive on reasoning models**. Reasoning models (Claude Opus 4.8 with effort set, GPT-5.6 with max reasoning, Gemini 3.5 Deep Think, Grok 4.3 reasoning, DeepSeek V4 maximum reasoning) allocate their own thinking budget via API parameters; external CoT instructions consume thinking budget that the model would otherwise allocate to its internal reasoning, producing worse output. The reasoning-model prompt states the problem completely, specifies the output, and lets the model reason. The first principles (specificity, role, examples, output definition, negative constraints) still apply; the CoT scaffolding does not.

### 4.4 The 22 power rules (summary)

The full prompt-engineering discipline can be expressed as 22 power rules, derived from the first principles and refined across thousands of production deployments. The rules are: (1) be specific; (2) assign a role; (3) provide context; (4) give examples; (5) define the output; (6) use delimiters; (7) chain-of-thought (non-reasoning models only); (8) never use CoT on reasoning models; (9) constrain negatively; (10) ground with data; (11) calibrate confidence; (12) specify audience; (13) compress — every token must earn its place; (14) use structured output (JSON/XML, OpenAI `strict: true`); (15) temperature matters (0–0.2 for facts, 0.7–1.0 for creativity); (16) model-match — prompting style differs per model; (17) iterate — treat prompts as code; (18) security — assume malicious input in customer-facing prompts; (19) test edge cases; (20) prompt chain — break complex tasks into sequential sub-prompts; (21) document — add a comment header to every production prompt; (22) engineer context, not just words — for production systems, what's IN the context window matters more than how the instruction is phrased.

The 22 rules are the practitioner's checklist; every prompt should be auditable against them. Rule 22 — "engineer context, not just words" — is the bridge to context engineering: it acknowledges that for production and agentic systems, the prompt is one component of a larger context window, and the curation of that window (the Write/Select/Compress/Isolate operations of context engineering) matters as much as the prompt itself.

---

## 5. The Technique Landscape (2026)

Prompt engineering has accreted a substantial technique catalog since 2022. The catalog divides into classic techniques (for standard models and for teaching), reasoning-model techniques (which eliminate classic CoT), and frameworks (which structure the prompt rather than specifying the reasoning path). This section walks the catalog with when-to-use, when-not-to-use, and the characteristic failure mode for each.

### 5.1 Classic techniques (standard models)

| Technique | When to use | When NOT to use | Characteristic failure |
|---|---|---|---|
| **Zero-Shot** | Simple, one-shot tasks; baseline | Anything requiring format consistency or multi-step reasoning | Generic output; format drift |
| **Few-Shot** (3–5 examples) | Classification, extraction, formatting; highest-ROI technique | When examples would overfit to a narrow pattern | Overfitting; example selection bias |
| **Chain-of-Thought (CoT)** | Math, logic, multi-step reasoning on standard models | Reasoning models (counterproductive); trivial tasks | Reasoning chains that look correct but contain errors |
| **Tree-of-Thought (ToT)** | Complex problems requiring exploration of multiple paths | Problems with a single correct path | Expensive (many model calls); branches that never converge |
| **ReAct** (Reason + Act) | Agents with tool access | Non-agentic tasks; reasoning models with native tool use | Tool-call loops without progress |
| **Chain-of-Verification (CoV)** | Fact verification; reducing hallucination | Creative tasks; tasks where verification is impossible | Verification that confirms the original (uncorrelated) error |
| **Self-Consistency** | Subjective tasks with uncertain confidence | Deterministic tasks; cost-sensitive tasks | Expensive (multi-sample + vote); variance in vote |
| **Skeleton-of-Thought** | Long-form content | Short-form content | Outlines that constrain the content too tightly |
| **Multi-Agent Debate** | High-stakes decisions; bias reduction | Time-sensitive tasks | Convergence to a confident wrong answer; cost |

### 5.2 Frameworks (prompt structures)

**COSTAR** (Context, Objective, Style, Tone, Audience, Response) — best for content and communication tasks. The six components force the prompt writer to specify the situation, the goal, the writing structure, the emotional register, the reader, and the output format. Particularly effective for branded content, customer communications, and marketing copy.

**RISEN** (Role, Instructions, Steps, End goal, Narrowing) — best for procedural and task-oriented prompts. The five components force the prompt writer to specify the expert persona, the action verbs, the numbered sequence, the success criterion, and the scope constraints. Particularly effective for agents, tutoring systems, and workflow automation.

**Anatomy Template** (Role, Context, Task, Format, Tone, Examples, Constraints, Evaluation) — the general-purpose eight-layer template. Use only what the task needs; don't pad. Every world-class prompt has some subset of these layers; the discipline is in choosing the right subset.

### 5.3 The reasoning-model fast-path

Reasoning models — Claude Opus 4.8 with effort set, GPT-5.6 with max reasoning, Gemini 3.5 Deep Think, Grok 4.3 reasoning, DeepSeek V4 maximum reasoning — eliminate the need for external CoT scaffolding. The reasoning-model prompt has a different shape:

```
[Complete problem statement with ALL relevant context]

Constraints:
- [Hard limits and requirements]
- [Edge cases to handle]

Provide: [Exact output format — e.g., JSON, numbered list, decision + rationale]
```

Do NOT add "think step by step", "reason through this", or CoT scaffolding. These models reason internally; extra instructions waste thinking budget. The 2026 reasoning-effort dials (set via the API parameter, not language) are: Claude Opus 4.8 `thinking: {type: "enabled", budget_tokens: N}` → effort `low`/`high`/`extra`/`max`; GPT-5.6 `reasoning.effort: "max"` (plus `ultra mode` for subagent-accelerated work); Gemini 3.5 adaptive thinking (medium/low/high); Grok 4.3 configurable reasoning + non-reasoning mode; DeepSeek V4 `maximum reasoning effort mode`. The heuristic: dial effort UP for hard problems, async long-running workflows, and deep analysis; dial DOWN for latency/cost-sensitive chat and simple lookups.

### 5.4 Automated prompt optimization

| Tool | Approach | When to use |
|---|---|---|
| **DSPy** (Stanford NLP) | Treat prompts as parameters to optimize given training set + metric | Production systems with a training set and a clear metric |
| **TextGrad** | Gradient-based prompt optimization | When DSPy's discrete search is too coarse |
| **PromptBreeder / EvoPrompt** | Evolutionary prompt search | Exploratory; when the search space is large |
| **LLM-as-judge** | Use a strong LLM to grade outputs | Mature pattern via DeepEval/Promptfoo; cheap and effective |

Automated prompt optimization is the production-grade answer to "how do I know my prompt is good?" — it treats the prompt as a parameter to be optimized against a metric, with the optimization done by an LLM (LLM-as-judge) or by gradient-based methods (TextGrad). The frontier of the discipline in 2026 is in this direction; manual prompt iteration remains useful for exploration but is insufficient for production.

### 5.5 Deprecated / superseded techniques (2026)

The technique catalog has deprecated entries. "Be honest" coaxing (Claude 4.6 sycophancy) is deprecated — anti-sycophancy is now a training goal in Opus 4.8. "Think step by step" on reasoning models is counterproductive. Midjourney `--cref` / `--oref` are removed in V8.1. Sora prompting guides are discontinued (Sora shut down April 26, 2026). AutoGen (Microsoft) is in maintenance mode, replaced by Microsoft Agent Framework. LangChain "Agent Builder" is rebranded as LangSmith Fleet. Claude Code SDK is renamed Claude Agent SDK. Llama as Meta flagship is superseded by Muse Spark. The six multi-agent patterns (strict) are consolidated into five dominant production patterns. Imagen 4 is end-of-life (shuts down August 17, 2026). Ideogram 3.0 (closed) is superseded by Ideogram 4.0 (open-weight 9.3B, June 3, 2026). Stable Diffusion 3.5 is displaced by FLUX.2 family + Ideogram 4.0. The practitioner should periodically audit their technique catalog against the current state; techniques that worked in 2024 may be counterproductive in 2026.

---

## 6. The Anatomy Template — Eight Layers

Every world-class prompt has some subset of eight layers. The discipline is in choosing the right subset for the task; padding a prompt with unused layers dilutes the signal. This section walks each layer with when to include it, when to omit it, and a worked example.

### 6.1 The eight layers

```
[ROLE]        Who the AI is / what expertise it holds
[CONTEXT]     Background knowledge, constraints, scenario
[TASK]        The specific action (use strong verbs: analyze, generate, compare, extract)
[FORMAT]      Output structure: JSON, table, bullet list, numbered steps, paragraphs
[TONE]        Formal / casual / technical / empathetic / concise
[EXAMPLES]    1–5 concrete input → output demonstrations (highest-ROI addition)
[CONSTRAINTS] Length limits, things to avoid, scope boundaries
[EVALUATION]  Success criteria or self-check rubric
```

### 6.2 Layer-by-layer guidance

**ROLE.** Include for any task where expertise matters. Be specific: not "you are an expert" but "you are a senior engineer familiar with TypeScript/React monorepos, with experience in performance optimization." Omit for trivial tasks where the role adds no information.

**CONTEXT.** Include whenever the model lacks the background to answer. Paste documents, data, or prior conversation. For long documents: source material at the TOP, question/instructions at the BOTTOM. Omit when the model already knows (general knowledge questions).

**TASK.** Always include. One sentence, strong verb. "Analyze the Q3 revenue trend and identify the top three growth drivers." Not "let's think about Q3 revenue."

**FORMAT.** Include for anything parsed programmatically or anything where shape matters. Specify JSON schema, table columns, numbered list, paragraph count. Use OpenAI `strict: true` for guaranteed JSON Schema conformance. Omit for free-form creative tasks where the format is the model's to choose.

**TONE.** Include when the audience matters. "Authoritative but accessible — Harvard Business Review register." Omit for internal/technical tasks where tone is irrelevant.

**EXAMPLES.** Include whenever format or approach matters. 3–5 examples covering the variation in the input space. This is the highest-ROI addition; few-shot typically improves accuracy by ~40%. Omit for tasks where the format is fully specified by FORMAT.

**CONSTRAINTS.** Include for any production prompt. Length limits, things to avoid, scope boundaries. "Under 300 words. Do not use 'delve'. Do not promise delivery dates." Negative constraints close off common failure modes.

**EVALUATION.** Include for production prompts where the model should self-check. "Before returning, verify that (a) the JSON parses, (b) every required field is present, (c) no field is null." Omit for one-shot tasks where the human will review.

### 6.3 The Claude Opus 4.8 contract-style template

Claude Opus 4.8 follows instructions even more literally than 4.7 and is ~4× less likely to let flaws in its own code pass unremarked. Don't coax or flatter — write a precise contract:

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

### 6.4 The ChatGPT / OpenAI template (system + user split)

```
SYSTEM:
You are a [role]. [Persistent rules, persona, output format defaults].

USER:
Context: [background]
Task: [what to do — use action verb]
Format: [how to structure the answer]
Constraints: [limits and exclusions]
```

For GPT-5.6: add explicit cache breakpoints for long-running agent sessions (cache writes billed at 1.25× uncached input rate; cache reads get 90% discount; 30-minute minimum cache life). Use `strict: true` on function/tool definitions for guaranteed JSON Schema conformance.

### 6.5 The COSTAR framework (content/communication tasks)

```
Context: [Relevant situation, background, or prior events]
Objective: [Specific goal — what the output must achieve]
Style: [Writing structure: listicle, narrative, technical doc, etc.]
Tone: [Emotional register: authoritative, friendly, urgent, empathetic]
Audience: [Who will read this — expertise level, role, mindset]
Response: [Exact format and length expected]
```

### 6.6 The RISEN framework (procedural / task-oriented)

```
Role: [Expert persona]
Instructions: [What to do — clear action verbs]
Steps: [Numbered sequence the AI must follow]
End goal: [Definition of a successful output]
Narrowing: [Scope constraints, exclusions, format]
```

### 6.7 The reasoning-model template

```
[Complete problem statement with ALL relevant context]

Constraints:
- [Hard limits and requirements]
- [Edge cases to handle]

Provide: [Exact output format — e.g., JSON, numbered list, decision + rationale]
```

No CoT scaffolding. State the problem completely; let the model reason.

---

## 7. Model-Specific Prompting (Late-June 2026)

The 2026 frontier-model landscape requires model-specific prompting. A prompt that works on Claude Opus 4.8 may underperform on GPT-5.6, GLM-5.2, or Gemini 3.5. This section walks the current frontier models with their prompting specifics. Cross-reference the full model guide in the `references/model-guide.md` of the source prompt-engineer skill.

### 7.1 Claude Opus 4.8 (Anthropic, May 28, 2026)

Specs: 1M context by default, dynamic workflows (research preview — can spawn hundreds of parallel subagents), effort control via `thinking: {type: "enabled", budget_tokens: N}` → `low`/`high` (default)/`extra`/`max`. Anti-sycophancy is now a training goal — explicitly invite the model to flag uncertainties. Use contract-style XML prompts (Section 6.3). Use Agent Skills for reusable capability packaging. MCP Tool Search now available in Claude Code (not just Developer Platform). System instructions can be updated mid-task without breaking the prompt cache. Minimum cacheable prompt length lowered to 1,024 tokens.

### 7.2 GPT-5.6 Sol / Terra / Luna (OpenAI, June 26, 2026)

Specs: ~1.5M context (limited preview), three tiers (Sol = flagship, Terra = balanced ~2× cheaper than GPT-5.5, Luna = fastest/cheapest). Use clean markdown headers; state task first. Use `max reasoning effort` for hardest tasks and `ultra mode` for subagent-accelerated complex work. Use explicit cache breakpoints (30-minute minimum cache life; cache reads 90% discount). Use `strict: true` on function/tool definitions for guaranteed JSON Schema conformance.

### 7.3 Gemini 3.5 Flash + Pro (Google, May 19, 2026)

Specs: 1M context, GA Flash; Pro announced but slipped to July 2026 (will bring 2M context + new Deep Think). Add "Be concise." Use adaptive thinking (medium/low/high). Native multimodal. Computer Use tool (June 24, 2026) for browser/mobile/desktop with prompt-injection detection.

### 7.4 GLM-5.2 (Z.ai / Zhipu, June 13, 2026)

Specs: 753B MoE (40B active), 1M context, MIT-licensed open weights, coding-first/agent-first. Independent reporting: "beats GPT-5.5 at ~1/6 the cost." Usable inside Claude Code (set model name to `GLM-5.2`). Native "Agent Mode" — built for agentic workloads. Works well with ChatGPT-style system/user split. Coding-first: works well with coding agent patterns (CLAUDE.md-style, .cursorrules-style). 1M context — can ingest entire codebases or document sets. Strong Chinese-language support.

### 7.5 DeepSeek V4-Pro / V4-Flash (April 24, 2026)

Specs: ~1M context, open-source. Use `maximum reasoning effort mode` for hardest problems. Prompt clearly and directly; works well with ChatGPT system/user split. Open-weights — can self-host for sensitive workloads. Among the cheapest frontier pricing ($0.14/$0.28 per M tokens). Excellent for math, code, scientific reasoning.

### 7.6 Grok 4.3 (xAI, April 30, 2026)

Specs: 1M context, $1.25/M input. Configurable reasoning (`none` / `auto` / `high`) + non-reasoning mode. Native video input. Real-time X/web data requires enabling Web Search / X Search tools. Note: Grok 5 has NOT been released — treat any "Grok 5" claims as rumor.

### 7.7 Qwen3.7-Max + Qwen 3.7 Plus (Alibaba, May 19 / June 1, 2026)

Specs: 1M context, 65,536 max output. "The Agent Frontier" — built for the agent era; Max claimed 35-hour autonomous operation. Both OpenAI- and Anthropic-API-compatible via Alibaba Cloud Model Studio. Qwen 3.7 Plus is low-cost multimodal agent with vision + video understanding. Built for long-running autonomous agent workflows. Strong Chinese + English bilingual support. Use Anthropic-style contract prompts or OpenAI-style system/user split.

### 7.8 Meta Muse Spark (April 9, 2026)

Reasoning-first, multimodal. Replaces Llama as Meta's flagship — Meta pivoted away from open-weights Llama. Treat like Gemini/GPT for prompting style.

### 7.9 Image / video / audio models

Image generation in 2026: Midjourney V8.1 (personalization-first, longer prompts now work, push `--stylize` to 1000), gpt-image-2 (text-in-image, layouts, UI mockups), Nano Banana Pro (Gemini 3 Pro Image, conversational editing; replaces Imagen 4 which shuts down August 17, 2026), FLUX.2 family (klein/pro/max), Ideogram 4.0 (open-weight 9.3B, legible text-in-image), Recraft V4.1 (editable SVG vectors), Stable Diffusion 4 Ultra. Video: Veo 3.1 (4K + native synchronized audio), Kling 3.0 (4K 60fps), Seedance 2.0, Runway Gen-4.5, Grok Imagine 1.5, Wan 2.6 (open-source), Luma Ray 3.14, Pika 2.2. Sora is discontinued. Audio: Eleven v3 (most expressive TTS), Cartesia Sonic 3.5 (fastest natural TTS, sub-90ms latency), Hume EVI 3 (any-voice speech-language model), Suno v5.5 (AI music), Lyria 3 (Google). Reasoning-based image/video models respond to visual parameters, not vibes — replace "stunning" with "overcast daylight, shallow depth of field", "epic" with "low-angle shot, wide 24mm lens", "cinematic" with "anamorphic 2.39:1, teal-and-orange grade, lens flare".

---

## 8. Failure Taxonomy — Diagnosing Broken Prompts

When a prompt produces wrong output, the cause is almost always one of a small number of failure modes. This taxonomy is the practitioner's diagnostic; identify the mode, apply the fix.

| Failure | Symptom | Fix |
|---|---|---|
| **Vagueness** | Random, generic output | Add specificity, role, constraints |
| **Context starvation** | Wrong assumptions, made-up facts | Supply background knowledge, documents, or history |
| **Hallucination trigger** | Fabricated facts/numbers | Add "Only use provided context; say 'I don't know' if absent" |
| **Format drift** | Inconsistent structure | Add explicit output format with example; use OpenAI `strict: true` |
| **Tone mismatch** | Wrong voice for audience | Use COSTAR; specify audience + tone explicitly |
| **Token overload** | Confused, contradictory output | Split into chained sub-prompts; compress context |
| **Prompt injection** | Malicious input hijacks instructions | Add injection defense; apply Lethal Trifecta / Rule of Two |
| **Reasoning failure** | Wrong answer on logic/math | Add CoT on standard models; OR switch to reasoning model |
| **Reasoning model mismatch** | Good model, poor results on reasoning task | Remove CoT scaffolding; state problem directly and completely; verify effort dial is set high enough |
| **Sycophancy (legacy Claude 4.6)** | Model agrees instead of corrects | Largely resolved in Opus 4.8 (training goal). For 4.6/4.7: add "Be honest. Disagree with me if I'm wrong." |
| **Context window overflow** | Agent loses track mid-task | Summarize history; use external memory; compress prior steps; consider MCP Tool Search to reclaim tool-schema tokens |
| **Multi-agent drift** | Agents produce inconsistent outputs | Add shared state schema; specify handoff format explicitly; use A2A protocol for cross-vendor agents |
| **Tool poisoning (MCP, 2026)** | Malicious MCP tool description manipulates model | Use allow-listed tool schemas; no free-text tool descriptions from untrusted servers |

The diagnostic flow: identify the symptom, find the matching failure mode, apply the fix, re-test. For production prompts, the diagnostic should be codified as a regression suite (see Section 9, PromptOps).

---

## 9. PromptOps — Evaluation, Testing & Governance

In 2026, treating prompts as code that must be tested, versioned, and monitored is table stakes for production systems. PromptOps is the discipline that operationalizes this. This section walks the evaluation frameworks, agent benchmarks, automated optimization, testing checklist, and governance patterns.

### 9.1 Evaluation frameworks

| Framework | Best For |
|---|---|
| **Promptfoo** | Red-teaming (50+ attack plugins), multi-model A/B comparison, YAML-driven test matrices, CI/CD; red-team module is its most distinctive 2026 feature — auto-generates adversarial inputs |
| **DeepEval** (Confident AI) | Unit-testing prompts as Python code; 50+ built-in metrics including G-Eval, hallucination detection, answer relevancy, contextual recall, faithfulness |
| **Giskard** | Open-source LLM eval; vulnerability scanning + bias/safety/quality metrics |
| **RAGAS** | RAG pipeline quality (retrieval + generation accuracy) |
| **LangSmith / LangSmith Fleet** | Full observability + eval for LangChain/LangGraph agents (formerly Agent Builder) |
| **Langfuse** (open-source) | Production observability + prompt management + experiment tracking; self-hostable |
| **Braintrust / Helicone / Arize Phoenix** | Production observability + experiment tracking |
| **Arthur Bench** | Open-source eval for comparing LLMs, prompts, hyperparameters |
| **Inspect AI** (UK AISI) | Security-focused eval framework for safety testing |

### 9.2 Agent benchmarks (2026 — harder, more realistic)

The 2026 theme: benchmarks closer to production reality produce dramatically lower scores than SWE-Bench Verified. The field is regrounding in harder, more realistic evals.

| Benchmark | What it measures |
|---|---|
| **τ-bench (tau-bench)** (Sierra) | Real-world tool-using agents; checks database state post-task. τ2 adds `pass^k` (reliable success across repeated runs). Top-model scores: 50–70% |
| **SWE-Bench Verified** (500-instance) | Coding default, increasingly criticized as over-fit |
| **SWE-Bench Pro** (1,865 problems) | Harder variant of SWE-Bench |
| **Terminal-Bench** (arXiv 2601.11868) | Hard, realistic terminal tasks; built on the Harbor framework |
| **AppWorld** | Cited alongside τ-bench and SWE-Bench |
| **MCP-SafetyBench** | LLM-agent safety across 5 domains and 20 attack types on real-world MCP servers |
| **NRT-Bench** (arXiv 2606.20408, June 2026) | Multi-turn red-teaming, fixed-judge replay pipeline |

### 9.3 The 2026 trend — layered eval pipelines

Leading teams ship three eval layers: (1) fast property checks in CI (schema, format, regex assertions — seconds), (2) quality metrics pre-release (DeepEval/Promptfoo + LLM-as-judge — minutes), (3) red-team + adversarial pre-deploy (Promptfoo red-team / PyRIT / Garak — hours). The layered approach catches different failure classes at different points in the development cycle, with the cheap fast checks running on every commit and the expensive red-team running pre-deploy.

### 9.4 The prompt testing checklist

Before deploying any production prompt: regression suite (run eval across all known test cases), multi-model test (does it work on Claude AND GPT AND Gemini AND open-weights?), edge cases (empty input, very long input, adversarial input, multi-language evasion), format drift detection (does structured output always parse correctly? Use OpenAI strict mode / `strict: true`), cost/quality tradeoff (does a shorter prompt perform as well?), model-fit test (re-validate when the underlying model updates — model drift), red-team (Garak / PyRIT / Promptfoo red-team against OWASP LLM Top 10 (2025) + OWASP ASI Top 10 for Agentic (2026)), MCP-specific (if using MCP servers, run MCP-SafetyBench against the tool surface).

### 9.5 Governance patterns

Prompt versioning — track changes; every prompt has a version hash. Eval-as-code — eval suites in CI/CD (Promptfoo YAML or DeepEval pytest). Regression cron — scheduled eval runs to catch model drift. Prompt libraries — catalog with owner, version, eval coverage, last validated model. System cards / model cards — treated as compliance artifacts under EU AI Act (full enforcement August 2, 2026 for GPAI providers). Agent governance — Microsoft Agent Governance Toolkit maps controls to OWASP ASI Top 10.

The governance patterns are not optional for production systems in 2026. The EU AI Act's full enforcement (August 2, 2026) makes system cards and model cards legal compliance artifacts; the 70–95% production agent failure rate (Fiddler AI, 2026) makes eval-as-code and regression crons operational necessities. PromptOps is the discipline that turns prompt engineering from a craft into an engineering practice.

---

## 10. Worked Examples

This section walks four worked examples — vague-to-precise transformations — that illustrate the discipline in practice. Each example shows the "before" (vague prompt) and the "after" (precise prompt using the appropriate framework), with a brief note on what changed and why.

### 10.1 Example A: Vague → COSTAR-precise (blog post)

**Before:** `Write a blog post about AI.`

This prompt fails on first principles 1 (vagueness), 4 (no output definition), 5 (no negative constraints). The model has no information about audience, length, tone, or format; it will produce a generic ~500-word essay with bullet lists and the word "delve."

**After (COSTAR):**

```
Context: AI agents are increasingly replacing manual knowledge work in 2025/26.
Objective: Convince non-technical executives that AI agents will reshape middle management.
Style: Opinion piece — punchy paragraphs, no bullet lists, 2–3 real company examples.
Tone: Authoritative but accessible — Harvard Business Review register.
Audience: C-suite readers with no technical background.
Response: 600 words. Headline + subheadline + 3 paragraphs + one-sentence CTA.
  Do not use "delve". Do not use bullet lists. End with a CTA, not a summary.
```

**What changed:** Every COSTAR component is filled. The audience is specified (C-suite, no technical background), the tone is specified (HBR register), the format is specified (600 words, headline + 3 paragraphs + CTA), and negative constraints close off the model's defaults ("delve", bullet lists, summary endings). The prompt now constrains the output space to a small region that matches the user's intent.

### 10.2 Example B: Hallucination fix (Context Engineering + Constraints)

**Problem:** "Tell me about our Q3 revenue" → Model makes up numbers.

This is a context-starvation problem (the model lacks the Q3 report) combined with a missing negative constraint (no instruction to refuse when the data is absent). The fix uses context engineering (inject the report) plus a constraint (only use provided numbers).

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

**What changed:** The role activates the financial-analyst persona with an explicit honesty directive. The context injects the actual Q3 report. The constraints close off the three gaming vectors (estimation, interpolation, inference) and add an uncertainty flag. The model now has the data it needs and explicit instructions not to fabricate.

### 10.3 Example C: Reasoning model prompt (Claude Opus 4.8 ET / GPT-5.6 max reasoning)

**Wrong:** `Think step by step: Is it better to invest in index funds or individual stocks for a 32-year-old with $10,000?`

This prompt adds CoT scaffolding to a reasoning model, wasting thinking budget. The model will produce a longer, less focused answer than it would with a direct problem statement.

**Right:**

```
A 32-year-old with $10,000 to invest asks whether index funds or individual stocks are better
for a 20-year horizon. Consider: risk tolerance implied by age/timeline, diversification
trade-offs, empirical data on average investor performance in each approach, and fee structures.

Provide: A clear recommendation (1 sentence) followed by a 200-word rationale.
```

Set `thinking: {type: "enabled", budget_tokens: 8000, effort: "extra"}` (Claude) or `reasoning: {effort: "max"}` (GPT-5.6) via the API parameter — do NOT put CoT instructions in the prompt itself.

**What changed:** The CoT scaffolding is removed. The problem statement is complete (age, amount, horizon, considerations). The output format is specified (recommendation + 200-word rationale). The reasoning effort is set via the API parameter, not the prompt. The model now allocates its thinking budget to the actual reasoning rather than to following external CoT instructions.

### 10.4 Example D: RISEN for an agentic workflow

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

**What this illustrates:** RISEN is the right framework for procedural / task-oriented prompts. The Role activates the support-agent persona. The Instructions specify the action verb (investigate and resolve). The Steps are a numbered sequence the agent follows. The End goal defines success. The Narrowing closes off scope creep (English only, 150-word limit, no unverified delivery dates). The order_context is injected as XML-tagged context. This is the bridge to harness engineering (Section 11): the RISEN prompt is, in 2026 parlance, a minimal harness specification — it defines role, tools (implicit: order database), workflow, decision rules, and exit conditions.

---

## 11. The Bridge to Higher Layers

Prompt engineering is the foundation, but production systems in 2026 require all four layers. This section briefly bridges to the higher layers, with cross-references to the companion briefs.

### 11.1 Prompt → Context engineering

Rule 22 of the 22 power rules — "engineer context, not just words" — is the bridge. For production and agentic systems, what's IN the context window matters as much as how the instruction is phrased. The four canonical context operations (Write / Select / Compress / Isolate, per Anthropic's September 2025 essay) are the consensus vocabulary. The full deep research on context engineering is at `/home/z/my-project/download/context-engineering-deep-research.md`.

### 11.2 Context → Harness engineering

Even a perfectly-engineered prompt inside a perfectly-engineered context fails if the agent lacks the right tools, the right memory, the right runtime, the right middleware. The harness is the per-execution system around one agent run — prompt + context + tools + memory + runtime + middleware. The harness engineering checklist (reasoning step before action, hard cap on tool calls, tool schemas as public API contracts, context-first, explicit exit conditions, runtime engineering over prompt tricks, middleware/hook architecture) is the production discipline. The full deep research on harness engineering is at `/home/z/my-project/download/harness-engineering-deep-research.md`.

### 11.3 Harness → Loop engineering

Even a perfectly-harnessed single agent run fails in production if it must run on a schedule, respond to events, persist state across crashes, verify its own output independently, and stop itself when a goal is met. The loop is the recurring control system above the harness — trigger + verifiable goal + harness + verification + iteration + stopping. Loop engineering is the discipline that takes the 70–95% production agent failure rate seriously. The full deep research on loop engineering is at `/home/z/my-project/download/loop-engineering-deep-research.md`.

### 11.4 Why all four layers are necessary

The four layers stack rather than replace because each addresses a different failure class. A loop with a broken prompt fails on every iteration. A harness with polluted context produces a confidently-wrong agent. A well-contextualized agent with a vague prompt produces generic output. Production systems in 2026 need all four layers, and the discipline of the loop engineer explicitly includes the engineering of the harness, context, and prompt for each iteration. The naive reading — that loop engineering "replaces" prompt engineering — is wrong; loop engineering *automates* the invocation of a well-engineered harness, which itself contains a well-engineered context, which itself contains a well-engineered prompt.

---

## 12. Comparison to Adjacent Layers + Open Problems

### 12.1 Comparison table

| Layer | Scope | What it designs | Time horizon | Primary failure mode | When to use | When NOT to use | Key question | Representative thinker |
|---|---|---|---|---|---|---|---|---|
| **Prompt engineering** | The instruction | The prompt | One model call | Vagueness, hallucination | Every task — table stakes | Never (always required) | "What should the model do?" | Many (2022) |
| **Context engineering** | The context window | What's in the context (Write/Select/Compress/Isolate) | One model call or one agent run | Context starvation, context pollution | Multi-turn agents, RAG, long sessions | Rarely omit (only trivial one-shots) | "What should the model know right now?" | Anthropic (Sept 2025) |
| **Harness engineering** | The per-execution system | Prompt + context + tools + memory + runtime + middleware | One agent run | Single-run fragility (wrong tools, no exit conditions) | Any production agent | Never (always required for production) | "What system does the model need to succeed once?" | Karpathy / Hashimoto (Feb 2026) |
| **Loop engineering** | The multi-execution control system | Trigger + verifiable goal + harness + verification + iteration + stopping | Many agent runs (scheduled/event-driven/until-goal) | Verification gaming, runaway loops, state loss | Recurring/event-driven/multi-iteration agent fleets | One-shot tasks, unverifiable goals, irreversible actions, low volume | "What system triggers, verifies, and stops many runs?" | Addy Osmani / LangChain (June 2026) |

### 12.2 Open problems in prompt engineering

**The reasoning-model transition.** The shift from CoT-scaffolded prompts to reasoning-model direct prompts is incomplete. Many production prompts still contain "think step by step" instructions that are counterproductive on Claude Opus 4.8, GPT-5.6, and Gemini 3.5 Deep Think. The discipline needs a clear migration path: identify reasoning-model deployments, audit their prompts for CoT scaffolding, remove the scaffolding, set the effort dial via the API parameter, and re-evaluate. The migration is mechanical but frequently skipped.

**Model-specific prompt shapes.** The divergence of prompt shapes across models (Claude's XML, GPT's markdown headers, GLM's system/user split, Gemini's "Be concise.") means a prompt that works on one model may underperform on another. Production systems that support multiple models need per-model prompt variants, with the variant selected at runtime. The discipline lacks a clean abstraction for this; current practice is hand-maintained per-model prompt files.

**Automated prompt optimization at scale.** DSPy, TextGrad, and LLM-as-judge work for individual prompts but do not scale to the hundreds of prompts in a production system. The discipline needs tooling that automates prompt optimization across a prompt library, with regression testing and rollback. Current tooling (Promptfoo, DeepEval) is good for testing but not for optimization at scale.

**The prompt-as-code boundary.** As prompts become more structured (XML, JSON schemas, function calls), the boundary between "prompt" and "code" blurs. The discipline needs a clear abstraction: what is the prompt, what is the harness, what is the application code? The four-layer stack is the current answer, but the boundaries are not always clean in practice.

**Prompt injection defense.** Customer-facing prompts are vulnerable to prompt injection — malicious user input that hijacks the model's instructions. The OWASP LLM Top 10 (2025) and OWASP ASI Top 10 for Agentic (2026) formalize the attack classes. The Lethal Trifecta / Rule of Two security pattern (agent has private data + exfiltration tools + untrusted input → strict instruction/data separation and human approval) is the mitigation. The discipline needs better default-defenses; current practice is per-prompt hardening, which does not scale.

---

## 13. Glossary

- **Prompt:** The instruction given to an LLM that produces output. The artifact of prompt engineering.
- **Zero-Shot:** A prompt with no examples; the model produces output from the instruction alone.
- **Few-Shot:** A prompt with 3–5 input → output examples; the model infers format and approach from the examples. Highest-ROI technique (~40% accuracy improvement).
- **Chain-of-Thought (CoT):** A prompt that asks the model to reason step by step before answering. Effective on standard models; counterproductive on reasoning models.
- **Tree-of-Thought (ToT):** A prompt that explores multiple reasoning paths in parallel; expensive.
- **ReAct:** A prompt pattern for agents — Reason + Act in a loop, calling tools as needed.
- **Chain-of-Verification (CoV):** A prompt that asks the model to verify its own output; reduces hallucination.
- **Self-Consistency:** A prompt sampled multiple times with the model voting on the answer; for subjective tasks.
- **COSTAR:** A framework — Context, Objective, Style, Tone, Audience, Response. Best for content/communication tasks.
- **RISEN:** A framework — Role, Instructions, Steps, End goal, Narrowing. Best for procedural/task-oriented prompts.
- **Reasoning model:** A model that reasons internally via API parameters (Claude Opus 4.8 effort, GPT-5.6 max reasoning, Gemini 3.5 Deep Think, Grok 4.3 reasoning, DeepSeek V4 maximum reasoning). Eliminates CoT scaffolding.
- **Effort dial:** The API parameter that controls a reasoning model's thinking budget. Claude: `low`/`high`/`extra`/`max`. GPT: `reasoning.effort`. Gemini: adaptive. Grok: `none`/`auto`/`high`. DeepSeek: `maximum reasoning effort mode`.
- **Anatomy Template:** The general-purpose eight-layer prompt template — Role, Context, Task, Format, Tone, Examples, Constraints, Evaluation.
- **Contract-style XML:** Claude Opus 4.8's preferred prompt shape — `<role>`, `<context>`, `<task>`, `<constraints>`, `<output_format>`, `<examples>` tags.
- **System/user split:** The ChatGPT-style prompt shape — system message (persistent rules, persona, output format defaults) + user message (context, task, format, constraints).
- **PromptOps:** The discipline of treating prompts as code — versioned, tested, monitored. Tools: Promptfoo, DeepEval, Giskard, RAGAS, LangSmith, Langfuse.
- **DSPy:** Stanford NLP's automated prompt optimization framework; treats prompts as parameters to optimize.
- **LLM-as-judge:** Using a strong LLM to grade outputs of another LLM; the mature pattern for prompt evaluation.
- **Strict mode:** OpenAI's `strict: true` parameter on function/tool definitions for guaranteed JSON Schema conformance.
- **Prompt caching:** Caching the prompt prefix to reduce cost (Claude prompt caching, GPT-5.6 cache breakpoints). Cache reads typically 90% discount.
- **Lethal Trifecta / Rule of Two:** A security pattern — agent has private data + exfiltration tools + untrusted input → strict instruction/data separation and human approval.
- **MCP (Model Context Protocol):** The standard protocol for agent-to-tool communication, under the Linux Foundation's Agentic AI Foundation. The 2026-07-28 spec is stateless.
- **Agent Skills:** Anthropic's modular, discoverable capability packages — bundle a prompt with optional scripts, references, and assets.
- **A2A (Agent2Agent) protocol:** Google Cloud / Linux Foundation; agent-to-agent counterpart to MCP's agent-to-tool.
- **OWASP LLM Top 10 (2025) / OWASP ASI Top 10 for Agentic (2026):** The standard taxonomies of LLM and agentic security risks.
- **EU AI Act:** EU regulation; full enforcement for GPAI providers begins August 2, 2026. Makes system cards and model cards legal compliance artifacts.
- **Fiddler AI (2026):** The 70–95% production agent failure rate statistic. The bottleneck is the surrounding system, not the model.

---

## 14. Primary Sources & Further Reading

### Foundational documentation

- **OpenAI Prompt Engineering Guide** (late 2022) — platform.openai.com/docs/guides/prompt-engineering. The original practitioner guide; crystallized the discipline.
- **Anthropic Prompt Design documentation** (early 2023) — docs.anthropic.com. The Claude-specific prompting guide; introduced XML-tagged prompts.
- **Anthropic, "Effective Context Engineering for AI Agents"** (September 2025) — anthropic.com/engineering/effective-context-engineering-for-ai-agents. The canonical four context operations: Write / Select / Compress / Isolate. The bridge from prompt to context engineering.
- **Karpathy / Hashimoto (Feb 2026)** — coinage of "harness engineering." See the Harness Engineering brief for full sourcing.

### Technique references

- **Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"** (NeurIPS 2022) — the original CoT paper.
- **Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models"** (ICLR 2023) — the ReAct pattern.
- **Yao et al., "Tree of Thoughts"** (NeurIPS 2023) — ToT.
- **Li et al., "Chain-of-Verification"** (2023) — CoV.
- **Khatab et al., "Skeleton-of-Thought"** (2023) — SoT.
- **Wang et al., "Self-Consistency Improves Chain of Thought Reasoning"** (ICLR 2023) — self-consistency.
- **COSTAR framework** — GovTech Singapore; popularized 2024.
- **RISEN framework** — practitioner framework; popularized 2024.
- **DSPy** (Stanford NLP) — dspy.ai. Automated prompt optimization.
- **TextGrad** — github.com/zou-group/textgrad. Gradient-based prompt optimization.

### Model-specific guides

- **Anthropic Claude Opus 4.8 documentation** — docs.anthropic.com. Contract-style XML, effort dial, dynamic workflows.
- **OpenAI GPT-5.6 documentation** — platform.openai.com. Markdown headers, max reasoning effort, cache breakpoints, strict mode.
- **Google Gemini 3.5 documentation** — ai.google.dev. Adaptive thinking, Computer Use.
- **Z.ai / Zhipu GLM-5.2** — bigmodel.cn and github.com/zai-org/GLM-5.2. Native Agent Mode, system/user split, coding-first.
- **DeepSeek V4** — github.com/deepseek-ai. Maximum reasoning effort mode.
- **xAI Grok 4.3** — x.ai. Configurable reasoning.
- **Alibaba Qwen3.7** — alibabacloud.com. OpenAI/Anthropic API compatible.

### PromptOps & evaluation

- **Promptfoo** — promptfoo.dev. Red-teaming, multi-model A/B, CI/CD.
- **DeepEval (Confident AI)** — github.com/confident-ai/deepeval. 50+ metrics, pytest-style.
- **Giskard** — github.com/Giskard-AI/giskard. Vulnerability scanning.
- **RAGAS** — github.com/explodinggradients/ragas. RAG pipeline quality.
- **LangSmith / Langfuse** — smith.langchain.com / langfuse.com. Observability + eval.
- **Inspect AI (UK AISI)** — github.com/UKGovernmentBEIS/inspect_ai. Security-focused eval.

### Security & governance

- **OWASP LLM Top 10 (2025)** — owasp.org/www-project-top-10-for-large-language-model-applications.
- **OWASP ASI Top 10 for Agentic (2026)** — agent security.
- **Simon Willison, "Lethal Trifecta"** (June 16, 2025) — simonwillison.net. The security pattern.
- **Meta AI, "Agents Rule of Two"** (October 2025) — research.facebook.com.
- **EU AI Act** — full enforcement August 2, 2026 for GPAI providers.

### Benchmarks

- **τ-bench (Sierra)** — github.com/sierra-research/tau-bench. Real-world tool-using agents.
- **SWE-Bench Verified / SWE-Bench Pro** — github.com/princeton-nlp/SWE-bench. Coding benchmarks.
- **Terminal-Bench** (arXiv 2601.11868) — terminal-task benchmark.
- **MCP-SafetyBench** — MCP-server safety benchmark.
- **NRT-Bench** (arXiv 2606.20408) — multi-turn red-teaming.

### Companion briefs (this project)

- **Context Engineering deep research** — `/home/z/my-project/download/context-engineering-deep-research.md`. The four operations (Write/Select/Compress/Isolate), the five context layers, RAG, memory systems, MCP.
- **Harness Engineering deep research** — `/home/z/my-project/download/harness-engineering-deep-research.md`. The six harness components, the seven-item checklist, tool engineering, middleware, multi-agent orchestration.
- **Loop Engineering deep research** — `/home/z/my-project/download/loop-engineering-deep-research.md`. The six loop components, the seven topologies, the verification problem, the ten-point checklist, the worked CI-fix example.

---

## 15. Appendix A — The 22 Power Rules (annotated)

1. **Be specific** — every vague word is a failure opportunity.
2. **Assign a role** — `"You are a senior [expert]"` activates specialized knowledge.
3. **Provide context** — what the model doesn't know, it will invent.
4. **Give examples** — few-shot is the single highest-ROI technique (+40% accuracy).
5. **Define the output** — specify format, length, structure BEFORE the task.
6. **Use delimiters** — separate sections with XML tags, `---`, or `###`.
7. **Chain-of-thought (non-reasoning models only)** — ask for reasoning before final answer.
8. **Never use CoT on reasoning models** — it wastes their thinking budget; state the problem directly.
9. **Constrain negatively** — tell the model what NOT to do.
10. **Ground with data** — inject relevant documents to reduce hallucination.
11. **Calibrate confidence** — "If confidence < 80%, flag it and suggest verification".
12. **Specify audience** — same content differs for a CEO vs. a developer.
13. **Compress** — remove filler; every token must earn its place.
14. **Use structured output** — JSON/XML for anything parsed programmatically; use OpenAI `strict: true` for guaranteed JSON Schema conformance.
15. **Temperature matters** — 0–0.2 for facts, 0.7–1.0 for creativity.
16. **Model-match** — prompting style differs per model.
17. **Iterate** — treat prompts as code: version, test, measure, improve.
18. **Security** — assume malicious user input in customer-facing prompts (see Lethal Trifecta).
19. **Test edge cases** — adversarial inputs reveal prompt weaknesses.
20. **Prompt chain** — break complex tasks into sequential sub-prompts.
21. **Document** — add a comment header to every production prompt.
22. **Engineer context, not just words** — for production/agentic systems, what's IN the context window matters more than how the instruction is phrased. The bridge to context engineering.

---

## 16. Appendix B — Prompt Engineering Pre-Flight Checklist

Run this 15-item checklist before deploying any production prompt. Each item should be a clear yes; any "no" blocks deployment.

1. **Specific task verb?** The task uses a strong verb (analyze, generate, compare, extract), not a vague phrasing. [ ] Yes [ ] No
2. **Role assigned?** A specific expert role is activated, with relevant expertise. [ ] Yes [ ] No
3. **Context provided?** All background the model needs is in the prompt or injected context. [ ] Yes [ ] No
4. **Examples included?** 3–5 few-shot examples covering input variation (where format/approach matters). [ ] Yes [ ] No
5. **Output format specified?** JSON schema, table columns, numbered list, paragraph count — explicitly defined. [ ] Yes [ ] No
6. **Negative constraints?** "Do not..." clauses close off common failure modes. [ ] Yes [ ] No
7. **Length limits?** Word/character/line limits explicitly stated. [ ] Yes [ ] No
8. **Audience specified?** Who reads the output — expertise level, role, mindset. [ ] Yes [ ] No
9. **Tone specified?** Emotional register — authoritative, friendly, urgent, empathetic. [ ] Yes [ ] No
10. **Model-matched?** Prompt shape matches the target model (Claude XML, GPT markdown, GLM system/user, Gemini concise). [ ] Yes [ ] No
11. **Reasoning model handled?** If using a reasoning model (Opus 4.8, GPT-5.6 max, Gemini Deep Think, Grok reasoning, DeepSeek max), no CoT scaffolding; effort dial set via API. [ ] Yes [ ] No
12. **Delimiters used?** Sections separated with XML tags, `---`, or `###` for clarity. [ ] Yes [ ] No
13. **Security check?** If customer-facing, Lethal Trifecta / Rule of Two applied; instruction/data separation; allow-listed tools. [ ] Yes [ ] No
14. **Eval suite written?** Regression suite (Promptfoo YAML or DeepEval pytest) covers known test cases. [ ] Yes [ ] No
15. **Versioned + documented?** Prompt has a version hash, owner, last-validated-model, and comment header. [ ] Yes [ ] No

---

## 17. Appendix C — Changelog

Brief compiled July 2026. Reflects the state of prompt engineering as of the late-June 2026 model landscape (Claude Opus 4.8, GPT-5.6 Sol/Terra/Luna, Gemini 3.5 Flash + Pro announced, GLM-5.2, DeepSeek V4, Grok 4.3, Qwen3.7-Max, Meta Muse Spark). The technique catalog (Zero-Shot, Few-Shot, CoT, ToT, ReAct, CoV, Self-Consistency, SoT, Multi-Agent Debate, COSTAR, RISEN) is stable; the reasoning-model fast-path (no CoT scaffolding, effort dial via API) is the major 2026 transformation. Frontier-model specifics (names, pricing, context windows, capability claims) will drift; re-verify against current vendor docs before relying on them in production. The EU AI Act's full enforcement for GPAI providers begins August 2, 2026; the OWASP LLM Top 10 (2025) and OWASP ASI Top 10 for Agentic (2026) are the current security references. Companion briefs on context engineering, harness engineering, and loop engineering are at `/home/z/my-project/download/`.
