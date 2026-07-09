# Prompting Techniques — Deep Dive (End-of-June 2026)

## Table of Contents
1. Zero-Shot
2. Few-Shot / In-Context Learning
3. Chain-of-Thought (CoT)
4. Zero-Shot CoT
5. Tree of Thought (ToT)
6. ReAct (Reason + Act)
7. Self-Consistency
8. Meta-Prompting
9. Prompt Chaining
10. Skeleton-of-Thought (SoT)
11. Role / Persona Prompting
12. Prompt Compression
13. RAG Prompting
14. Calibrated Confidence
15. Program-Aided Language Models (PAL) / Program-of-Thought (PoT)
16. Multimodal Prompting
17. **COSTAR Framework**
18. **RISEN Framework**
19. **Chain-of-Verification (CoV)**
20. **Multi-Agent Debate Prompting**
21. **Context Engineering (Write / Select / Compress / Isolate)**
22. **Harness Engineering (2026)**
23. **Agent Skills Pattern (Anthropic, 2026)**
24. **MCP Tool Search / Dynamic Tool Loading (2026)**
25. **A2A Protocol for Cross-Vendor Agents**
26. **Subagent Orchestration (2026)**
27. **DSPy / Automated Prompt Optimization**
28. **Anti-Slop Prompting (image/video)**
29. **Personalization-Driven Prompting (Midjourney V8+)**
30. **Voice/Music Stage-Direction Prompting (2026)**
31. **Loop Engineering (June 2026)**

---

## 1. Zero-Shot
Direct task instruction without examples. Best for: simple, well-defined tasks.

```
Classify this review as Positive, Negative, or Neutral:
"The product arrived on time but felt cheaply made."
```

**When to use:** Task is unambiguous, model already has strong priors, speed matters.
**When to avoid:** Novel task formats, domain-specific outputs, high-accuracy requirements.

---

## 2. Few-Shot / In-Context Learning
Providing 3–5 worked examples dramatically improves accuracy (+40% on reasoning tasks).

**Template:**
```
Task: [description]

Example 1:
Input: [input]
Output: [ideal output]

Example 2:
Input: [input]
Output: [ideal output]

Now complete:
Input: [new input]
Output:
```

**Tips:**
- Examples should cover edge cases, not just the easy center case
- Order matters: put the hardest/most representative example last
- Keep example format identical to what you want in the output
- 3 examples is the sweet spot for most tasks; 5 for ambiguous ones
- **2026 note:** For models with strong reasoning (Opus 4.8, GPT-5.6), zero-shot often matches few-shot — reserve few-shot for ambiguous formats or domain-specific output shapes

---

## 3. Chain-of-Thought (CoT)
Instructs the model to reason step-by-step before answering. Dramatically improves math, logic, multi-step tasks.

**⚠️ 2026 critical update:** CoT is **counterproductive on reasoning models** (Claude Opus 4.8 ET, GPT-5.6 max reasoning, Gemini 3.5 Deep Think, Grok 4.3 reasoning, DeepSeek V4 max reasoning). These models reason internally; explicit CoT instructions waste thinking budget. Use CoT only on non-reasoning models.

**Template (non-reasoning models only):**
```
Question: [problem]

Let's think through this step by step:
1. First, [aspect 1]
2. Then, [aspect 2]
3. Finally, [conclusion]

Answer: [final answer]
```

**Few-shot CoT example:**
```
Q: Roger has 5 tennis balls. He buys 2 more cans of 3 each. How many does he have?
A: Let's think step by step.
   - Start: 5 balls
   - Bought: 2 cans × 3 = 6 balls
   - Total: 5 + 6 = 11 balls ✓

Q: [your question]
A: Let's think step by step.
```

---

## 4. Zero-Shot CoT
Simply append "Let's think step by step" — no examples needed. Works on GPT-4, Claude, Gemini.

```
[Question or problem]

Let's think step by step.
```

Also effective: "Work through this carefully before giving your final answer."

**⚠️ Do NOT use on reasoning models** — set the reasoning effort via API parameter instead.

---

## 5. Tree of Thought (ToT)
Explores multiple reasoning paths simultaneously — like a decision tree. Best for: complex problem-solving, strategic planning, creative tasks with multiple valid approaches.

**Template:**
```
Consider [problem/question].

Explore THREE different approaches:

Approach A: [path 1 — describe direction]
[Reason through approach A]
Assessment: [strengths/weaknesses]

Approach B: [path 2]
[Reason through approach B]
Assessment: [strengths/weaknesses]

Approach C: [path 3]
[Reason through approach C]
Assessment: [strengths/weaknesses]

Final recommendation: [synthesize best path]
```

---

## 6. ReAct (Reason + Act)
Combines chain-of-thought with tool use. Standard in agent frameworks (LangChain, CrewAI, AutoGen → Microsoft Agent Framework, OpenAI Agents SDK).

**Pattern:**
```
Thought: [what I need to figure out]
Action: [tool to use]
Observation: [what the tool returned]
Thought: [what I now know]
Action: [next tool call or final answer]
```

**User-side prompt:**
```
You have access to: [list tools].
To answer the question, think about what you need to know, choose a tool, observe the result, and repeat until you can give a final answer.

Question: [question]
```

**2026 note:** For MCP-connected agents with many tools, use **MCP Tool Search** (dynamic tool loading) instead of preloading all tool descriptions in the prompt — saves context and prevents tool-description injection attacks.

---

## 7. Self-Consistency
Generate multiple independent answers with different reasoning, then majority-vote the final answer. Reduces variance on high-stakes tasks.

**Template:**
```
Solve the following problem three independent times, using different reasoning approaches each time.

Problem: [problem]

Solution 1: [reason and answer]
Solution 2: [reason and answer]
Solution 3: [reason and answer]

Final answer (majority vote or best-justified): [answer]
```

**Use case:** Legal analysis, financial modeling, medical diagnosis support — anywhere variance is costly.

**2026 note:** This is essentially what reasoning models do internally when you set effort to "max" — they sample multiple chains and select the best. Use explicit self-consistency when you must inspect intermediate states or use non-reasoning models.

---

## 8. Meta-Prompting
Using AI to write or improve prompts. A "meta-prompt" asks the model to generate an optimized downstream prompt.

**Template:**
```
Write the best possible [system/user] prompt for an AI that will [task description].

The AI will be used by [audience] to [goal].
The most important quality is [key criterion].
It should avoid [failure mode].

Return only the prompt, ready to use.
```

**Example:**
```
Write the best possible system prompt for an AI customer service agent
specializing in B2B SaaS billing support. It must be empathetic, technically
accurate, and know when to escalate to a human agent. Avoid promising anything
not in the product terms of service. Return only the system prompt.
```

---

## 9. Prompt Chaining
Break complex tasks into sequential sub-prompts. Output of Step N becomes input to Step N+1.

**When to use:** Document analysis → synthesis → action plan; multi-step code generation; research pipelines.

**Template:**
```
Step 1: Extract [structured data] from [input]
 [output passed to Step 2]

Step 2: Analyze [structured data] for [criterion]
 [output passed to Step 3]

Step 3: Generate [deliverable] based on [analysis]
```

**Example chain:**
1. "Extract all action items from this meeting transcript as a JSON array"
2. "For each action item, assign priority (1–3) and a suggested owner role"
3. "Write a follow-up email using these prioritized action items"

**2026 note:** On Claude Opus 4.8 with adaptive thinking (default), explicit prompt chaining is only needed when you must inspect or log intermediate outputs. The model handles multi-step reasoning internally.

---

## 10. Skeleton-of-Thought (SoT)
Model generates outline first, then fills each section. Reduces latency for long-form content.

**Template:**
```
Write a [document type] about [topic].

First, generate just the outline (headers only).
Then fill in each section one at a time, clearly labeling each.

Topic: [topic]
Audience: [audience]
Length target: [length]
```

---

## 11. Role / Persona Prompting
Assigning a specific expert persona activates relevant training patterns and improves quality.

**Template:**
```
You are a [specific title] with [N] years of experience in [domain],
specializing in [specialty]. You have [relevant credential/context].

[Task]
```

**High-impact roles:**
- "You are a board-certified [specialist] explaining to [audience]"
- "You are a principal engineer at a FAANG company doing code review"
- "You are an NYT editor reviewing this draft for clarity and impact"
- "You are a skeptical investor stress-testing this business plan"

**Tip:** The more specific and credentialed the persona, the better. "Senior" > "expert" > nothing.

**2026 note:** For Claude Opus 4.8, you don't need to over-credential — 4.8 follows instructions literally. Keep personas specific but not flowery.

---

## 12. Prompt Compression
Removing unnecessary tokens while preserving intent. Compressed prompts are often 40% shorter with equal or better performance.

**Before:**
```
I would like you to please help me by analyzing the following customer feedback
that I have collected and then providing me with a comprehensive summary that
covers the main themes and sentiment.
```

**After:**
```
Analyze this customer feedback. Return: main themes, overall sentiment, top 3 actionable insights.
```

**Rules:**
- Remove filler: "please", "I would like you to", "could you"
- Convert paragraphs to bullet constraints
- Remove adjectives that don't change the instruction
- Merge redundant constraints

---

## 13. RAG Prompting (Retrieval-Augmented Generation)
Inject retrieved documents into context to reduce hallucination and keep knowledge current.

**Template:**
```
You are a [role]. Answer questions using ONLY the documents provided below.
If the answer is not present in the documents, say "I don't have that information in the provided context."
Do not use outside knowledge. Do not infer or estimate.

<documents>
[Document 1]
---
[Document 2]
</documents>

Question: [question]
```

**Hallucination prevention additions:**
- "Cite the document section for every claim"
- "If confidence is below 80%, flag it"
- "Quote the exact sentence that supports your answer"

**2026 note (RAG poisoning research):** Just 5 carefully crafted documents can manipulate AI responses 90% of the time through RAG retrieval (Jan 2026 research). Apply input validation on retrieved documents, not just user queries.

**Advanced RAG architectures (2026):**
- **GraphRAG** (Microsoft) — graph-structured retrieval; strong on cross-document reasoning but expensive.
- **HippoRAG 2** — neurobiologically-inspired long-term memory; **10–30× cheaper than GraphRAG, 6× faster**; enabling LLMs to continuously integrate knowledge across documents. Strong default for production memory in 2026.
- **MemGraphRAG** — memory-based multi-agent graph RAG (2026 paper); outperforms SOTA baselines at comparable efficiency.
- **LightRAG / nano-graphrag** — lighter-weight graph-RAG alternatives for self-hosting.
- **Corrective RAG (CRAG)** — retrieval + self-correction; model validates retrieved docs before use.
- **Adaptive RAG** — model decides per-query whether to retrieve, multi-retrieve, or answer from parametric knowledge.
- **Self-RAG** — model self-reflects on retrieval quality and decides when to retrieve more.
- **RAPTOR** — recursive abstractive tree retrieval; good for long documents.
- **HyDE** — Hypothetical Document Embedding; generate a hypothetical answer, embed it, retrieve real docs similar to it.

**Vector DB / retrieval stack (late June 2026):**
- Pinecone (serverless), Weaviate (Embeddings 3.0), Qdrant, Chroma, Milvus 2.5+, pgvector
- Managed: Astra DB (DataStax), MongoDB Atlas Vector, Supabase Vector, Elasticsearch/OpenSearch neural
- Embedding models: OpenAI text-embedding-3-large, Voyage AI, Cohere Embed v4, Jina, BGE (BAAI), Nomic Embed
- Frameworks: LlamaIndex (AgentWorkflow for multi-agent RAG), LangChain 1.0 RAG, Haystack 2.x, DSPy

---

## 14. Calibrated Confidence
Ask the model to express uncertainty when appropriate. Critical for high-stakes domains.

**Template:**
```
Answer the following. For each claim:
- If you are highly confident (>90%): state it directly
- If moderately confident (60–90%): note the uncertainty
- If low confidence (<60%): explicitly flag it and recommend verification sources

[Question]
```

**2026 note:** Claude Opus 4.8 has anti-sycophancy as a training goal and is ~4× less likely to let flaws pass unremarked. Invite this behavior explicitly: "Flag any uncertainties in your answer."

---

## 15. Program-Aided Language Models (PAL) / Program-of-Thought (PoT)
Model writes executable code to solve problems rather than computing directly in language. Reduces arithmetic errors.

**Template:**
```
Solve this problem by writing Python code.
Run the code mentally (or actually execute it), and return the final numeric answer.

Problem: [math/logic problem]

```python
# Write the solution code here
```

Answer: [result from running the code]
```

**2026 note:** Claude has a built-in **code execution tool** (`type: code_execution_20260120`) — GA, no beta header required. Runs Python in a secure sandbox. Available on Claude Fable 5, Mythos 5, Opus 4.5+, and others. Use this instead of asking the model to "run the code mentally."

---

## 16. Multimodal Prompting
Combining text + images/audio/video as input (GPT-5.6, Claude Opus 4.8, Gemini 3.5, Grok 4.3).

**Image + text template:**
```
[Attach image]

Analyze the image above and:
1. [Specific observation task]
2. [Comparison or measurement task]
3. [Action or recommendation based on observation]

Focus on: [key element]
Ignore: [irrelevant elements]
Output format: [structure]
```

**Tips:**
- Be explicit about what aspect of the image to focus on
- For charts/graphs: "Extract all data values and reproduce as a table"
- For documents: "Transcribe all text exactly, preserving layout"
- For comparisons: provide reference image + target image
- **2026 note:** Gemini 3.5 has best native multimodal support; embed images, PDFs, audio, video directly

**⚠️ Multimodal injection risk:** Image-based Prompt Injection (IPI) embeds adversarial textual instructions visually within images (arXiv 2603.03637). Apply image re-encoding (JPEG re-encoding) and instruction/data separation for production multimodal systems.

---

## 17. COSTAR Framework
**When:** Content creation, marketing copy, system prompts, branded communication
**Structure:** Context · Objective · Style · Tone · Audience · Response
**Why it works:** Explicitly separates *style* (structure) from *tone* (emotion) — most frameworks treat these as one "voice" parameter, leading to off-brand outputs.

```
Context: [Situation or background]
Objective: [What the output must achieve]
Style: [How it's structured: listicle, narrative, technical doc]
Tone: [Emotional register: authoritative, friendly, urgent]
Audience: [Who reads this — expertise level, role]
Response: [Format + length]
```

---

## 18. RISEN Framework
**When:** Agentic workflows, tutoring, structured task execution, customer service agents
**Structure:** Role · Instructions · Steps · End goal · Narrowing
**Why it works:** The Steps component gives procedural agents a reliable execution sequence; Narrowing prevents scope creep.

```
Role: [Expert persona]
Instructions: [What to do — clear action verbs]
Steps: [Numbered sequence the AI must follow]
End goal: [Definition of a successful output]
Narrowing: [Scope constraints, exclusions, format]
```

---

## 19. Chain-of-Verification (CoV)
**When:** Fact-sensitive outputs, reducing hallucination, research tasks
**How:** After the initial answer, ask the model to generate verification questions, answer them independently, then revise the original answer based on findings.

```
Step 1: [Answer the original question]
Step 2: Generate 3–5 verification questions about your answer that would detect errors.
Step 3: Answer each verification question independently.
Step 4: Revise your original answer based on any contradictions found.
```

---

## 20. Multi-Agent Debate Prompting
**When:** High-stakes decisions, complex analysis, reducing bias
**How:** Multiple agents (or multiple calls) argue from different perspectives; a synthesizer integrates the debate into a final answer.
**Why it works:** Cross-validation catches errors; forced multi-perspective reduces sycophancy.

```
Agent A prompt: Argue FOR [position X]. Be rigorous and evidence-based.
Agent B prompt: Argue AGAINST [position X]. Challenge every assumption in Agent A's argument.
Synthesizer prompt: Given these arguments: [A] and [B], provide a balanced final recommendation.
```

**2026 note:** This is one of the five dominant production multi-agent patterns. Use A2A protocol for cross-vendor agent teams.

---

## 21. Context Engineering (Write / Select / Compress / Isolate)

**When:** Any production or agentic system; multi-turn agents; RAG pipelines
**Core idea:** Prompt engineering = what you say. Context engineering = everything the model sees.
**Key insight:** Most agent failures in 2025/26 are context failures, not model failures or prompt failures.

The four canonical context strategies (Anthropic, Sept 2025 — still the consensus reference in 2026):

| Strategy | What it means | When | Production implementation |
|----------|---------------|------|---------------------------|
| **Write** | Offload context to external memory/files (model writes its own scratchpad) | Long sessions, large artifacts | LangChain Deep Agents virtual filesystem; Sourcegraph scratchpad pattern |
| **Select** | Retrieve only relevant context for the current step | Knowledge-heavy tasks | RAG, vector search, MCP Tool Search (dynamic tool loading) |
| **Compress** | Summarize long histories, preserving decisions/bugs while discarding redundant tool outputs | Multi-turn agents | Claude Code `/compact`; LLM-driven history summarization |
| **Isolate** | Run sub-tasks in sub-agents with their own context windows | Parallel exploration, deep search | Claude Agent Skills; Opus 4.8 dynamic workflows; Deep Agents subagents |

See `model-guide.md → Context Engineering & Harness Engineering` for production patterns and memory systems.

---

## 22. Harness Engineering (2026) — a.k.a. "Horse Engineering"

**When:** Production agent fleets; any system where the prompt is one component of a larger runtime
**Core idea:** Prompt engineering (2022) → Context engineering (2024–25) → **Harness engineering** (early 2026, coined by Andrej Karpathy). The harness = prompt + context + tools + memory + runtime.
**Key insight:** Most agent failures in 2026 are *harness* failures, not model or prompt failures.

> **Why "harness"?** A harness is **horse tack** — the reins, saddle, and bit that channel
> a powerful but unpredictable animal in a useful direction. You don't make the horse safer;
> you make the horse's *power* safer by giving it a harness.
>
> The LLM is the horse (power, agency, unpredictability). The harness is the system around
> it (tools, memory, runtime, guardrails, verification, error handling) that channels that
> power toward useful work. Some practitioners colloquially call this "horse engineering"
> to emphasize the metaphor — same concept, different framing.
>
> **Important disambiguation:** Karpathy coined **context engineering** (Dec 2025) and
> **agentic engineering** (Feb 2026), NOT "harness engineering" — the term emerged
> separately in early 2026 from practitioner writing (Louis Bouchard, Augment Code, and
> others) and was popularized as the shorthand for Karpathy's "agentic engineering" thesis.

### Harness Engineering Checklist

1. **Reasoning step before every action** — force a plan/think step before tool calls
2. **Hard cap on tool calls** — explicit per-turn and per-task limits
3. **Tool schemas as public API contracts** — stable names, typed inputs, clear descriptions
4. **Context-first** — present a complete picture of the world to the agent
5. **Explicit exit conditions and failure modes** — define what "done" and "stuck" look like
6. **Runtime engineering over prompt tricks** — versioned, observable runtime configurations beat clever one-shot prompt wording for production reliability
7. **Middleware / hook architecture** — guardrails, logging, and approval gates *around* the model call (LangChain 1.0 middleware, OpenAI Agents SDK harness), not embedded in prose

### Production harness examples (2026)
- **Shopify River** — Slack-native coding agent operating in public; parallel agents + sequential critique loops
- **LangChain Deep Agents** — planning tool + virtual filesystem + subagents with isolated context windows (open source)
- **OpenAI Agents SDK** (April 15, 2026 update) — native sandbox + model-native harness, separating harness from compute
- **Anthropic Claude Agent SDK** — same tools, agent loop, and context management that power Claude Code, programmable in Python and TypeScript

---

## 23. Agent Skills Pattern (Anthropic, 2026)

**When:** You find yourself copy-pasting the same prompt across sessions; the capability needs bundled scripts/reference docs; multiple agents/teams need the same capability
**Core idea:** Modular, discoverable capability packages Claude can invoke on demand. Supported across Claude.ai, Claude Code, Claude Agent SDK, and Claude Developer Platform.

### Skill anatomy
```
my-skill/
├── SKILL.md          # YAML frontmatter (name + description) + instructions
├── scripts/          # Optional executable code
├── references/       # Optional docs loaded as needed
└── assets/           # Optional files used in output
```

### Three-level loading system
1. **Metadata** (name + description) — always in context (~100 words)
2. **SKILL.md body** — in context whenever skill triggers (<500 lines ideal)
3. **Bundled resources** — as needed (unlimited; scripts can execute without loading)

### Why this matters
- **Reusability** — write once, invoke anywhere
- **Discoverability** — model picks the right skill based on description
- **Modularity** — skill = isolated capability, easier to test and version
- **Composability** — multiple skills can chain in a single agent run

### Skill ecosystem (2026)
Anthropic Claude Skills (https://github.com/anthropics/skills), Vercel skills.sh, OpenAI Codex plugins, Cline plugins, MCP server directory. Treat skills as a first-class axis alongside MCP (tools) and A2A (agents).

---

## 24. MCP Tool Search / Dynamic Tool Loading (2026)

**When:** Agents with many MCP tools (>10% of context would be consumed by tool schemas); production systems where tool descriptions could be injection vectors
**Core idea:** Claude dynamically discovers and loads tool definitions on demand instead of loading all upfront.

**Announced:** January 14, 2026 by Thariq Shihipar (Anthropic). Originally Claude Developer Platform only ("advanced tool use" / "Tool search tool"); **now also available in Claude Code** as of late June 2026. Supports working with hundreds or thousands of tools.

### Why this matters
- **Saves context** — only relevant tool schemas are loaded
- **Does not break prompt caching** — deferred tools are excluded from the initial prompt
- **Security** — reduces attack surface by not exposing all tool descriptions to potential injection
- **Scales** — supports hundreds or thousands of tools without context bloat

### How to use it
For systems with Tool Search enabled, you can omit the tool list from the prompt entirely:
```
Task: [Goal]
Workflow:
1. [Action description — the harness will load the appropriate tool]
2. [Action description]
3. Synthesize results

Output: [structure]
```

The harness loads tool schemas dynamically as the agent decides which tools to use.

---

## 25. A2A Protocol for Cross-Vendor Agents

**When:** Multi-agent systems where agents come from different vendors (Anthropic + OpenAI + Google + open-weights); production deployments on Google Cloud
**Core idea:** A2A (Agent2Agent) is the agent-to-agent counterpart to MCP's agent-to-tool. Initiated by Google Cloud; surpassed 150 organizations in its first year.

### Relationship to MCP
- **MCP** (Linux Foundation / AAIF) — agent-to-tool
- **A2A** (Google Cloud / Linux Foundation) — agent-to-agent

Use both: MCP for tools, A2A for cross-vendor agent teams.

### Native support
- Google ADK (Agent Development Kit) — adk.dev
- Major cloud platforms

### A2A Multi-Agent Template
```
## Orchestrator Agent (Claude Opus 4.8)
Goal: [complex task requiring multiple specialists]

## Specialist Agents (via A2A)
- Research Agent (Gemini 3.5): gathers and summarizes sources
- Analysis Agent (GPT-5.6): applies analytical framework
- Writing Agent (Claude Opus 4.8): drafts final deliverable

## Handoff Protocol (A2A)
- Orchestrator → Research: {task, scope, output_format}
- Research → Analysis: {sources, summaries, gaps}
- Analysis → Writing: {findings, recommendations, audience}
- Writing → Orchestrator: {deliverable, citations, open_questions}
```

---

## 26. Subagent Orchestration (2026)

**When:** Codebase-scale migrations; long-running complex tasks; parallel exploration
**Core idea:** A single agent run spawns multiple subagents, each with isolated context, working in parallel. The parent agent aggregates results.

### Production realizations (2026)
- **Claude Opus 4.8 dynamic workflows** (research preview, Claude Code) — can spawn hundreds of parallel subagents in one session
- **GPT-5.6 ultra mode** — uses subagents to accelerate complex work
- **Kimi K2.6 Agent Swarm** — scales to 300 sub-agents / 4,000 coordinated steps
- **DeerFlow 2.0** (ByteDance, June 23, 2026) — open-source super-agent harness built on LangGraph/LangChain with sandboxed execution, persistent memory, skills, and sub-agents
- **LangChain Deep Agents** — async subagents with isolated context windows (open source)
- **Grok 4.20 multi-agent variants** — multi-agent mode

### Pattern
```
## Parent Agent Goal
[Complex task that can be decomposed]

## Subagent Plan
- Subagent 1: [sub-task 1] — independent context
- Subagent 2: [sub-task 2] — independent context
- Subagent 3: [sub-task 3] — independent context

## Aggregation
Parent agent receives outputs from all subagents and synthesizes final answer.
Each subagent's intermediate state is isolated — no context pollution.
```

---

## 27. DSPy / Automated Prompt Optimization

**When:** Production systems with measurable outcomes; scale > 1,000 prompt executions/day
**How:** Treat prompts as parameters to optimize, not hand-crafted artifacts. DSPy (Stanford NLP) automatically finds optimal prompt wording and few-shot examples given a training set and metric.
**Use when:** Manual iteration is too slow; you have labeled examples; prompts need to survive model updates.
**Reference:** https://github.com/stanfordnlp/dspy

### Related approaches (2026)
- **TextGrad** — gradient-based prompt optimization
- **PromptBreeder / EvoPrompt** — evolutionary prompt search
- **GEPA** (Generate-Reflect-Critique) — 2026 prompt optimization via reflective feedback
- **LLM-as-judge** — mature pattern via DeepEval/Promptfoo; use a strong LLM to grade outputs
- **Prompt management platforms** — Vellum, Humanloop, Orq.ai, Portkey, Langfuse Prompts,
  Microsoft Promptflow, Pezzo — production prompt versioning + A/B testing + rollout
- **Layered eval pipelines (2026 trend):** Fast property checks in CI (DeepEval/Promptfoo
  asserts) → quality metrics pre-release (DeepEval 50+ metrics + LLM-as-judge) → red-team
  + adversarial pre-deploy (Promptfoo red-team / PyRIT / Garak / Giskard)

### When to use automated optimization
- You have labeled examples and a measurable metric
- Manual iteration has plateaued
- Prompt needs to survive model updates (DSPy can re-optimize automatically)
- High-volume production (>1,000 executions/day) where small improvements compound

---

## 28. Anti-Slop Prompting (image/video)

**When:** Image generation (gpt-image-2, Nano Banana Pro, FLUX.2, Ideogram 4.0) and video generation (Veo 3.1, Gemini Omni, Kling 3.0)
**Core idea:** Replace vague aesthetic words with concrete visual facts. Reasoning-based image/video models respond to **visual parameters, not vibes**.

### The substitution table

| Don't say | Say instead |
|---|---|
| "stunning" | "overcast daylight, shallow depth of field" |
| "epic" | "low-angle shot, wide 24mm lens" |
| "cinematic" | "anamorphic 2.39:1, teal-and-orange grade, lens flare" |
| "realistic" | "shot on Canon R5, 85mm f/1.4, natural window light" |
| "high quality" | "8K texture detail, film grain ISO 400" |
| "dramatic" | "chiaroscuro lighting, deep shadows, single source" |
| "beautiful" | "[specific aesthetic: golden hour, rim lighting, 35mm]" |

### Why it works
Reasoning-based models (gpt-image-2, Nano Banana Pro, Veo 3.1) interpret language literally. Vague aesthetic words don't map to specific visual outputs; concrete visual facts do. This generalizes across every frontier image/video tool in 2026.

---

## 29. Personalization-Driven Prompting (Midjourney V8+)

**When:** Midjourney V8 / V8.1 (default since June 10, 2026); emerging pattern in other tools
**Core idea:** The personalization profile (rated taste) is the primary creative input; the prompt supplies subject/context.

### V8.1 prompting shifts (major)
- Personalization profile is the primary creative input
- **Longer prompts now work** (reverses V7's "concise is better"). Trend toward 40-word prompts.
- Push `--stylize` to **1000** with a trained profile
- `--cref` / `--oref` / `--q 4` **removed** in V8.1. Use Personalization + srefs for character/object consistency.
- HD (native 2K) is now default and cheap

### Template
```
[Subject and action], [environment and context], [specific details to preserve],
[lighting and atmosphere] --s 1000 --p --ar 16:9
```

### Personalization profile progression
- Activates at 40 ratings
- Stabilizes at ~200 ratings
- Improves to ~2,000 ratings

### When NOT to use this pattern
- For models without personalization (gpt-image-2, Nano Banana Pro, FLUX.2) — use structured slot templates instead
- For tools that don't support `--sref` or equivalent
- For users who haven't trained a profile (V8 runs at a fraction of capability without ≥200 ratings)

---

## 30. Voice/Music Stage-Direction Prompting (2026)

**When:** Voice generation (Eleven v3, Cartesia Sonic 3.5, Hume EVI 3, Sesame CSM),
music generation (Suno 5.5, Lyria 3 Pro), talking-avatar video (Hedra, HeyGen).
**Core idea:** Treat voice/music prompts like a **script with stage directions**, not
a content description. Voice models in 2026 respond to directed emotion cues, delivery
markers, and pacing directives — not vague affect words.

### TTS prompt pattern (script with stage directions)
```
[Voice: warm maternal, mid-40s, North American]
[Pace: 165 wpm; pause 0.4s at commas, 0.8s at periods]
[Emotion arc: starts reassuring → builds confidence → ends proud]

"This isn't your fault. [breath] Let's walk through what happened,
step by step. [pause 0.5s] By the end you'll see exactly what to fix."
```

### Music prompt pattern (concrete musical facts)
```
[BPM: 124] [Key: A minor] [Time signature: 4/4]
[Genre: UK garage with ambient pads]
[Instrumentation: Roland TR-808 drums, Reese bass, Juno-106 strings]
[Vocal: female, breathy, harmonized in thirds on the chorus]
[Production: 2020s London, sidechain compression, wide stereo image]
[Mood: melancholic but danceable]
[Structure: intro 8 bars → verse 16 → chorus 8 → drop]
```

### Anti-slop rule for audio (mirrors the image anti-slop rule)
| Don't say | Say instead |
|---|---|
| "upbeat" | "124 BPM, four-on-the-floor, major key" |
| "cinematic" | "orchestral strings + taiko drums, 60 BPM, building crescendo" |
| "energetic" | "165 BPM, distorted 808 kicks, sidechained Reese bass" |
| "calm" | "70 BPM, fingerpicked acoustic guitar, soft pad, no drums" |
| "narrator voice" | "mid-40s, North American, warm maternal, 165 wpm, reassuring" |
| "dramatic pause" | "[pause 1.2s]" or "[breath] [pause 0.6s]" |

### Per-tool tips
| Tool | Tip |
|------|-----|
| **Eleven v3** | Use directed emotion tags (`[whispered]`, `[urgent]`, `[laughing]`); multi-speaker control via speaker labels |
| **Cartesia Sonic 3.5** | Optimized for <100ms latency — keep prompts short; use stable-voice mode for agents |
| **Hume EVI 3** | Specify any voice profile (real or designed); no fine-tuning required |
| **Sesame CSM** | Open-weights; for local inference use markdown-like block formatting |
| **Suno 5.5** | Provide structure (intro/verse/chorus/drop); use "Custom Models" to fine-tune on your aesthetic |
| **Lyria 3 Pro** | Image + text input — combine a mood image with BPM/genre text for best results |
| **Hedra** | Upload a clear face image (front-facing, visible features); provide script with pacing markers |

---

## 31. Loop Engineering (June 2026)

**When:** Any agent that should run on a schedule, on an event, or until a verifiable goal
is met — without a human prompting it each iteration. Production agents that need to
"run themselves" (CI fixers, monitoring agents, research scrapers, content pipelines,
codebase janitors, overnight batch processors).
**Core idea:** Loop engineering is the next layer above harness engineering. The harness
equips a *single* agent run; the loop is the *recurring control system* that triggers,
verifies, and stops many agent runs.

> **Definition (Addy Osmani, June 8, 2026):** *"Loop engineering is replacing yourself as
> the person who prompts the agent. You design the system that does it instead."*

> **Definition (LangChain):** *"The core agent algorithm is simple: give the LLM context
> and let it call tools in a loop until it's done. This is the most fundamental loop."*

> **Harness vs Loop (Cobus Greyling):** *"The harness equips a single agent run; the loop
> is what keeps poking agents on a schedule, spawning helpers, and feeding itself."*

### Loop anatomy (six components)

```
1. Trigger         — what starts the loop (cron schedule, webhook, file watch, user request)
2. Verifiable goal — the SYSTEM (not you) decides when done — must be machine-checkable
3. Agent invocation — harness (prompt + context + tools + memory) for this run
4. Verification    — does the output meet the goal? (tests pass? diff empty? score > threshold?)
5. Iteration       — if not done, re-prompt with what was learned (NOT from scratch)
6. Stopping condition — success | max iterations | max cost | human escalation | timeout
```

### Loop topologies

| Topology | When to use | Example |
|----------|-------------|---------|
| **Single-shot loop** | One trigger → one agent run → one verification → done (basic ReAct) | Customer support reply |
| **Memory-aware loop** | Long-running tasks; agent persists state across iterations | Multi-step research task |
| **Scheduled loop** | Cron-triggered | "Every morning, review yesterday's CI failures and open PRs to fix them" |
| **Event-driven loop** | Triggered by external events | New GitHub issue → triage and assign |
| **Nested loop** | Outer loop supervises; inner loops do sub-tasks | Outer: "maintain codebase health"; Inner: "fix this specific issue" |
| **Fan-out loop** | Outer loop spawns N parallel sub-loops, aggregates | Research 10 competitors in parallel, then synthesize |
| **Self-feeding loop** | Loop's output becomes its next input | Research → outline → draft → review → publish |

### The verification problem (biggest unsolved challenge)

**70–95% of agents fail in production** (Fiddler AI, 2026). The biggest reason: verification
is hard. Common failure modes:

- **Verification gaming** — agent claims success without actually achieving the goal
- **Goal drift** — loop's understanding of the goal subtly shifts across iterations
- **Runaway loops** — never converges on success, burns budget indefinitely
- **Context pollution** — loop's context grows until it loses track of original goal
- **State loss** — mid-run crash loses all progress

**Mitigations:**
1. **Machine-checkable success criteria** — "Fix bug #1234" is NOT verifiable; "tests for
   #1234 pass and diff is non-empty" IS.
2. **Independent verifier** — use a *separate* model or agent to verify, not the same one
   that produced the output.
3. **Human-in-the-loop** for high-stakes or low-confidence iterations.
4. **Never trust agent self-reports** — always check the actual world state.
5. **Hard caps** — max iterations, max tokens, max cost per loop, max wall-clock time.

### Loop engineering checklist

1. **Machine-checkable goal** — "Fix bug #1234" is not; "tests for #1234 pass and diff
   is non-empty" is
2. **Hard caps** — max iterations, max tokens, max tool calls, max cost per loop
3. **Verification independence** — use a separate model/agent to verify, not the same one
4. **Permission tiers** — read-only / write-with-rollback / write-without-rollback /
   destructive (escalate each tier to human approval)
5. **Observability** — log every iteration, every tool call, every verification result
   (LangSmith, Langfuse, MLflow, Arize Phoenix, Helicone)
6. **Idempotency** — re-running the loop on the same input should produce the same output
7. **Backpressure** — if the loop falls behind (events pile up), shed load or queue
8. **Human escalation path** — define exactly when and how the loop asks for help
9. **Cost guardrails** — per-loop and per-day cost caps; alerting on runaway spend
10. **State durability** — if the loop crashes mid-run, can it resume? (durable state store)

### Loop engineering template

```
## Loop Name
[descriptive name]

## Trigger
[cron schedule | event source | manual]

## Verifiable Goal
[Machine-checkable success criterion — NOT "fix the bug" but
 "tests for the bug pass + diff is non-empty + lint clean"]

## Agent Harness (per-iteration)
- Role: [role]
- Tools: [list]
- Context: [what to load each iteration]
- Memory: [in-context | external store]

## Verification
- Method: [test suite | diff check | independent verifier agent | score threshold]
- Independence: [same model | different model | human]
- Failure behavior: [retry | refine | escalate | stop]

## Stopping Conditions
- Success: [verifiable criterion]
- Max iterations: [N]
- Max cost: [$X or N tokens]
- Timeout: [N minutes]
- Human escalation: [conditions that require human review]

## Cost Guardrails
- Per-iteration cap: [N tokens / $X]
- Per-loop cap: [N tokens / $X]
- Per-day cap: [N tokens / $X]
- Alert threshold: [X% of cap]

## Observability
- Log every: [iteration | tool call | verification result]
- Dashboard: [LangSmith | Langfuse | MLflow | Arize Phoenix]
- Alert on: [iteration > N | cost > X | verification failure rate > Y%]
```

### Example: scheduled CI-fix loop

```
## Loop Name
CI Failure Auto-Fixer

## Trigger
Cron: 0 9 * * * (every day at 09:00)

## Verifiable Goal
For each red build in the last 24h:
  - Identify root cause
  - Open a PR that fixes it
  - CI on the PR is green
  - PR is assigned to a human reviewer

## Agent Harness (per-iteration)
- Role: Senior engineer familiar with [tech stack]
- Tools: gh CLI, git, test runner, codebase search, web search for error messages
- Context: failing build log + relevant source files + recent commits to affected files
- Memory: Letta (cross-session memory of past similar failures and fixes)

## Verification
- Method: independent verifier agent (different model) checks:
  (a) PR diff is non-empty
  (b) CI on PR branch passes
  (c) PR description references the build ID
- Independence: verifier is a different model than the fixer
- Failure behavior: if verification fails twice, escalate to human

## Stopping Conditions
- Success: all red builds from last 24h have a green PR OR explicitly marked as "needs human"
- Max iterations per build: 5
- Max cost per build: $2
- Timeout: 30 minutes per build
- Human escalation: build references security, infra, or production-only code

## Cost Guardrails
- Per-iteration cap: 10K tokens / $0.20
- Per-loop cap (per build): 50K tokens / $2
- Per-day cap: 500K tokens / $20
- Alert threshold: 80% of any cap

## Observability
- Log every: iteration, tool call, verification result, PR opened/closed
- Dashboard: LangSmith
- Alert on: iteration > 5, cost > $2/build, verification failure rate > 30%
```

### Loop engineering tools (2026)

- **LangChain / LangGraph 1.0** — `AgentExecutor` and LangGraph state machines for loop orchestration
- **LangSmith / Langfuse** — observability for agent loops (every iteration, every tool call)
- **Temporal / Inngest / Trigger.dev** — durable execution for long-running loops with state
- **Modal / E2B / Daytona** — sandboxed execution environments for loop agents
- **MLflow / Arize Phoenix / Helicone** — production monitoring of agent loops

### Key sources

- Addy Osmani, "Loop Engineering" (June 8, 2026) — addyosmani.com/blog/loop-engineering
- LangChain, "The Art of Loop Engineering" — langchain.com/blog/the-art-of-loop-engineering
- Adnan Masood, "Loop Engineering: A Guide for Engineers and Practitioners" (June 24, 2026) — medium.com/@adnanmasood
- Cobus Greyling, "Loop Engineering" — cobusgreyling.substack.com/p/loop-engineering
- MindStudio, "What Is Loop Engineering? The New Meta for AI Coding Agents"
- Firecrawl, "Should You Stop Prompting Agents and Start Designing Loops?"
- Requesty.ai, "Loop Engineering: How to Build AI Agent Loops That Run Themselves"

### When NOT to use Loop Engineering

- **One-shot tasks** — if a human is going to prompt the agent once and review the output, you don't need a loop. Just use prompt + context engineering.
- **Tasks without verifiable goals** — if you can't define a machine-checkable success criterion, a loop will run forever (or game the verification). Use human-in-the-loop instead.
- **High-stakes / irreversible actions** — don't automate destructive actions in a loop. Keep humans in the loop for refunds, deletions, deploys, etc.
- **Low-volume tasks** — if the task only runs once a month, the engineering cost of building a loop may exceed the savings.

---

## Deprecated / Superseded Techniques (2026)

| Technique | Status | Replacement |
|-----------|--------|-------------|
| **"Be honest" coaxing (Claude 4.6)** | Deprecated | Anti-sycophancy is now a training goal in Opus 4.8 — invite it instead |
| **"Think step by step" on reasoning models** | Counterproductive | Set reasoning effort via API parameter |
| **Midjourney `--cref` / `--oref`** | Removed in V8.1 | Use Personalization + srefs |
| **Sora prompting guides** | Discontinued | Migrate to Veo 3.1, Kling 3.0, Seedance 2.0, Grok Imagine 1.5 |
| **AutoGen** (Microsoft) | Maintenance mode | Microsoft Agent Framework (Oct 2025) |
| **LangChain "Agent Builder"** | Rebranded | LangSmith Fleet (Feb 2026) |
| **Claude Code SDK** | Renamed | Claude Agent SDK (June 15, 2026 — separate SDK credit) |
| **Llama as Meta flagship** | Superseded | Muse Spark (April 9, 2026) — Meta pivoted to proprietary |
| **6 multi-agent patterns (strict)** | Consolidated | 5 dominant production patterns (fan-out, pipeline, debate, supervisor, swarm) |
| **Imagen 4 (Google)** | End-of-life | Nano Banana Pro (Gemini 3 Pro Image) — Imagen 4 shuts down Aug 17, 2026 |
| **Ideogram 3.0 (closed)** | Superseded | Ideogram 4.0 (June 3, 2026 — open-weight 9.3B) |
| **Stable Diffusion 3.5 (dominant OS image)** | Displaced | FLUX.2 family + Ideogram 4.0 now lead open-weight image gen |
