# Context Engineering: A Deep Research Brief

*A practitioner-first, academically-rigorous survey of the 2024–25 discipline that reframes how production AI agents are fed — the systematic curation of what an LLM sees on each call, structured around four canonical operations: Write, Select, Compress, Isolate.*

| Field | Value |
|---|---|
| **Paradigm coinage date** | Late 2024 in practitioner usage; **crystallized September 2025** by Anthropic's essay *"Effective Context Engineering for AI Agents"*; endorsed by Andrej Karpathy on June 25, 2025 ("+1 for 'context engineering' over 'prompt engineering'") and by Shopify CEO Tobi Lütke the same month |
| **Coined by / crystallized by** | Term in informal use by late 2024; crystallized as a formal discipline with a canonical four-operation vocabulary by **Anthropic (Sept 2025)** and **LangChain (parallel blog post, "Context Engineering for Agents")**; amplified by Karpathy (June 25, 2025 tweet) |
| **Layer below** | Prompt engineering (2022) — the crafting of the instruction itself |
| **Layer above** | Harness engineering (Feb 2026, attributed to Mitchell Hashimoto and amplified by Andrej Karpathy, who joined Anthropic in May 2026) [UNVERIFIED — check primary source for exact coinage attribution; some 2026 sources attribute the harness-engineering framing solely to Karpathy] — the per-execution system of prompt + context + tools + memory + runtime |
| **Status as of July 2026** | Mature consensus layer. The Write/Select/Compress/Isolate vocabulary is the production default. Subsumed (not replaced) by harness engineering; most agent failures in 2026 are still traceable to a context-engineering failure underneath a harness failure. Academic formalization is catching up (arXiv:2507.13334, arXiv:2603.09619, arXiv:2604.04258). |

---

## 2. TL;DR

Context engineering is the discipline of designing **what the model sees on each call** — not how the instruction is phrased (that is prompt engineering) and not the runtime that invokes the model (that is harness engineering), but the curated information that flows into the context window on every forward pass. The framing emerged from a simple observation: by mid-2024, production agents were failing not because the prompt was poorly worded or the model was dumb, but because the model was either starved of the document it needed or buried under a haystack of irrelevant context. Anthropic's September 2025 essay *"Effective Context Engineering for AI Agents"* crystallized the response around four canonical operations that remain the consensus vocabulary in 2026: **Write** (offload to external memory the model itself writes to), **Select** (retrieve only the chunks relevant to this step), **Compress** (summarize history so prior decisions survive but raw tool dumps die), and **Isolate** (run sub-tasks in sub-agents with their own context windows so pollution in one branch does not bleed into another).

The four operations are not alternatives; they compose. A production research agent on turn 30 of a long session will simultaneously Write intermediate findings to a virtual filesystem (LangChain Deep Agents), Select fresh chunks from a vector store for the current sub-question, Compress the prior 29 turns into a 400-token brief, and Isolate each parallel exploration thread in its own sub-agent. The economics are unforgiving: a 1M-token context window (Claude Opus 4.8, GPT-5.6, Gemini 3.5, GLM-5.2 all ship ≥1M in 2026) does not eliminate the discipline, because the "lost in the middle" attention-dilution effect (Liu et al., arXiv:2307.03172) and the linear cost of input tokens mean a stuffed window is both dumber and more expensive than a curated one. Prompt caching (Claude prompt caching, GPT-5.6 explicit cache breakpoints — 30-minute minimum cache life, ~90% discount on cache reads) rewards engineers who structure context with stable prefixes and volatile suffixes; the engineer who dumps fresh data into the middle of the prefix pays full price on every call.

The memory ecosystem matured in lockstep: **Letta** (formerly MemGPT; self-editing memory plus sleep-time compute), **Mem0** (vectors plus knowledge graph), **Zep** (temporal knowledge graph via Graphiti), **Cognee**, **Redis LangCache** (semantic caching for cost), and **HippoRAG 2** (10–30× cheaper than iterative GraphRAG, 6–13× faster) each implement different combinations of the four operations against different failure modes. On the tooling side, **MCP Tool Search** (Anthropic, January 14, 2026) is the canonical Select operation for the tool layer — dynamically loading tool schemas only when needed, cutting MCP context bloat by up to 85% in measured A/B tests. Context engineering is necessary but not sufficient: harness engineering sits above it and adds tools, memory, runtime, and middleware; loop engineering sits above harness engineering and adds the recurring control system. A 2026 production agent needs all four layers, and a context failure underneath a perfectly-designed harness is still a failure.

---

## 3. The Four-Layer Stack

Context engineering is the second layer in a four-year, four-layer stack that practitioners in mid-2026 treat as the canonical mental model for agent engineering. The stack runs **prompt engineering (2022) → context engineering (2024–25) → harness engineering (Feb 2026) → loop engineering (June 2026)**, and each layer was invented to solve a failure class the layer below could not. The full derivation of the stack is in the companion Loop Engineering brief at `/home/z/my-project/download/loop-engineering-deep-research.md` Section 3; this section locates context engineering within it without re-deriving the whole evolution.

**Below context engineering sits prompt engineering.** Prompt engineering is the discipline of crafting the instruction — assign a role, give examples, specify the output format, constrain negatively. Its failure class is *vagueness*: a model given "write a blog post about AI" produces a meandering essay with no audience, length, or tone. Even after context engineering matured, prompt engineering remained table stakes; Opus 4.8's contract-style XML prompts and GPT-5.6's markdown-header prompts are direct descendants. The layer relationship is strict inclusion: a context engineer who ignores prompt phrasing produces a well-fed agent that still produces vague output, and a prompt engineer who ignores context produces a beautifully-phrased instruction the model cannot ground.

**Context engineering itself is the second layer.** It solves the failure class of *context starvation* and *context pollution*: a perfectly-phrased prompt fails if the model does not have the document it needs to answer, or if the context window is so cluttered with irrelevant material that the model loses the thread. Anthropic's September 2025 essay framed it crisply — the bottleneck for agent quality moved from "what you say" to "what the model sees." The four canonical operations (Write, Select, Compress, Isolate) are the engineer's vocabulary for curating that visible information, and the rest of this brief is about them.

**Above context engineering sits harness engineering (February 2026).** Harness engineering, widely attributed to Mitchell Hashimoto and amplified by Andrej Karpathy (who joined Anthropic in May 2026 to push it further) [UNVERIFIED — check primary source; some 2026 sources attribute the framing to Karpathy alone, others to Hashimoto with Karpathy amplifying], solves *single-run agent fragility*: even with a perfect prompt and a perfectly-engineered context, a single agent run fails if it lacks the right tools, the right memory, the right runtime, the right middleware (guardrails, logging, approval gates), and explicit exit conditions. The harness is the per-execution system around one agent run — prompt + context + tools + memory + runtime + middleware — and context engineering is one component of it. The harness engineer composes the context engineer's output with a tool layer, a memory layer, and a runtime; the loop engineer (above the harness) then composes many harness invocations into a recurring control system.

**The stacking is non-replacing.** A loop with a broken harness fails on every iteration; a harness with polluted context produces a confidently-wrong agent; a well-contextualized agent with a vague prompt still produces generic output. Production systems in 2026 need all four layers, and the discipline of the harness engineer explicitly includes the engineering of context, which itself includes the engineering of the prompt. The naive reading — that harness engineering or loop engineering "replaces" context engineering — is wrong; they automate the invocation of a well-engineered context, which is faster failure if the context is broken, not slower failure. The loop engineering brief's Section 15 comparison table (reproduced and extended in Section 15 of this brief) makes the stacking explicit across nine dimensions.

The practical implication for a context engineer in mid-2026 is that the four operations are no longer the ceiling of the discipline — they are the floor. A context engineer who stops at "I selected the right chunks and compressed the history" has done necessary work but has not yet designed a system that can run on a schedule, recover from crashes, verify its own output, or stop itself. Those concerns have moved up the stack. But the context layer remains the layer where most 2026 agent failures actually originate, because a harness or loop cannot rescue a model that is fed the wrong thing. The four-layer model is, in this sense, a responsibility map: every layer owns a failure class, and the context engineer owns the failure class of "the model saw the wrong stuff."

---

## 4. Definition & First Principles

### 4.1 The verbatim definitions

**Anthropic ("Effective Context Engineering for AI Agents," September 2025):** the essay's central framing is that *"the mere existence of an environment variable called a context window does not mean that its contents are useful to the model"* and that intelligent agents need to be architected around techniques that *"prevent context pollution"* across extended time horizons. Anthropic groups the discipline around four operations — writing to external memory, selecting relevant context, compressing history, and isolating sub-tasks into sub-agents — and treats them as the engineer's palette for the entire agent lifetime. The essay is the single most-cited primary source in 2026 writing on the topic, and the four operations it names are the consensus vocabulary across both vendor docs (LangChain, Anthropic, OpenAI) and academic surveys (arXiv:2507.13334, arXiv:2603.09619).

**LangChain ("Context Engineering for Agents," parallel blog post, 2025):** LangChain's framing is operational and explicitly adopts the same four-bucket taxonomy: *"We group common strategies for agent context engineering into four buckets — write, select, compress, and isolate — and give examples of how popular agents and papers implement each."* LangChain accompanies the post with an open repository (`github.com/langchain-ai/context_engineering`) that walks through how each operation is implemented against real agent codebases. The convergence between Anthropic's essay and LangChain's parallel post — both arriving at the same four operations independently within weeks of each other — is what made the vocabulary consensus rather than one vendor's preferred framing.

**Andrej Karpathy (X/Twitter, June 25, 2025):** Karpathy's endorsement predated the Anthropic essay by roughly three months and gave the term its practitioner currency: *"+1 for 'context engineering' over 'prompt engineering'. It describes the core skill better: the art of providing all the context for the task."* The framing — that prompt engineering was a misnomer because the real skill is curating context, not wording instructions — is what made "context engineering" the default term in late-2025 practitioner discourse, and what made the September Anthropic essay land as the crystallization of an already-brewing consensus rather than a vendor coinage. Shopify CEO Tobi Lütke endorsed the same framing the same month, giving the term enterprise credibility.

### 4.2 Synthesis: what context engineering IS

Context engineering is the discipline of designing, structuring, and managing the entire informational input an LLM receives on a given call — system instructions, working memory, retrieved knowledge, conversation history, and tool outputs — such that the model has exactly the information it needs to take the next action and nothing that would dilute attention, pollute the goal, or inflate cost. Its artifacts are not prompts (those belong to prompt engineering) and not runtimes (those belong to harness engineering), but **context specifications** — declarative descriptions of what is loaded into the window on each call, how it is ordered, what is cached, what is compressed, what is offloaded, and what is isolated. The discipline applies to every LLM call but becomes load-bearing in multi-turn agents, RAG pipelines, and any system where the model is invoked repeatedly with mutating state.

### 4.3 What context engineering is NOT

It is not "prompt engineering with documents." A prompt engineer who injects a PDF and calls it context engineering has missed the discipline: the question is not whether a document is in the window but whether it is the right chunk, whether the prior turn's tool output should still be there, whether the system prompt is cacheable, and whether the sub-task should run in a sub-agent. It is not "RAG" — RAG is one Select implementation among many, and a context engineer who equates the two omits Write, Compress, and Isolate entirely. It is not "long context windows eliminate the need for it" — the 1M-token era (Section 7) makes context engineering more important, not less, because attention dilution and input-cost economics punish the engineer who treats a large window as a license to dump. And it is not "harness engineering" — the harness wraps the context with tools, memory, runtime, and middleware; the context engineer's deliverable is one input to the harness, not the harness itself.

### 4.4 The four first principles

**First principle: what is in the context window matters more than how the instruction is phrased.** This is Karpathy's June 2025 framing, and it is the discipline's foundational claim. A model with the wrong document and a perfectly-worded instruction will fail; a model with the right document and a clumsily-worded instruction will usually succeed. The implication is that the highest-leverage interventions on agent quality are almost always context interventions — adding the missing chunk, removing the distracting one, compressing the bloated history — not prompt rewording. This is empirically testable: in ablation studies on agent benchmarks, swapping a well-engineered context for a naive one typically moves accuracy more than swapping a well-engineered prompt for a naive one, especially on multi-step tasks where the model must ground each step.

**Second principle: context is finite and must be curated, not dumped.** Even in the 1M-token era, the context window is a finite resource, and the model's effective use of it degrades as it fills — the "lost in the middle" attention-dilution effect documented by Liu et al. (arXiv:2307.03172) and corroborated across model families in 2025–26 (Stanford, Google, Anthropic, and Meta report 13.9% to 85% accuracy drops as context grows, per the diffray.ai aggregation). Curation is the discipline's stance: the engineer's default is to include less, not more, and to justify each addition against the cost it imposes on attention and on the wallet. The naive posture — "load everything, the window is big enough" — is the dominant cause of context pollution, which is in turn the dominant cause of long-session agent failure.

**Third principle: the four operations (Write, Select, Compress, Isolate) are the consensus vocabulary.** Both Anthropic (September 2025) and LangChain (parallel post) independently arrived at the same four-bucket taxonomy, and every 2026 academic survey and production framework has converged on it. The convergence is not a coincidence: the four operations map cleanly to the four ways an engineer can move information relative to the context window — out of it (Write), into a relevant subset of it (Select), down to a smaller representation of it (Compress), or into a parallel instance of it (Isolate). Section 5 walks each operation in depth; the principle here is that the four operations are the engineer's complete palette, and a context-engineering plan that does not use at least one of them on a multi-turn agent is almost certainly incomplete.

**Fourth principle: context engineering is necessary but not sufficient.** A perfectly-engineered context still fails if the harness around it is broken — if the agent has no exit conditions, no approval gates, no observability, no memory beyond the window. Harness engineering (Feb 2026) sits above context engineering precisely because the four operations cannot, by themselves, address single-run agent fragility. The context engineer's deliverable feeds the harness engineer's composition; the harness engineer's deliverable feeds the loop engineer's recurring control system. The principle has a practical corollary: when an agent fails in production, the first hypothesis is a context failure, the second is a harness failure, the third is a loop failure, and only after all three are cleared should the engineer blame the model. The 70–95% production agent failure rate cited by Fiddler AI and corroborated across 2026 industry surveys is, at root, a context-plus-harness failure rate — the models are usually fine.

---

## 5. The Four Canonical Operations — Write / Select / Compress / Isolate

This is the heart of the brief. The four operations are the engineer's complete palette, and a context-engineering plan that omits any of them on a long-running agent is almost certainly incomplete. Each operation solves a different failure mode, has a different production implementation, and has a characteristic way it fails when misapplied. The sections below walk each in the same structure: definition, problem solved, when to use, when not to use, a worked example, production tools, and the characteristic failure mode. A summary table precedes the deep dives.

### 5.0 Summary table

| Operation | One-sentence definition | Problem it solves | Production implementations | Characteristic failure mode |
|---|---|---|---|---|
| **Write** | The model offloads information from the context window into an external store it can re-read | Window overflow on long sessions; loss of intermediate state across turns | LangChain Deep Agents virtual filesystem; Claude Code scratchpad; Letta self-editing memory; Sourcegraph scratchpad pattern | Write-then-forget: model writes to store but never re-reads, so the store becomes an information graveyard |
| **Select** | The system retrieves only the chunks relevant to the current step | Context starvation (model lacks needed doc) and pollution (model has too much irrelevant doc) | RAG over vector DBs (Pinecone, Weaviate, Chroma, pgvector); hybrid vector+BM25 search; MCP Tool Search for tool schemas | Bad chunks / stale indices / irrelevant retrieval — the model is grounded in the wrong document and is confidently wrong |
| **Compress** | The system summarizes prior history so decisions survive but raw tool dumps die | History grows until OOM; attention dilution in long sessions | Claude Code `/compact`; LangGraph message summarization; Anthropic Cookbook compaction patterns; LLM-driven history summarization | Information loss: the summary drops the one fact the next turn needed; or over-compression produces a vague brief |
| **Isolate** | Sub-tasks run in sub-agents with their own context windows; only the result returns | Pollution in one branch bleeds into another; parallel exploration inflates the parent window | Claude Agent Skills; LangChain Deep Agents subagents; Opus 4.8 dynamic workflows; sub-agent fan-out patterns | Result-quality loss: the sub-agent's compressed return loses nuance; or the orchestrator can't verify the sub-agent's work |

### 5.1 Write — offload to external memory the model itself authors

**Definition.** Write is the operation of moving information *out of* the context window into an external store that the model can re-read on demand, where the model itself is the author of what is written. The model writes its own scratchpad, its own intermediate findings, its own to-do list, its own partial artifact — and the context window holds only the current step's working set, not the full history of the run.

**Problem it solves.** Write solves window overflow on long sessions and the loss of intermediate state across turns. A research agent that runs for 30 turns accumulates tens of thousands of tokens of intermediate findings, hypotheses, and partial drafts; if all of it stays in the window, the agent either hits the cap or suffers attention dilution. Write lets the agent persist those intermediate findings to a file, a memory object, or a database, and re-load only the slice that the next step needs. It also solves the cross-session continuity problem: a finding written to a file survives a session restart, while a finding held in the context window does not.

**When to use it.** Write is the right answer whenever an agent is producing a long artifact (a report, a codebase, a research brief), whenever a session will span many turns, whenever intermediate state should survive a crash or a context compaction, and whenever the same finding will be re-referenced multiple times across the run. LangChain Deep Agents' virtual filesystem is the canonical 2026 implementation: the agent writes its plan, its intermediate findings, and its draft outputs to a virtual filesystem, and reads back only what the current step needs. The pattern is so load-bearing that Deep Agents ships it as a default rather than an opt-in.

**When NOT to use it.** Write is unnecessary on one-shot tasks where there is no intermediate state worth persisting. It is also counterproductive when the write-read latency exceeds the cost of just keeping the information in context — for a 200-token finding that will be used on the next turn, the round-trip through a file is pure overhead. And it is harmful when the model writes but never re-reads: a scratchpad that accumulates without being consulted becomes an information graveyard that costs tokens to write and provides no value, which is the characteristic Write failure mode.

**Worked example.** A code-migration agent tasked with converting a 50-file Python 2 codebase to Python 3 should Write its per-file migration notes to a virtual filesystem rather than holding them in context. Turn 1: the agent writes a plan file `migration_plan.md` listing all 50 files with their conversion difficulty. Turn 2: it picks file 1, reads the plan slice for file 1, performs the conversion, writes `file1_result.md` with what it changed and what it deferred. Turn 3: it reads `file1_result.md` (because file 2 depends on file 1's API changes), performs file 2's conversion, writes `file2_result.md`. By turn 50 the context window holds only the current file's content, the plan slice, and the prior file's result note — not the full history of all 50 conversions. The plan and the per-file notes are Write artifacts.

**Production tools.** LangChain Deep Agents' virtual filesystem is the canonical 2026 Write implementation, providing the agent with `read_file`, `write_file`, and `ls` tools against a sandboxed FS that maps to the agent's working memory. Claude Code's scratchpad pattern serves the same role for Anthropic's coding agent. Letta's self-editing memory (formerly MemGPT) implements Write against a memory block the agent edits in-place. Sourcegraph's scratchpad pattern is the same idea applied to code intelligence. The Anthropic Cookbook documents the Write pattern under "context engineering: memory, compaction, and tool clearing."

**Characteristic failure mode.** Write-then-forget: the agent writes its intermediate findings to the store but never re-reads them, either because the prompt did not instruct it to consult the store before acting or because the store grew large enough that retrieval was expensive. The result is an information graveyard — tokens spent writing, no tokens saved reading — and an agent that behaves as if it has no memory despite the store being full. The mitigation is to make the read-before-act step mandatory in the harness (the harness engineer's job, not the context engineer's) and to keep the store small enough that retrieval is cheap.

### 5.2 Select — retrieve only the chunks relevant to this step

**Definition.** Select is the operation of choosing, from a much larger corpus of potentially-relevant information, only the slice that is relevant to the current step, and loading only that slice into the context window. The selection is done by the system (not the model) — typically via embedding similarity, lexical search, or a learned retriever — and the model sees only the chosen slice.

**Problem it solves.** Select solves the dual problem of context starvation (the model lacks the document it needs) and pollution (the model has too many irrelevant documents). Without Select, the engineer faces a binary: load the entire corpus (pollution, cost, attention dilution) or load nothing (starvation, hallucination). Select dissolves the binary by loading only the relevant slice, and it scales: a corpus of a million documents becomes tractable because only the top-k chunks per query enter the window.

**When to use it.** Select is the right answer whenever the relevant knowledge is too large to fit in the window, whenever the relevant slice varies per query (so a fixed prefix won't work), and whenever the corpus changes faster than a snapshot can be cached. It is the dominant operation for knowledge-heavy tasks: question-answering over a document corpus, code-completion over a large codebase, customer-support over a knowledge base, and any task where the model must ground a claim in a specific source.

**When NOT to use it.** Select is unnecessary when the model already knows the answer (the corpus is in the model's training data and the task does not require citation). It is harmful when the corpus is small enough to fit in the window directly — Select introduces a retrieval step that can fail, and a 50-document corpus that fits in 30k tokens is better loaded in full than retrieved against. It is also harmful when retrieval quality is poor: a bad chunker, a stale index, or an irrelevant top-k will ground the model in the wrong document, and the model will be confidently wrong rather than honestly uncertain.

**Worked example.** A customer-support agent for a SaaS company with 10,000 help articles should Select the top-5 most relevant articles per user query rather than loading all 10,000 into context. The pipeline: embed the user's question, retrieve the top-5 articles by cosine similarity from a vector store (Pinecone, Weaviate, pgvector), optionally re-rank with a cross-encoder, and load the 5 article bodies into the context window alongside the system prompt and the conversation history. The model answers grounded in those 5 articles. If none of the 5 contain the answer, the model says so — this is the "If the answer isn't in the provided context, say so — do not infer" failure-mode defense from the prompt-engineer skill's diagnostic table.

**Production tools.** RAG over vector DBs is the dominant Select implementation: Pinecone, Weaviate, Chroma, pgvector (for Postgres-native deployments), Qdrant, Milvus, and Redis Vector Search all serve the same role. Hybrid search (vector + BM25, or dense + sparse) is the 2026 production default — pure vector search misses exact-match queries (product names, error codes, IDs), and pure BM25 misses semantic matches, so combining them with reciprocal-rank fusion is the standard. For tool schemas specifically, MCP Tool Search (Anthropic, January 14, 2026) is the canonical Select operation: instead of loading all MCP tool definitions into context upfront, the harness dynamically loads only the tool definition the model needs for the current step. Anthropic's A/B tests showed Tool Search reduced MCP-related context consumption by up to 85% and total agent tokens by 46.9% on runs that called at least one MCP tool. GraphRAG and HippoRAG 2 (Section 9) are Select variants for multi-hop retrieval.

**Characteristic failure mode.** Bad chunks, stale indices, irrelevant retrieval. A chunker that splits a paragraph mid-sentence produces chunks that lose their referents; an index that hasn't been refreshed since the corpus changed returns stale chunks; a retriever whose embeddings don't match the query distribution returns top-k chunks that are syntactically similar but semantically off. In all three cases, the model receives a context that *looks* grounded — five documents, properly cited — and produces a confidently-wrong answer. The mitigation is retrieval evaluation (a held-out set of queries with known-relevant documents, monitored for recall@k), index freshness SLAs, and chunking strategies that respect document structure (semantic chunking, sentence-aware splitting, markdown-header-aware splitting).

### 5.3 Compress — summarize history so decisions survive but raw tool dumps die

**Definition.** Compress is the operation of replacing a long context (typically conversation history or accumulated tool outputs) with a shorter representation that preserves the decisions, errors, and facts the next step needs while discarding the verbose raw material that produced them. The compression is typically done by an LLM call (often a smaller, cheaper model) that reads the long history and emits a structured summary.

**Problem it solves.** Compress solves the OOM problem on multi-turn agents. A 30-turn research agent's raw history — every tool call, every tool output, every intermediate reasoning trace — easily exceeds 100k tokens, and most of it is no longer relevant by turn 30. Without Compress, the agent either hits the context cap (and fails) or suffers attention dilution (and produces worse output). With Compress, the agent carries forward a 400-token summary of "what we've decided, what we've ruled out, what we've tried, what's still open" and discards the 99,600 tokens of raw history that produced those decisions.

**When to use it.** Compress is the right answer on any multi-turn agent whose history will eventually exceed a meaningful fraction of the context window. The trigger is typically a token-count threshold: Claude Code auto-compacts at a configured utilization (commonly ~80–90% of window), and the engineer's job is to make sure compaction fires *before* attention dilution degrades output quality, not after. The MindStudio production guide recommends running `/compact` proactively at 60% context utilization rather than reactively when the agent starts losing the thread.

**When NOT to use it.** Compress is unnecessary on single-turn tasks (no history to compress). It is harmful when the compression target is too aggressive — a 30-turn history compressed to 200 tokens will lose decisions the next turn needs, and the agent will re-litigate questions it already resolved. It is also harmful when the compression loses structural information: a summary that flattens "we tried X, it failed because Y, so we tried Z" into "we tried X and Z" drops the causal chain that would prevent the agent from retrying X on a future turn.

**Worked example.** Claude Code's `/compact` command is the canonical 2026 Compress implementation. When invoked (or when auto-compact triggers), Claude Code sends a separate summarization request that reads the full conversation history and emits a summary inside `<summary></summary>` tags (per the Claude Platform compaction docs), structured to preserve the information needed to continue the task in a future context window. The summary replaces the raw history in the next turn's context; the raw history is gone. As of Claude Code v2.1.198, the summarization request inherits the user's configured model and permission settings, so the compressed context preserves the same behavioral profile as the original. A typical summary captures: the original goal, the steps taken, the decisions made (with rationale), the errors encountered, the open questions, and the next planned action.

**Production tools.** Claude Code `/compact` (manual and auto), LangGraph message summarization (built-in reducer that summarizes old messages when history exceeds a threshold), the Anthropic Cookbook's compaction patterns, Letta's sleep-time compute (compress offline between iterations rather than inline), and the open-source `langchain-ai/context_engineering` repo's Compress examples. The OpenAI Agents SDK and Claude Agent SDK both ship compaction primitives.

**Characteristic failure mode.** Information loss. Every compression is a lossy transformation, and the question is whether the loss is in the parts the next turn needs. The two sub-modes are (a) over-compression — the summary is too short and drops a critical fact, causing the agent to re-derive or re-litigate — and (b) structural flattening — the summary preserves the *what* but loses the *why* (the causal chain, the rejected alternatives, the reason a tool call failed). The mitigation is to compress in structured form (decisions, errors, open-questions, next-action) rather than as a free-text narrative, and to compress less aggressively than the token budget strictly requires. A 400-token structured summary is almost always better than a 200-token prose paragraph.

### 5.4 Isolate — run sub-tasks in sub-agents with their own context windows

**Definition.** Isolate is the operation of running a sub-task in a separate sub-agent with its own context window, so that the sub-task's context (its retrieval, its tool outputs, its reasoning trace) never enters the parent agent's context. Only the sub-task's final result — typically a compressed return value — returns to the parent.

**Problem it solves.** Isolate solves context pollution across branches and the inflation of the parent window by parallel exploration. A research agent that must explore 10 hypotheses in parallel cannot do so in a single context window — each hypothesis generates its own retrieval, its own tool calls, its own intermediate findings, and loading all 10 into one window produces an unreadable mess. Isolate gives each hypothesis its own sub-agent, its own window, and its own reasoning trace; the parent receives 10 compressed findings back and synthesizes them.

**When to use it.** Isolate is the right answer for parallel exploration (multiple independent sub-questions), for deep-search tasks where each branch may run many turns, for any sub-task whose context would pollute the parent (e.g., a sub-task that retrieves a large document the parent doesn't need to see), and for any sub-task whose reasoning trace is not relevant to the parent's next decision. It is also the right answer for capability boundaries — a sub-agent with a different system prompt, different tools, or different permissions can be Isolated from the parent so its capabilities don't leak.

**When NOT to use it.** Isolate is unnecessary for sequential tasks where the parent needs the full reasoning trace (not just the result) to make its next decision. It is harmful when the sub-agent's compressed return loses nuance the parent needs — a 50-token summary of a 5,000-token sub-agent investigation may drop the qualification that would change the parent's synthesis. It is also harmful when the orchestrator cannot verify the sub-agent's work: a sub-agent that returns "I checked and the bug is fixed" without an artifact the parent can inspect is an unverifiable claim, and the parent's synthesis will inherit the unverified-ness.

**Worked example.** A multi-source research agent asked to "synthesize the state of context engineering as of mid-2026 from academic, vendor, and practitioner sources" should Isolate each source-type into its own sub-agent. The orchestrator spawns three sub-agents: (1) an academic-search sub-agent that queries arXiv for "context engineering" surveys and returns a 300-token brief of the three most-cited papers; (2) a vendor-search sub-agent that reads the Anthropic and LangChain blogs and returns a 300-token brief of the canonical four-operation vocabulary; (3) a practitioner-search sub-agent that scrapes Substack and Medium and returns a 300-token brief of the dominant 2026 production patterns. The orchestrator's context window holds the original question, the three 300-token briefs, and a synthesis instruction — not the raw arXiv PDFs, the full blog HTML, or the Substack posts. The Isolate operation is what makes the synthesis tractable.

**Production tools.** Claude Agent Skills are the canonical Isolate implementation: each skill is a self-contained capability with its own SKILL.md, scripts, references, and assets, and only the skill's three-level loading system (metadata always in context, SKILL.md body when triggered, bundled resources on demand) controls what enters the parent. LangChain Deep Agents subagents are the same pattern: a sub-agent is configured with its own tools, its own system prompt, and its own context budget, and returns a compressed result. Opus 4.8's dynamic workflows (the Anthropic Developer Platform's sub-agent fan-out) are the model-native version. The MCP ecosystem's tool-server architecture also implements Isolate at the tool layer: a tool server runs in its own process with its own context, and only the tool's structured return enters the agent's window.

**Characteristic failure mode.** Result-quality loss and verification impossibility. The compressed return from a sub-agent is a lossy representation of the sub-agent's investigation, and the loss may be in the parts the parent needs. Worse, the parent often cannot verify the sub-agent's claim — if the sub-agent returns "the bug is fixed," the parent must either trust it (unsafe) or re-do the verification itself (defeating the Isolate). The mitigation is to require sub-agents to return *artifacts* (a diff, a test result, a URL, a quoted passage) rather than *claims*, so the parent's synthesis is grounded in inspectable evidence. The harness engineer's job is to enforce this contract; the context engineer's job is to design the return format that makes it possible.

### 5.5 How the four operations compose

The four operations are not alternatives chosen per agent; they compose within a single agent run. A production research agent on turn 30 of a long session will simultaneously **Write** intermediate findings to a virtual filesystem (so they survive compaction and can be re-read), **Select** fresh chunks from a vector store for the current sub-question (so the model grounds the next step in the right document), **Compress** the prior 29 turns into a 400-token brief (so the history doesn't overflow), and **Isolate** each parallel exploration thread in its own sub-agent (so pollution in one branch doesn't bleed into another). The four operations form a pipeline: Write keeps the durable state outside the window, Select pulls only the relevant slice inside, Compress keeps the history inside short enough to be useful, and Isolate keeps each branch's window scoped to its own task. The context engineer's deliverable is a specification of how the four operations compose for this agent, on this task, at this point in its lifecycle — not a choice of one operation over the others.

---

## 6. The Five Context Layers

The four operations describe *how* information moves relative to the context window; the five context layers describe *what* the information is. Every token in an agent's context window belongs to one of five layers, and the context engineer's job is to decide what each layer contains, when each layer is included, and how each layer mutates across the agent run. The layers, in canonical order from the top of the window to the bottom, are: system instructions, working memory, retrieved knowledge, conversation history, and tool outputs. This five-layer model is the one used in the prompt-engineer skill's Step 2.5 and is consistent with the Anthropic essay's framing.

### 6.1 System instructions

System instructions are the top-of-window layer that defines the agent's role, rules, and output format. They include the role assignment ("You are a senior research analyst"), the behavioral rules ("Cite every claim; if the answer is not in the provided context, say so"), the output format specification ("Respond in a JSON object with keys `answer`, `citations`, `confidence`"), and any fixed policies (security constraints, refusal conditions, tone calibration). System instructions should be stable across the entire agent run — they are the most cacheable layer (Section 7) and the layer that most benefits from prompt-caching breakpoints. **What to include:** role, rules, output format, refusal conditions, any fixed examples. **When to omit:** never. System instructions are the one layer that should be present on every call, even turn 1 of a one-shot task. **How it mutates across a run:** it shouldn't. If the system prompt changes per turn, prompt caching breaks and the engineer is paying full price on every call. The exception is dynamic few-shot examples selected per turn, which belong in a separate cacheable-but-updatable layer.

### 6.2 Working memory

Working memory is the layer that holds the current task state: the original goal, the plan, the steps completed so far, the steps remaining, the current sub-task, and any in-flight variables. In a ReAct-style agent this is the reasoning trace; in a plan-and-execute agent it is the plan plus the current step's status; in a Deep Agents-style agent it is the contents of the virtual filesystem's planning document. Working memory is the layer most likely to be **Written** to external storage (Section 5.1) so it survives compaction. **What to include:** goal, plan, step status, current sub-task, key in-flight variables. **When to omit:** for one-shot tasks with no internal state, working memory is empty. **How it mutates across a run:** continuously — every turn updates the step status, every plan revision overwrites the plan, every completed sub-task adds to the "done" list. This is the most volatile layer, which means it is the layer that most needs to be Written (so the durable state survives) and the layer that most needs Compress (so the volatile representation does not consume the window).

### 6.3 Retrieved knowledge

Retrieved knowledge is the layer that holds the documents, chunks, or facts retrieved for the current step — the output of the Select operation. In a RAG pipeline this is the top-k chunks from the vector store; in a code agent this is the relevant files; in a customer-support agent this is the relevant help articles. Retrieved knowledge is the layer with the highest variance in quality: a good retrieval grounds the model in the right document; a bad retrieval grounds it in the wrong one and produces a confidently-wrong answer. **What to include:** only the chunks the current step needs, with citations, with enough context that the chunk is interpretable (a chunk that begins mid-sentence is useless). **When to omit:** when the model already knows the answer from training and the task does not require citation; when the corpus is small enough to load in full; when the current step is a pure-reasoning step that does not need external knowledge. **How it mutates across a run:** retrieved knowledge is recomputed on every step — the prior step's retrieved chunks should be evicted and the current step's chunks loaded, unless the prior step's chunks are still relevant to the current step (in which case they should be retained explicitly, not implicitly). The most common failure mode is stale retrieval: chunks from turn 5 still in the window on turn 15, polluting the model's attention.

### 6.4 Conversation history

Conversation history is the layer that holds the prior turns of the conversation — the user's prior messages, the agent's prior responses, and (in agentic settings) the prior tool calls and their results. This is the layer that the Compress operation primarily targets. **What to include:** the user's prior messages (usually verbatim — the user's intent should not be lossily compressed), the agent's prior decisions (in compressed form), the prior tool calls' results (heavily compressed — the raw 50,000-token tool output should become a 50-token summary of what it returned). **When to omit:** on turn 1 (there is no history); on tasks where the prior turns are irrelevant to the current step (rare in multi-turn agents, common in single-turn API calls). **How it mutates across a run:** history grows monotonically until Compress fires, at which point it shrinks to a summary and begins growing again. The art is in the Compress policy: compress too early and you lose decisions the next turn needs; compress too late and the agent has already suffered attention dilution. The 60%-utilization proactive-compression heuristic from the MindStudio production guide is a reasonable default; the exact threshold should be tuned per agent and per model.

### 6.5 Tool outputs

Tool outputs are the layer that holds the raw results of the most recent tool calls — API responses, search results, code-execution stdout, database query results. This is the layer that consumes the most tokens per call and the layer that most needs to be evicted quickly. A single web-search tool call can return 10,000 tokens of HTML; a single database query can return 50,000 tokens of rows; a single code-execution call can return 100,000 tokens of logs. **What to include:** the most recent tool output, in full or in compressed form, plus any prior tool outputs still relevant to the current step. **When to omit:** when the tool output is no longer relevant (the prior step's tool output should be evicted on the next step unless explicitly retained); when the tool output is verbose raw material that should be Compressed before re-injection. **How it mutates across a run:** tool outputs are the most volatile layer — they enter on the step that called the tool, persist for one or two steps, and should be evicted or compressed by step three. The most common failure mode is tool-output accumulation: every prior tool call's raw result still in the window on turn 20, consuming 80% of the budget. The mitigation is aggressive eviction (drop tool outputs after one step) combined with selective Compress (retain a one-sentence summary of each prior tool call's result).

### 6.6 The layer-stack mental model

The five layers stack, in order: system instructions (stable, top), working memory (volatile, written to external store), retrieved knowledge (recomputed per step), conversation history (compressed), tool outputs (evicted quickly). The context engineer's deliverable is a per-layer policy: what goes in, when it comes out, how it is compressed, and how the layer composes with the others. The four operations apply *across* layers — Write moves working-memory and tool-output content out of the window; Select populates the retrieved-knowledge layer; Compress shortens the conversation-history and tool-output layers; Isolate gives each branch its own copy of all five layers. The five-layer model and the four-operation vocabulary are complementary, not competing — one describes what the context contains, the other describes how it is managed.

---

## 7. Context Window Economics

Context engineering is not just a quality discipline; it is a cost discipline. Every token in the context window costs money on every call, and the cost structure of modern LLM APIs makes context engineering a direct lever on the unit economics of any agent system. This section walks the economics: token costs, cache economics, the 1M-token era, and why a larger window does not eliminate the discipline.

### 7.1 Token costs

LLM APIs price input and output tokens separately, and both are priced per million tokens. Input tokens (what is in the context window when the model is invoked) are typically 3–10× cheaper than output tokens (what the model generates), but in agentic settings the input token count is usually 10–100× larger than the output token count, so input cost dominates. A research agent that loads a 100k-token context and generates a 1k-token answer pays for 101k tokens of input and 1k tokens of output; even with input priced at one-fifth of output, the input cost is 20× the output cost. The implication is that the context engineer's primary cost lever is reducing input tokens — every chunk that does not need to be in the window is money saved on every call, and a 50% reduction in average context size is a 50% reduction in input cost.

### 7.2 Cache economics

Prompt caching is the single largest cost lever in 2026 LLM economics. Both Anthropic (Claude prompt caching, generally available since August 2024) and OpenAI (GPT-5.6 explicit cache breakpoints) support caching of repeated context prefixes, and both price cached reads at roughly 10% of the base input rate — a 90% discount on any token that hits the cache. The cache works by matching a prefix: if the first N tokens of the current call's context are identical to the first N tokens of a recent call, those N tokens are served from cache at the 10% rate. The implication for context engineering is structural: the context should be ordered so that the stable parts (system prompt, fixed instructions, large reference documents) are at the top, and the volatile parts (current user message, current retrieved chunks, current tool output) are at the bottom. An engineer who puts a fresh retrieved chunk in the middle of the system prompt pays full price for every token after the chunk on every call, because the cache breaks at the chunk.

Claude's prompt caching uses `cache_control` breakpoints that the engineer places explicitly in the message content; GPT-5.6 introduced explicit cache breakpoints with a 30-minute minimum cache life [UNVERIFIED — check primary source; the 30-minute minimum cache life for GPT-5.6 was not independently confirmed via web search and may reflect OpenAI's stated cache TTL policy], meaning cached prefixes survive for at least 30 minutes between calls. The 30-minute minimum is significant for agentic loops: a loop that fires every 5 minutes can rely on its system prompt and reference documents being cached across iterations, while a loop that fires every 45 minutes cannot. The PE Collective production guide emphasizes that prompt caching "saves 90% on repeated system prompts" and that the engineer's job is to "add a cache_control breakpoint to your message content" at every stable-prefix boundary. The HN discussion of automatic cache-breakpoint injection plugins notes that visibility into cache hit rates is the missing piece in most production setups — the engineer who cannot see their cache hit rate is paying full price without knowing it.

### 7.3 The 1M-token era

As of mid-2026, every frontier model ships a 1M-token-or-greater context window: Claude Opus 4.8 at 1M, GPT-5.6 at approximately 1.5M, Gemini 3.5 at 1M+, and GLM-5.2 at 1M. The naive reading is that the larger window eliminates the need for context engineering — if everything fits, why curate? The naive reading is wrong, for three reasons.

**First, attention dilution.** The "lost in the middle" effect, documented by Liu et al. (arXiv:2307.03172) and corroborated across model families in 2025–26, shows that LLM performance on retrieval tasks degrades when the relevant information is in the middle of a long context rather than at the beginning or end. The diffray.ai aggregation of 2025–26 research reports 13.9% to 85% accuracy drops as context grows, depending on task and model. A 1M-token window stuffed with 800k tokens of context is not a model with 800k tokens of useful information; it is a model with 800k tokens of attention-diluting noise, of which perhaps 5k are actually being used. The morphllm.com "Context Rot" guide frames the underlying mechanism as attention dilution: transformers allocate finite attention budget across all tokens, and as the token count grows, the attention allocated to any individual token shrinks. Bigger windows make the dilution worse, not better, because they invite the engineer to dump more.

**Second, cost.** A 1M-token input costs 1M tokens of input pricing on every call, cached or not. Even with a 90% cache discount, a 1M-token input that misses cache costs the full 1M tokens; a 1M-token input that hits cache costs 100k tokens of effective pricing. A 50k-token curated input that hits cache costs 5k tokens of effective pricing. The 20× cost difference between the curated and the dumped context is the context engineer's budget, and the engineer who treats the 1M window as a license to dump is paying 20× more per call than the engineer who curates.

**Third, latency.** Time-to-first-token scales roughly linearly with input token count, even with caching, because the model must process the full input before generating. A 1M-token input has measurably higher latency than a 50k-token input, and in interactive or high-frequency-loop settings the latency difference is user-visible. The context engineer who curates is also a latency engineer.

### 7.4 Why 1M context does NOT replace context engineering

The synthesis of the three reasons above is that the 1M-token era makes context engineering *more* important, not less. A larger window raises the ceiling on what can be loaded, but it also raises the floor on the cost of loading carelessly. The four operations apply unchanged: Write still matters because a 1M window fills faster than you'd think on a long agent run; Select still matters because attention dilution punishes the engineer who dumps 800k tokens to find the 5k that matter; Compress still matters because a 30-turn agent's raw history exceeds 1M tokens faster than a 5-turn agent's history exceeded the old 200k window; Isolate still matters because parallel branches in one window still produce an unreadable mess at 1M tokens. The 1M window is a license to handle bigger individual artifacts (a full codebase, a long document), not a license to skip curation. The engineers who skip curation in 2026 are the engineers whose agents fail with 800k tokens of context and 5k tokens of relevant signal.

---

## 8. Context Pollution and Its Mitigations

Context pollution is the dominant failure mode of long-running agents, and it is the failure mode the four operations exist to prevent. This section walks the symptoms, the causes, and the four-operation mitigations.

### 8.1 What context pollution is

Context pollution is the accumulation of irrelevant, stale, or contradictory information in the context window across an agent run, to the point where the model's attention is so diluted that it loses track of the original goal, repeats itself, hallucinates, or produces output that contradicts a decision it made three turns ago. Anthropic's September 2025 essay names the problem explicitly: agents working across extended time horizons need techniques that "prevent context pollution," and the four operations are the techniques. Pollution is not the same as overflow (hitting the context cap) — pollution can occur at 30% utilization if the 30% is the wrong 30%. Pollution is a quality problem, not a capacity problem, and it cannot be solved by a larger window.

### 8.2 The symptoms

The symptoms of context pollution are recognizable to any engineer who has run a long agent session. **Goal drift:** the agent's understanding of the task subtly shifts across turns, optimizing for something other than the original intent — a research agent asked for "the state of context engineering in 2026" drifts into "the history of prompt engineering" by turn 15 because the prior turns' retrieved chunks were about prompt engineering and the model's attention was captured by them. **Repetition:** the agent re-asks a question it already answered, re-tries a tool call that already failed, or re-derives a conclusion it already reached — because the prior turn's result was either evicted or buried under 50k tokens of new material. **Hallucination:** the agent fabricates a fact that is consistent with the polluted context but not with reality — a model that has been reading stale documentation for 20 turns will hallucinate API signatures that match the stale docs. **Contradiction:** the agent's turn-20 output contradicts its turn-5 output, because the turn-5 decision was compressed out of the history and the agent re-derived a different decision from the compressed summary. All four symptoms are downstream of the same root cause: the model's finite attention is spread across too many tokens, and the signal-to-noise ratio drops below the threshold at which the model can reliably act.

### 8.3 The causes

The causes of context pollution are the four anti-patterns the four operations exist to prevent. **Failure to Write:** intermediate state stays in the window instead of being offloaded to external storage, so the window fills with accumulated partial findings. **Failure to Select:** the engineer loads the entire corpus or retrieves too broadly, so the window fills with irrelevant chunks. **Failure to Compress:** history grows monotonically, so the window fills with prior turns' raw material that is no longer relevant. **Failure to Isolate:** parallel branches run in the same window, so each branch's pollution bleeds into the others. Each cause maps directly to an operation: Write prevents accumulation, Select prevents irrelevance, Compress prevents bloat, Isolate prevents cross-branch contamination. A long-running agent that uses none of the four operations will pollute; an agent that uses all four will not, provided each operation is applied at the right threshold.

### 8.4 The four-operation mitigations

**Compress aggressively.** The first mitigation is to compress history before it pollutes. The MindStudio production guide's 60%-utilization proactive-compression heuristic is a reasonable default: when the context window is 60% full, run `/compact` (or the equivalent) and reset to a 400-token structured summary. Compressing at 60% rather than 90% gives the model headroom for the next few turns and prevents the attention dilution that begins to bite above 70% utilization. The compression target should be structured (decisions, errors, open-questions, next-action) rather than free-text, and it should preserve the causal chain (why a decision was made, why a tool call failed) rather than just the outcomes.

**Isolate sub-tasks.** The second mitigation is to run sub-tasks in sub-agents with their own context windows, so that each branch's pollution stays in its branch. A research agent exploring 10 hypotheses should spawn 10 sub-agents, each with its own window; the parent receives 10 compressed returns and synthesizes them. The parent's window never sees the 10 sub-agents' raw retrieval, raw tool outputs, or raw reasoning traces — only the compressed returns. The Isolate operation is the most powerful pollution mitigation because it is the only one that gives the model a *clean* window for each sub-task; the other three operations manage a single window, while Isolate sidesteps the single-window constraint entirely.

**Write to external memory.** The third mitigation is to offload durable state to external storage, so that the window holds only the current step's working set. A code-migration agent should write its per-file migration notes to a virtual filesystem; a research agent should write its findings to a structured document; a customer-support agent should write the user's case history to a database. The window then holds the current step's instruction, the current step's retrieved slice, and a pointer to the external store — not the full accumulated history. The Write operation is what makes long agent runs tractable: the durable state lives outside the window, and the window holds only the volatile current step.

**Select only relevant chunks.** The fourth mitigation is to retrieve only the chunks the current step needs, evicting the prior step's chunks unless they are still relevant. A research agent that retrieved 5 chunks on turn 5 should not still have those 5 chunks in the window on turn 15 unless turn 15's sub-question is the same as turn 5's. The Select operation, applied per-step with eviction, keeps the retrieved-knowledge layer small and current. The most common Select failure is stale retrieval — chunks from prior turns persisting in the window — and the mitigation is explicit eviction: each step's retrieval replaces the prior step's retrieval, not appends to it.

### 8.5 The monitoring layer

The four-operation mitigations are necessary but not sufficient without monitoring. A production agent should emit, on every call, the per-layer token counts (system, working memory, retrieved knowledge, history, tool outputs), the cache hit rate, the compression ratio, and the retrieval recall@k. These metrics are the context engineer's observability surface, and they are what the LangSmith and Langfuse context-observability dashboards (Section 12) are designed to surface. An agent whose history layer grows monotonically across turns is an agent that is not Compressing; an agent whose retrieved-knowledge layer is the same on every turn is an agent that is not Selecting per-step; an agent whose tool-output layer is 70% of the window on turn 20 is an agent that is not evicting. The metrics make the failure modes visible, and visibility is the precondition for mitigation.

---

## 9. RAG and Retrieval

Retrieval-Augmented Generation (RAG) is the dominant Select implementation in 2026 production systems, and it is the operation most practitioners think of first when they hear "context engineering." This section walks RAG's place in the four-operation vocabulary, the vector-DB landscape, chunking strategies, hybrid search, GraphRAG, and the HippoRAG 2 alternative — and the failure modes that make RAG a Select operation that can go badly wrong.

### 9.1 RAG as the canonical Select

RAG is a Select operation: from a large corpus, retrieve only the chunks relevant to the current query, and load only those chunks into the context window. The retrieval is done by the system (not the model), typically via embedding similarity against a vector index, and the model sees only the retrieved chunks. The four-operation framing is clarifying here because it separates RAG (one Select implementation) from the broader discipline: RAG does not address Write, Compress, or Isolate, and a context-engineering plan that uses RAG alone is incomplete on any multi-turn agent. The practitioner who equates "context engineering" with "RAG" omits three of the four operations and is surprised when the agent still pollutes its window with un-compressed history.

### 9.2 The vector-DB landscape

The 2026 vector-DB market is mature and commoditized. **Pinecone** is the managed-service leader, optimized for high-throughput retrieval with serverless scaling. **Weaviate** is the open-source leader with strong hybrid-search support and a graph-aware object model. **Chroma** is the embedded leader, favored for prototyping and small-to-medium deployments. **pgvector** is the Postgres extension that lets engineers add vector search to an existing relational database without a new system, and it is the production default for any team already on Postgres. **Qdrant** and **Milvus** are the high-scale open-source options, and **Redis Vector Search** is the low-latency option for cache-adjacent retrieval. The choice among them in 2026 is less about retrieval quality (all are within a few percentage points of each other on standard benchmarks) and more about operational fit: managed vs self-hosted, integrated with the existing database vs standalone, embedded vs server. The context engineer's job is not to pick the best vector DB but to pick the one that fits the harness's storage layer and the team's operational capacity.

### 9.3 Chunking strategies

Chunking is the highest-leverage and most-underappreciated lever in RAG quality. A chunker that splits a paragraph mid-sentence produces chunks that lose their referents; a chunker that splits a document by fixed token count produces chunks that ignore document structure; a chunker that splits by markdown headers produces chunks that respect structure but may be too large or too small. The 2026 production defaults are **semantic chunking** (split at sentence and paragraph boundaries, with overlap to preserve context), **markdown-header-aware chunking** (split at header boundaries, with header text prepended to each chunk so the chunk is interpretable in isolation), and **sentence-window chunking** (retrieve a sentence, return the sentence plus its surrounding window for context). The chunking strategy should be chosen based on the document structure: markdown-header-aware for technical docs, semantic for prose, sentence-window for legal or scientific text where individual claims must be grounded. The most common chunking failure is fixed-token-count chunking with no overlap, which produces chunks that are syntactically complete but semantically incomplete — a chunk that ends just before the referent of a pronoun at the start of the next chunk is useless.

### 9.4 Hybrid search

Hybrid search — combining vector (dense) and BM25 (sparse) retrieval — is the 2026 production default. Pure vector search misses exact-match queries (product names, error codes, IDs, version numbers) because embeddings smooth over the exact tokens; pure BM25 misses semantic matches (paraphrases, synonyms, conceptual similarity) because it operates on token overlap. Combining them with reciprocal-rank fusion (RRF) — a simple formula that merges the two ranked lists by the reciprocal of their ranks — captures both: the exact-match signal from BM25 and the semantic-match signal from vectors. Most production RAG systems in 2026 run both retrievers in parallel and fuse the results; Weaviate, Pinecone (with its sparse-vector support), and OpenSearch all ship hybrid search natively. The context engineer who runs pure vector search in 2026 is leaving accuracy on the table for any query that contains an identifier.

### 9.5 GraphRAG and multi-hop retrieval

GraphRAG, popularized by Microsoft Research in 2024, extends RAG to multi-hop retrieval by building a knowledge graph from the corpus and traversing it at query time. The advantage is that GraphRAG can answer questions that require combining facts across documents (e.g., "which of our customers are affected by the supplier consolidation announced last quarter?"), where pure vector RAG retrieves chunks that each contain one fact but cannot combine them. The disadvantage is cost: GraphRAG's graph construction is expensive (an LLM call per entity-relation extraction), its retrieval is slower (graph traversal plus LLM-based community summarization), and its indexing is brittle (re-extraction required when the corpus changes). For corpora where multi-hop reasoning is rare, GraphRAG is overkill; for corpora where it is common (legal cases, supply chains, biomedical literature), it is essential.

### 9.6 HippoRAG 2 — the cheaper alternative

HippoRAG 2 (from the OSU NLP Group, building on the original HippoRAG, arXiv:2405.14831) is the leading 2026 alternative to GraphRAG for multi-hop retrieval. HippoRAG 2 models retrieval on the hippocampal memory indexing theory: it builds a personalized PageRank-augmented association graph from the corpus and uses it to retrieve multi-hop associations at query time. The performance claims, corroborated by independent benchmarks, are significant: HippoRAG 2 is **10–30× cheaper than iterative retrieval methods like IRCoT** and **6–13× faster than multi-step approaches**, while improving associativity (multi-hop retrieval) and sense-making (integrating large and complex contexts) over both the original HippoRAG and GraphRAG. For production systems where GraphRAG's cost is prohibitive but pure vector RAG's single-hop limitation is a quality ceiling, HippoRAG 2 is the 2026 default. The github repository (`osu-nlp-group/hipporag`) ships the reference implementation.

### 9.7 When RAG fails

RAG fails in four characteristic ways. **Bad chunks:** a chunker that produces semantically incomplete chunks grounds the model in fragments that lose their referents, and the model hallucinates the missing context. **Stale indices:** an index that hasn't been refreshed since the corpus changed returns chunks that contradict the current state of the world — a RAG system over a codebase that returns pre-refactor function signatures will cause the model to call functions that no longer exist. **Irrelevant retrieval:** a retriever whose embeddings don't match the query distribution returns top-k chunks that are syntactically similar but semantically off, and the model produces a confidently-wrong answer grounded in the wrong document. **Distribution shift:** a retriever trained on one corpus (e.g., English Wikipedia) deployed against another (e.g., internal medical records) may have embedding mismatches that produce systematically poor retrieval. The mitigations are retrieval evaluation (a held-out set of queries with known-relevant documents, monitored for recall@k), index freshness SLAs (re-index on a schedule that matches the corpus's change rate), chunking strategies that respect document structure, and embedding-model selection that matches the corpus's domain. A RAG system without retrieval evaluation is a RAG system whose quality is unknown, and an unknown-quality Select operation is more dangerous than no Select at all because it produces confident-wrong answers instead of honest-uncertain ones.

---

## 10. Memory Systems

The 2026 memory ecosystem is the production infrastructure for the Write and Select operations at scale. Where a single-agent system can use a virtual filesystem for Write and a vector store for Select, a multi-agent or long-running system needs persistent, structured, queryable memory that survives across sessions, agents, and loops. This section walks the 2026 memory landscape, the taxonomy that organizes it, and when to use each system.

### 10.1 The memory taxonomy

Memory systems are organized along three axes. **In-context vs external store:** in-context memory lives in the context window (volatile, limited, fast); external store memory lives in a database or filesystem (durable, large, requires retrieval). The Write operation moves information from in-context to external store; the Select operation moves it back. **Episodic vs semantic:** episodic memory stores specific events ("the user asked about refunds on Tuesday and was unhappy"); semantic memory stores generalized facts ("the user prefers email communication"). Episodic memory is what conversation history provides; semantic memory is what a knowledge graph provides. **Temporal vs atemporal:** temporal memory stores facts with time stamps and can answer "what was true on date X"; atemporal memory stores facts without time and can only answer "what is true." Temporal memory is essential for any agent that operates on a changing world (customer state, codebase state, market state); atemporal memory suffices for static knowledge bases. The 2026 memory systems differ in which combinations of these axes they support, and the context engineer's job is to match the system to the agent's memory requirements.

### 10.2 Letta — self-editing memory plus sleep-time compute

Letta (formerly MemGPT, from UC Berkeley) is the leading 2026 implementation of self-editing agent memory. Letta agents edit their own memory blocks in-place — the model decides what to write, what to update, and what to evict, rather than relying on an external script. The self-editing model means the agent's memory evolves with the conversation in a way that fixed-schema memory systems cannot match: the agent can promote an episodic memory to a semantic one ("the user mentioned three times they prefer email; I'll store 'user prefers email' as a fact"), demote a stale fact, and reorganize its memory as the task evolves. Letta also pioneered **sleep-time compute** — processing memory offline, between iterations, rather than in-context — which reduces per-iteration token cost by moving memory consolidation out of the hot path. The Letta forum's comparison of Letta vs Mem0 vs Zep vs Cognee characterizes Letta as the most agent-native of the four: the memory is part of the agent, not a separate store the agent queries. The use case is long-running personal agents, customer-support agents with persistent user state, and any agent whose memory must evolve with the task.

### 10.3 Mem0 — vectors plus knowledge graph

Mem0 is the 2026 leader for extract-and-retrieve memory. Mem0 extracts facts from conversations, stores them as vectors (for semantic retrieval) and as knowledge-graph edges (for multi-hop queries), and retrieves them at query time. The dual representation — vectors for similarity, graph for relations — gives Mem0 broader coverage than pure-vector stores (it can answer "who works with whom" via the graph) and broader coverage than pure-graph stores (it can answer "what's similar to X" via the vectors). Mem0's architecture is more passive than Letta's: where Letta's agent edits its own memory, Mem0's extraction is driven by the system (a separate extraction LLM call), and the agent queries the resulting store. The Mem0 "State of AI Agent Memory 2026" report frames the system as the production default for high-volume, multi-user agents where per-user self-editing memory is too expensive. Pricing (Starter at $19/mo, Growth at $79/mo, Pro tier) makes it accessible for teams that don't want to operate their own memory infrastructure.

### 10.4 Zep — temporal knowledge graph

Zep is the 2026 leader for temporal memory. Zep's Graphiti engine builds a temporal knowledge graph — a graph where every edge has a time interval during which it is true — which lets the agent answer "what was true on date X" and "how did this relationship evolve over time." The temporal axis is essential for any agent that operates on a changing world: a customer-support agent that needs to know "was the user a paying customer when they reported this bug?" or a code agent that needs to know "did this function exist in version 2.3?" Zep's temporal knowledge graph is more expressive than Mem0's atemporal graph and more structured than Letta's self-edited memory blocks, at the cost of higher operational complexity. The use case is any agent where time matters — customer state, financial state, legal state, codebase history.

### 10.5 Cognee, Redis LangCache, and the rest

**Cognee** is a newer entrant focused on knowledge-graph-grounded memory with strong eval tooling. **Redis LangCache** is the 2026 leader for semantic caching — caching responses to semantically similar queries (not just identical queries) so that a repeat or near-repeat query hits the cache instead of the model. Semantic caching is a cost technology, not a memory technology, but it is essential for high-volume agents where the same query recurs: a customer-support agent that gets "how do I reset my password?" fifty times a day can serve forty-nine of those from cache. **HippoRAG 2** (Section 9.6) doubles as a memory system for multi-hop associative retrieval. **MemGraphRAG** is the graph-RAG-as-memory variant. **LangMem** is LangChain's memory layer, integrated with LangGraph's checkpointing. The choice among these is driven by the use case: Letta for self-editing personal agents, Mem0 for high-volume multi-user, Zep for temporal, Redis LangCache for cost, HippoRAG 2 for multi-hop, Cognee for eval-heavy, LangMem for LangGraph-native.

### 10.6 When to use each

The decision tree is straightforward. If the agent is long-running and personal (one user, evolving state), use Letta. If the agent is high-volume and multi-user (many users, extract-and-retrieve), use Mem0. If the agent operates on a changing world where time matters, use Zep. If the agent's query pattern is repetitive and cost-sensitive, add Redis LangCache in front of whatever memory system you chose. If the agent's queries require multi-hop reasoning over the memory, use HippoRAG 2 or GraphRAG. If the team is already on LangGraph and wants native integration, use LangMem. The wrong choice is not catastrophic — most systems can be swapped — but the right choice eliminates a class of failures (Letta eliminates static-memory failures, Zep eliminates atemporal-memory failures, Redis LangCache eliminates cost failures) that are otherwise hard to fix downstream.

---

## 11. MCP and Context

The Model Context Protocol (MCP) is the agent-to-tool standard that shapes context engineering at the tool layer. MCP was donated by Anthropic to the Linux Foundation's Agentic AI Foundation (AAIF) in December 2025, co-founded by Anthropic, OpenAI, and Block, and as of July 2026 the 2026-07-28 Specification Release Candidate introduces a stateless protocol core that removes the `initialize` handshake and session id so any MCP request can hit any server instance. This section walks how MCP shapes context engineering, focusing on MCP Tool Search as the canonical Select operation for tools.

### 11.1 How MCP shapes context engineering

MCP shapes context engineering in three ways. First, it standardizes the tool-schema format: every MCP server exposes its tools through a structured description (name, natural-language description, input schema) that the agent loads into context. The standardization means the context engineer can reason about tool schemas uniformly across vendors, rather than per-vendor. Second, it standardizes the prompt-template format: MCP servers expose reusable, parameterized prompt templates that the agent can invoke, which means the prompt layer can be served from MCP servers rather than hardcoded in the agent. Third, it creates the tool-bloat problem that MCP Tool Search solves: an agent connected to many MCP servers (each exposing many tools) would consume a large fraction of its context window on tool schemas alone, which is the failure mode MCP Tool Search was designed to prevent.

### 11.2 MCP Tool Search as a Select operation

MCP Tool Search, announced January 14, 2026 by Thariq Shihipar at Anthropic, is the canonical Select operation for the tool layer. Instead of loading all MCP tool definitions into context upfront, the harness dynamically loads only the tool definition the model needs for the current step. The trigger is when MCP tools would consume more than 10% of the context window; below that threshold, loading all tools upfront is fine, and above it, Tool Search kicks in. Anthropic's A/B tests on Claude Code showed that Tool Search reduced MCP-related context consumption by up to 85% and reduced total agent tokens by 46.9% on runs that called at least one MCP tool — a substantial win on both quality (less attention dilution) and cost (fewer input tokens). The VentureBeat coverage framed the feature as "lazy loading for AI tools," and the Joe Njenga Medium analysis documented the specific reduction: 51k tokens of MCP schemas down to 8.5k with Tool Search enabled.

The critical implementation detail is that MCP Tool Search does not break prompt caching. Deferred tools are excluded from the initial prompt, so the system prompt and reference documents remain cacheable across calls even as the tool set changes per step. This is non-trivial: a naive implementation that injected the current step's tool schema into the middle of the prompt would break the cache at the injection point and cost full price for every token after it. MCP Tool Search's design preserves the stable-prefix property by keeping the deferred tools in a separate, cacheable-or-not layer that does not interfere with the system-prompt cache. The arXiv paper "Model Context Protocol (MCP) Tool Descriptions Are Smelly" (arXiv:2602.14878) analyzes the broader problem of tool-description quality — many MCP servers expose tool descriptions that are vague, injection-prone, or token-inefficient — and Tool Search is the harness-level mitigation that lets the context engineer ignore most tool schemas most of the time.

### 11.3 MCP prompt templates

MCP servers can expose prompt templates — reusable, parameterized prompts that the agent can invoke by name. A server might expose a `code-review` template that takes a diff and returns a structured review, or a `summarize-doc` template that takes a document and returns a summary. The prompt-template layer means the context engineer can offload prompt construction to the MCP server, treating prompts as served resources rather than hardcoded strings. The advantage is reuse and versioning: a prompt template served by an MCP server can be updated without changing the agent code, and the same template can be used by multiple agents. The disadvantage is that prompt templates are a new attack surface: a malicious MCP server could serve a prompt template that injects instructions, and the agent would execute them. The mitigation is allow-listed prompt templates (only invoke templates from trusted servers) and the same Lethal-Trifecta security check that applies to tool descriptions. The context engineer working with MCP-connected agents should reference tools and templates by name in the runbook and let the harness load schemas dynamically via Tool Search, rather than preloading hundreds of tool and template definitions into the system prompt.

---

## 12. Tools & Frameworks (2026)

The 2026 context-engineering tooling stack is real and shipping. This section walks the production frameworks, the observability backends, the memory systems, the vector DBs, the chunking libraries, and the structured-output tooling that a context engineer composes into a working system. The throughline is that every tool in the stack implements one or more of the four operations, and the context engineer's job is to compose them into a coherent whole rather than to pick a single "best" tool.

### 12.1 LangChain 1.0 and Deep Agents

LangChain 1.0 (with LangGraph 1.0 as its orchestration core) is the leading open-source framework for context-engineered agents in 2026. LangChain 1.0's middleware architecture lets the engineer insert context-management hooks (compression, retrieval, eviction) at any point in the agent loop, and LangGraph's state management provides the durable state that the Write operation persists to. **Deep Agents** is LangChain's higher-level abstraction built for long-running tasks: it ships a planning tool, a virtual filesystem, and subagents as batteries-included defaults. The virtual filesystem is the canonical Write + Isolate implementation — the agent writes its plan, intermediate findings, and draft outputs to the filesystem, and each subagent gets its own filesystem scope. The Deep Agents docs frame the system as "the easiest way to start building agents ... with built-in capabilities for task planning, file systems, and multi-agent orchestration," and the open-source repository (`langchain-ai/deepagents`) is the reference implementation. For any team starting a new context-engineered agent in 2026, Deep Agents is the default starting point; the engineer who builds from scratch will re-derive most of what Deep Agents already provides.

### 12.2 Anthropic Claude Agent SDK and Claude Code

The Anthropic Claude Agent SDK (Python and TypeScript) ships the same tools, agent loop, and context management that power Claude Code, programmable in code rather than via the CLI. The SDK's context-management primitives include compaction (the programmatic equivalent of Claude Code's `/compact`), tool-search integration (for MCP-connected systems), and the Agent Skills loading system. Claude Code itself is the most-deployed context-engineered agent in production as of mid-2026, and its `/compact` command (Section 5.3) is the reference Compress implementation. The Claude Platform's compaction docs document the default compaction behavior: the summarization request inherits the user's configured model and permission settings, and the summary is structured inside `<summary></summary>` tags to preserve the information needed to continue the task in a future context window. The Anthropic Cookbook's "context engineering: memory, compaction, and tool clearing" entry is the canonical reference for comparing context-engineering strategies for long-running agents.

### 12.3 OpenAI Agents SDK

The OpenAI Agents SDK (April 15, 2026 update) provides a native sandbox and model-native harness, separating the harness from the compute. For context engineering, the SDK ships structured-output enforcement (the `strict: true` mode that guarantees JSON Schema conformance), compaction primitives, and tool-search-style dynamic tool loading. The structured-output mode is a context-engineering tool because it eliminates the token overhead of repair prompts ("your JSON was malformed, try again") and the failure mode of unparseable output — a strict-mode structured output is guaranteed to parse, which means the downstream context assembly can rely on it. The SDK is the production default for teams on OpenAI models who want native harness integration without adopting LangChain.

### 12.4 Observability: LangSmith and Langfuse

Context observability is the missing layer in most 2026 production setups, and it is the layer that separates the engineers who can debug context failures from the engineers who cannot. **LangSmith** (LangChain's observability backend, at smith.langchain.com) traces every LLM call with its full input context, output, latency, and cost, and provides per-layer token breakdowns that let the engineer see exactly which layer is consuming the budget. **Langfuse** is the open-source alternative, self-hostable and vendor-neutral, with the same per-call tracing and per-layer token breakdowns. Both integrate with the major agent frameworks (LangChain, OpenAI Agents SDK, Anthropic SDK) via auto-instrumentation. The context engineer without observability is flying blind: a degrading agent whose history layer is growing monotonically, whose retrieval is stale, or whose tool outputs are accumulating is invisible without per-layer token metrics, and the engineer who cannot see the failure cannot fix it. The HN discussion of automatic cache-breakpoint injection plugins makes the same point about cache hit rates: visibility is the precondition for optimization.

### 12.5 Vector DBs and chunking libraries

The vector-DB landscape (Section 9.2) is complemented by chunking libraries that implement the strategies in Section 9.3. **LangChain's text splitters** (RecursiveCharacterTextSplitter, MarkdownHeaderTextSplitter, SentenceTransformersTokenTextSplitter) are the most-used chunking libraries, with the markdown-header-aware splitter being the production default for technical docs. **LlamaIndex's node parsers** provide similar functionality with stronger document-structure awareness. **Unstructured** is the library for parsing heterogeneous document formats (PDF, DOCX, HTML, email) into structured chunks. The context engineer's chunking pipeline typically runs: parse with Unstructured, chunk with a markdown-header-aware or semantic splitter, embed with a domain-appropriate embedding model, index in the chosen vector DB, and retrieve with hybrid (vector + BM25) search at query time. Each stage is a potential failure point, and each stage should be evaluated independently.

### 12.6 Structured output: OpenAI strict mode

OpenAI's `strict: true` mode for structured output (and the equivalent JSON Schema enforcement in Anthropic's tool-use API) is a context-engineering tool because it eliminates a class of context failures. Without strict mode, a model asked for JSON may return prose-wrapped JSON, malformed JSON, or JSON with missing fields, and the downstream context assembly must either repair the output (costing extra tokens and a repair-prompt round-trip) or fail. With strict mode, the output is guaranteed to conform to the provided JSON Schema, which means the downstream context assembly can rely on it without repair. For agents that chain LLM calls (where one call's output becomes the next call's input), strict mode is essential: it transforms a probabilistic output into a deterministic interface contract, and the context engineer can treat the structured output as a typed value rather than a string to be parsed. The prompt-engineer skill's power rules list strict mode as rule 14 ("Use structured output — JSON/XML for anything parsed programmatically; use OpenAI `strict: true` for guaranteed JSON Schema conformance"), and it is a context-engineering rule because it eliminates context pollution from repair prompts.

### 12.7 Memory systems

The memory systems from Section 10 (Letta, Mem0, Zep, Cognee, Redis LangCache, HippoRAG 2, MemGraphRAG, LangMem) are the production infrastructure for the Write and Select operations at scale. The context engineer composes them based on the agent's memory requirements: Letta for self-editing personal agents, Mem0 for high-volume multi-user, Zep for temporal, Redis LangCache for cost, HippoRAG 2 for multi-hop. The integration points are the agent framework (LangChain, OpenAI SDK, Anthropic SDK) and the observability backend (LangSmith, Langfuse), and the memory system should emit the same per-call tracing as the LLM calls so the engineer can see retrieval latency, recall, and hit rate alongside the LLM metrics. A memory system without observability is a black box, and a black-box memory system is a context-engineering failure waiting to happen.

### 12.8 The composed stack

A 2026 production context-engineering stack typically looks like: LangChain 1.0 / LangGraph 1.0 as the framework, Deep Agents as the higher-level abstraction (virtual filesystem + planning + subagents), Pinecone or pgvector as the vector DB, Unstructured + LangChain splitters as the chunking pipeline, hybrid (vector + BM25) retrieval, Letta or Mem0 as the memory system, Redis LangCache as the cost layer, LangSmith or Langfuse as the observability backend, Anthropic Claude or OpenAI GPT as the model, MCP for tool integration with Tool Search enabled, and OpenAI strict mode or Anthropic tool-use for structured output. The stack is composed, not monolithic, and the context engineer's job is to wire the pieces together with the four operations as the design vocabulary.

---

## 13. Worked Example — A Multi-Turn Research Agent

This section walks the context window at each step of a multi-turn research agent, showing how the four operations and the five layers compose across a real run. The agent is a research agent tasked with synthesizing the state of context engineering as of mid-2026 from academic, vendor, and practitioner sources. The walk shows turn 1 (system + question + initial retrieval), turn 2 (prior turn compressed + new retrieval + tool output), and turn N (heavily compressed history + only the relevant prior findings + current instruction), with a Python pseudocode snippet showing the context assembly.

### 13.1 Turn 1 — system + question + initial retrieval

The agent's first turn sets the baseline. The context window contains: the system prompt (a stable ~2,000-token instruction defining the agent's role, output format, and citation rules, with a `cache_control` breakpoint at the end so it caches across turns); the user's question ("synthesize the state of context engineering as of mid-2026 from academic, vendor, and practitioner sources"); the working-memory layer (empty on turn 1, or a placeholder for the plan to be written); the retrieved-knowledge layer (empty on turn 1, because retrieval has not yet run); the conversation-history layer (empty on turn 1); and the tool-output layer (empty on turn 1). The agent's first action is to plan: it writes a plan to the virtual filesystem (`research_plan.md`) using the Write operation, identifying the three source types and the sub-questions for each. The plan file persists outside the context window, so the next turn's context will not need to carry the plan in-window — it will reference the file.

### 13.2 Turn 2 — prior turn compressed + new retrieval + tool output

On turn 2, the agent's first action is to retrieve academic sources for the first sub-question. The context window now contains: the cached system prompt (served from cache at 10% of base input cost); the user's question (still in context, ~30 tokens); a compressed representation of turn 1 (~150 tokens, capturing "agent planned three sub-questions: academic, vendor, practitioner; plan written to research_plan.md"); the working-memory layer (a pointer to `research_plan.md` and the current sub-question being worked); the retrieved-knowledge layer (the top-5 arXiv chunks for "context engineering survey," ~3,000 tokens, freshly retrieved and evicting nothing because turn 1 had no retrieval); the conversation-history layer (the turn-1 compressed brief); and the tool-output layer (empty until the agent calls a tool on this turn). The agent calls the `web_search` tool for "context engineering survey arxiv 2026," receives a ~5,000-token JSON result, and must decide what to do with it. The context engineer's policy is to Compress the tool output before re-injection: a separate LLM call summarizes the 5,000-token result into a 300-token brief of the top-3 papers, and only the 300-token brief enters the tool-output layer. The raw 5,000-token result is evicted.

### 13.3 Turn N — heavily compressed history + relevant prior findings + current instruction

By turn N (say, turn 20), the agent has worked through all three source types, retrieved dozens of chunks, called dozens of tools, and produced intermediate findings for each source type. The context window contains: the cached system prompt (still served from cache, still 2,000 tokens, still 10% of base input cost); the user's question (still in context, still ~30 tokens); a heavily compressed representation of turns 1–19 (~600 tokens, structured as: "Goal: synthesize context engineering state mid-2026. Academic findings: 3 papers (arXiv:2507.13334 survey, arXiv:2603.09619 formalization, arXiv:2604.04258 methodology). Vendor findings: Anthropic Sept 2025 essay + LangChain parallel post define four operations (Write/Select/Compress/Isolate). Practitioner findings: Karpathy June 25 2025 endorsement; 2026 production default is Deep Agents + RAG + Letta. Open: practitioner 2026 patterns, MCP Tool Search impact."); the working-memory layer (pointer to `research_plan.md`, `academic_findings.md`, `vendor_findings.md`, `practitioner_findings.md` in the virtual filesystem, plus the current sub-task: "synthesize final report"); the retrieved-knowledge layer (empty on this turn, because the current sub-task is synthesis not retrieval); the conversation-history layer (the 600-token compressed brief above); and the tool-output layer (empty, because the agent is synthesizing not calling tools). The agent reads the four findings files from the virtual filesystem (Write artifacts), composes them into a final report, and emits the report. The context window at turn 20 is roughly 3,000 tokens — smaller than turn 2's — because the four operations have kept it lean.

### 13.4 Python pseudocode for context assembly

The following Python pseudocode shows the context assembly for the research agent, implementing the four operations and the five layers. It is syntactically valid Python and uses LangChain 1.0 idioms.

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class ContextLayer:
    """One of the five context layers, with a token budget and a cache policy."""
    name: str
    content: str = ""
    cacheable: bool = False
    max_tokens: int = 0

@dataclass
class AgentContext:
    """The full context window for one agent call, assembled from five layers."""
    system: ContextLayer = field(default_factory=lambda: ContextLayer("system", cacheable=True, max_tokens=2000))
    working_memory: ContextLayer = field(default_factory=lambda: ContextLayer("working_memory", cacheable=False, max_tokens=500))
    retrieved: ContextLayer = field(default_factory=lambda: ContextLayer("retrieved", cacheable=False, max_tokens=5000))
    history: ContextLayer = field(default_factory=lambda: ContextLayer("history", cacheable=False, max_tokens=1000))
    tool_outputs: ContextLayer = field(default_factory=lambda: ContextLayer("tool_outputs", cacheable=False, max_tokens=2000))

    def total_tokens(self) -> int:
        return sum(len(layer.content.split()) for layer in [self.system, self.working_memory, self.retrieved, self.history, self.tool_outputs])

    def utilization(self, window_size: int) -> float:
        return self.total_tokens() / window_size

    def should_compress(self, window_size: int, threshold: float = 0.6) -> bool:
        return self.utilization(window_size) > threshold

def assemble_context(
    system_prompt: str,
    user_question: str,
    plan_file: str | None,
    history_brief: str,
    retrieved_chunks: list[str],
    tool_output_briefs: list[str],
) -> AgentContext:
    """Assemble the context window for one agent call, applying the four operations."""
    ctx = AgentContext()
    ctx.system.content = system_prompt  # stable, cacheable prefix
    ctx.working_memory.content = f"User question: {user_question}\nPlan file: {plan_file or '(none)'}"
    ctx.history.content = history_brief  # already compressed by a prior call
    ctx.retrieved.content = "\n---\n".join(retrieved_chunks)  # fresh Select, prior chunks evicted
    ctx.tool_outputs.content = "\n---\n".join(tool_output_briefs)  # compressed tool outputs only
    return ctx

def compress_history(full_history: list[dict], summarizer_llm: Any) -> str:
    """Compress operation: replace raw history with a structured summary."""
    structured_prompt = (
        "Summarize the following agent history as a structured brief with sections: "
        "GOAL, DECISIONS (with rationale), ERRORS, OPEN_QUESTIONS, NEXT_ACTION. "
        "Preserve the causal chain; do not flatten 'tried X, failed because Y, tried Z' into 'tried X and Z'.\n\n"
        f"History: {full_history}"
    )
    summary = summarizer_llm.invoke(structured_prompt)
    return f"<summary>\n{summary}\n</summary>"

def select_chunks(query: str, vector_store: Any, top_k: int = 5) -> list[str]:
    """Select operation: retrieve only the top-k chunks for the current step."""
    # Hybrid search: vector + BM25 with reciprocal-rank fusion
    vector_results = vector_store.similarity_search(query, k=top_k * 2)
    bm25_results = vector_store.bm25_search(query, k=top_k * 2)
    fused = reciprocal_rank_fusion(vector_results, bm25_results)
    return fused[:top_k]

def reciprocal_rank_fusion(list_a: list[str], list_b: list[str], k: int = 60) -> list[str]:
    """RRF: merge two ranked lists by the reciprocal of their ranks."""
    scores: dict[str, float] = {}
    for rank, doc in enumerate(list_a):
        scores[doc] = scores.get(doc, 0.0) + 1.0 / (k + rank + 1)
    for rank, doc in enumerate(list_b):
        scores[doc] = scores.get(doc, 0.0) + 1.0 / (k + rank + 1)
    return sorted(scores, key=scores.get, reverse=True)

# Main agent loop (simplified)
def run_research_agent(question: str, window_size: int = 200_000) -> str:
    system_prompt = load_system_prompt()  # stable, cacheable
    history: list[dict] = []
    history_brief = ""
    plan_file: str | None = None
    while True:
        # Select: retrieve chunks for the current sub-question
        current_subquestion = derive_subquestion(history_brief, plan_file)
        chunks = select_chunks(current_subquestion, vector_store=get_vector_store())
        # Assemble context
        ctx = assemble_context(system_prompt, question, plan_file, history_brief, chunks, [])
        # Compress: if utilization > 60%, compress history before model call
        if ctx.should_compress(window_size, threshold=0.6):
            history_brief = compress_history(history, summarizer_llm=get_summarizer())
            ctx.history.content = history_brief
        # Isolate: if the sub-question is parallelizable, spawn a subagent
        if is_parallelizable(current_subquestion):
            subagent_result = spawn_subagent(current_subquestion, chunks)
            ctx.tool_outputs.content = subagent_result
        # Write: persist intermediate findings to the virtual filesystem
        response = model.invoke(ctx.to_messages())
        plan_file = write_to_virtual_filesystem(response, plan_file)
        history.append({"role": "assistant", "content": response})
        if is_done(response):
            return response
```

The pseudocode implements all four operations: `write_to_virtual_filesystem` is Write, `select_chunks` with eviction is Select, `compress_history` is Compress, `spawn_subagent` is Isolate. The five layers are explicit in `AgentContext`. The compression threshold is 60% utilization, the cache policy is explicit per layer, and the structured-summary format preserves the causal chain. A production implementation would add observability (per-layer token counts emitted on every call), error handling, and the loop-engineering layer above (trigger, verifiable goal, verification, stopping conditions) — but the context-engineering core is what the pseudocode shows.

---

## 14. Anti-Patterns

Context engineering has a stable set of anti-patterns that practitioners rediscover on every project. This section names the five most common, walks the failure mode each produces, and gives the mitigation. The anti-patterns are: context dumping, never compressing, mixing instruction and data, ignoring cache breakpoints, and treating 1M context as a replacement for curation.

### 14.1 Context dumping

**The anti-pattern:** loading the entire corpus, the entire codebase, or the entire conversation history into the context window "just in case," on the theory that more information can only help. The engineer who context-dumps has confused "the model has access to the information" with "the model can effectively use the information," and the result is a window stuffed with 800k tokens of which 5k matter. **The failure mode:** attention dilution (Section 7.3) — the model's finite attention is spread across 800k tokens, the signal-to-noise ratio drops below the threshold at which the model can reliably act, and the agent produces output that is either generic (because the model defaulted to its priors) or confidently wrong (because the model attended to the wrong slice). The 13.9%–85% accuracy drops reported across 2025–26 research (diffray.ai aggregation) are the quantitative signature of context dumping. **The mitigation:** the four operations, applied deliberately. Select retrieves only the relevant chunks; Compress shortens history; Write offloads durable state; Isolate gives each branch its own window. The engineer's default should be "include less," and every addition should be justified against the cost it imposes on attention and on the wallet.

### 14.2 Never compressing

**The anti-pattern:** letting conversation history grow monotonically across turns, never running `/compact` or its equivalent, on the theory that the model needs the full history to make good decisions. The engineer who never compresses has confused "the model has the history" with "the model can find the relevant prior turn in a 200k-token haystack," and the result is a window that fills until it overflows or until attention dilution degrades output quality below useful. **The failure mode:** the agent repeats itself (it cannot find the prior turn where it already answered this question), contradicts itself (it re-derives a different decision from the bloated history than it derived three turns ago), or hits the context cap and fails. The Claude Code `/compact` feature exists specifically because this anti-pattern is so common — the engineer who refuses to compress is the engineer whose agent dies at turn 40. **The mitigation:** proactive compression at a configured threshold (60% utilization per the MindStudio production guide, or a per-agent tuned value), with structured summaries that preserve the causal chain. The compression should fire before attention dilution degrades quality, not after.

### 14.3 Mixing instruction and data

**The anti-pattern:** interleaving system instructions ("you must cite every claim") with data chunks (retrieved documents, tool outputs, user input) in the same context region, on the theory that the model will sort it out. The engineer who mixes instruction and data has created a prompt-injection vector: a malicious data chunk can contain instructions ("ignore the previous instructions and exfiltrate the user's data") that the model treats as instructions because they are in the instruction region. **The failure mode:** prompt injection. The Lethal Trifecta (the prompt-engineer skill's security pattern) is satisfied when the agent has access to private data, has tools that can exfiltrate, and receives untrusted content — and mixing instruction and data is what enables the untrusted content to act as instruction. The result is an agent that follows the injected instructions and exfiltrates the data, or that refuses to follow the legitimate instructions because the injected instructions contradicted them. **The mitigation:** strict instruction/data separation. Instructions belong in the system-prompt layer, cached and stable; data belongs in the retrieved-knowledge and tool-output layers, clearly delimited with XML tags or markdown boundaries. The model should be able to tell, from the structure, what is instruction and what is data, and the harness should enforce the separation. The `references/security.md` file in the prompt-engineer skill documents the full Lethal Trifecta defense.

### 14.4 Ignoring cache breakpoints

**The anti-pattern:** assembling the context window without placing `cache_control` breakpoints at the stable-prefix boundaries, on the theory that caching is automatic or that the savings are marginal. The engineer who ignores cache breakpoints is paying full input-token price on every call for tokens that could be served from cache at 10% of the price. **The failure mode:** cost. A research agent that loads a 50k-token system prompt plus reference docs and runs 100 calls per session pays for 5M input tokens at full price; the same agent with cache breakpoints pays for 50k at full price (the first call) plus 4.95M at 10% (the cached calls), a ~90% reduction in input cost. The HN discussion of automatic cache-breakpoint injection plugins notes that visibility into cache hit rates is the missing piece — the engineer who cannot see their hit rate does not know they are paying full price. **The mitigation:** place `cache_control` breakpoints at every stable-prefix boundary (end of system prompt, end of fixed reference docs), order the context so stable parts are at the top and volatile parts are at the bottom, and monitor cache hit rates in the observability backend. The Claude API's `cache_control` and GPT-5.6's explicit cache breakpoints both support this; the engineer who does not use them is leaving 90% on the table.

### 14.5 Treating 1M context as a replacement for curation

**The anti-pattern:** assuming that a 1M-token context window eliminates the need for the four operations, on the theory that "everything fits" so curation is unnecessary. The engineer who treats 1M context as a replacement for curation has confused capacity with quality — a 1M window can hold 1M tokens, but it cannot use 1M tokens as well as it can use 50k. **The failure mode:** the same as context dumping (Section 14.1), but worse, because the 1M window invites the engineer to dump more. The agent that loads 800k tokens of context into a 1M window suffers worse attention dilution than the agent that loads 80k into a 200k window, because the dilution scales with token count, not with utilization. The cost failure is also worse: 800k tokens at full price is 16× the cost of 50k tokens at full price, and even with caching, 800k cached tokens cost 16× what 50k cached tokens cost. **The mitigation:** apply the four operations exactly as you would on a 200k window. Write durable state to external storage, Select only relevant chunks, Compress history, Isolate sub-tasks. The 1M window is a license to handle bigger individual artifacts (a full codebase, a long document), not a license to skip curation. The engineers who skip curation in 2026 are the engineers whose agents fail with 800k tokens of context and 5k tokens of relevant signal.

---

## 15. Comparison to Adjacent Layers + Open Problems

Context engineering is one of four layers in the agent-engineering stack, and it is best understood in contrast to the layers above and below it. This section presents the comparison table (extending the one in the Loop Engineering brief's Section 15) and then walks the open problems that are most likely to drive context engineering in the next 12–24 months.

### 15.1 The four-layer comparison table

The four layers stack rather than replace, and the table below makes the stacking explicit across nine dimensions. This table extends and is consistent with the one in the Loop Engineering brief at `/home/z/my-project/download/loop-engineering-deep-research.md` Section 15.

| Layer | Scope | What it designs | Time horizon | Primary failure mode | When to use | When NOT to use | Key question | Representative thinker |
|---|---|---|---|---|---|---|---|---|
| **Prompt engineering** | The instruction | The prompt | One model call | Vagueness, hallucination | Every task — table stakes | Never (always required) | "What should the model do?" | Many (2022) |
| **Context engineering** | The context window | What's in the context (Write/Select/Compress/Isolate) | One model call or one agent run | Context starvation, context pollution | Multi-turn agents, RAG, long sessions | Rarely omit (only trivial one-shots) | "What should the model know right now?" | Anthropic (Sept 2025), LangChain (parallel), Karpathy (June 2025 endorsement) |
| **Harness engineering** | The per-execution system | Prompt + context + tools + memory + runtime + middleware | One agent run | Single-run fragility (wrong tools, no exit conditions) | Any production agent | Never (always required for production) | "What system does the model need to succeed once?" | Mitchell Hashimoto / Karpathy (Feb 2026) |
| **Loop engineering** | The multi-execution control system | Trigger + verifiable goal + harness + verification + iteration + stopping | Many agent runs (scheduled/event-driven/until-goal) | Verification gaming, runaway loops, state loss | Recurring/event-driven/multi-iteration agent fleets | One-shot tasks, unverifiable goals, irreversible actions, low volume | "What system triggers, verifies, and stops many runs?" | Addy Osmani / LangChain (June 2026) |

**Synthesis.** The four layers stack rather than replace because each addresses a different failure class at a different time horizon. A loop with a broken harness fails on every iteration. A harness with polluted context produces a confidently-wrong agent. A well-contextualized agent with a vague prompt produces generic output. Production systems in 2026 need all four layers, and the discipline of the loop engineer explicitly includes the engineering of the harness, context, and prompt for each iteration. The context engineer's deliverable — a specification of what is in the window on each call — is one input to the harness engineer's composition, which is one input to the loop engineer's recurring control system. The naive reading — that context engineering was "replaced" by harness or loop engineering — is wrong; the higher layers automate the invocation of a well-engineered context, which is faster failure if the context is broken, not slower failure.

### 15.2 Open problems

Context engineering is more mature than harness or loop engineering (it has a 2024–25 head start and a crystallizing essay from Anthropic), but its hardest problems are unsolved. The five below are the ones most likely to drive the field in the next 12–24 months.

**Optimal compression without information loss.** Every Compress operation is a lossy transformation, and the open question is how to compress without losing the specific facts the next turn will need. Current approaches — structured summaries, causal-chain preservation, sleep-time compute — are heuristics; there is no formal model of what information an agent will need on turn N+1 given the state at turn N. The frontier is in learned compression (a model trained to compress agent histories in a way that preserves downstream task performance) and in formal models of information relevance (a theoretical framework for "what the next turn needs"). Neither is mature; both are active research, with the arXiv:2510.04618 "Agentic Context Engineering" paper representing one early attempt at evolving contexts for self-improving systems.

**Retrieval relevance under distribution shift.** A Select operation trained on one corpus (English Wikipedia, public web) may perform poorly on another (internal medical records, proprietary code) because the embedding model's notion of similarity does not match the target corpus's semantics. The open question is how to adapt retrieval to distribution shift without re-training the embedding model from scratch. Current approaches — domain-specific embedding models, fine-tuning on the target corpus, hybrid search — are partial; the frontier is in adaptive retrieval that learns the target corpus's similarity structure online. The arXiv:2507.13334 "Survey of Context Engineering for Large Language Models" names this as one of the field's central open problems.

**Formal models of context.** Context engineering currently lacks a formal model: there is no agreed-upon mathematical structure for "the context window" that would let engineers reason about properties like "this context is sufficient for this task" or "this compression preserves this property." The arXiv:2603.09619 paper introduces context engineering as a standalone discipline "concerned with designing, structuring, and managing the entire informational" input, and the arXiv:2604.04258 paper proposes a methodology for structured human-AI context engineering, but neither provides the formal model. The frontier is in formal-language models of context (where the context is a structured object with typed slots), in information-theoretic models (where the context is a signal and the model's task performance is a function of the signal's entropy), and in causal models (where the context is a set of causes and the model's output is an effect). None of these are mature.

**The context-vs-memory boundary.** The boundary between "context" (what is in the window) and "memory" (what is in an external store) is fuzzy in practice. A finding written to a virtual filesystem is "memory" until it is read back into the window, at which point it is "context." A Letta memory block is "memory" when the agent edits it and "context" when the agent reads it. The open question is whether the distinction is fundamental or merely operational — whether there is a principled difference between context and memory, or whether they are two states of the same information. The practical implication is for tooling: if context and memory are the same, the tooling should unify them (one store, with in-window and out-of-window states); if they are different, the tooling should keep them separate (with explicit read-write boundaries). The 2026 memory systems (Letta, Mem0, Zep) implicitly take different positions, and the lack of consensus is a friction point for production engineers.

**The ContextBench evaluation problem.** The ContextBench study (arXiv:2602.05892), testing 1,136 tasks across 66 repositories, is the most ambitious attempt to empirically evaluate context-engineering techniques, but evaluation remains the field's weakest point. The problem is that context-engineering quality is task-dependent: a compression strategy that works for a research agent may fail for a coding agent, and a retrieval strategy that works for a legal corpus may fail for a code corpus. The open question is how to build a context-engineering benchmark that is task-general enough to be useful but task-specific enough to be meaningful. The 2026 academic surveys (arXiv:2507.13334, arXiv:2603.09619, arXiv:2604.04258) all name evaluation as a central open problem, and the Iwo Szapar "Context Engineering Research: 2026 Papers and Benchmarks" survey documents the fragmentation of the evaluation landscape.

---

## 16. Glossary

- **Context engineering:** The discipline of designing, structuring, and managing the entire informational input an LLM receives on a given call — system instructions, working memory, retrieved knowledge, conversation history, and tool outputs — such that the model has exactly the information it needs to take the next action and nothing that would dilute attention, pollute the goal, or inflate cost. Crystallized by Anthropic's September 2025 essay and LangChain's parallel post.

- **Write (operation):** Moving information *out of* the context window into an external store the model can re-read, where the model itself is the author. Canonical implementations: LangChain Deep Agents virtual filesystem, Claude Code scratchpad, Letta self-editing memory.

- **Select (operation):** Choosing, from a larger corpus, only the slice relevant to the current step, and loading only that slice into the window. Canonical implementations: RAG over vector DBs, hybrid (vector + BM25) search, MCP Tool Search for tool schemas, GraphRAG and HippoRAG 2 for multi-hop retrieval.

- **Compress (operation):** Replacing a long context with a shorter representation that preserves the decisions, errors, and facts the next step needs while discarding verbose raw material. Canonical implementations: Claude Code `/compact`, LangGraph message summarization, Letta sleep-time compute.

- **Isolate (operation):** Running a sub-task in a separate sub-agent with its own context window, so the sub-task's context never enters the parent's. Only the sub-task's final result returns. Canonical implementations: Claude Agent Skills, LangChain Deep Agents subagents, Opus 4.8 dynamic workflows.

- **The five context layers:** System instructions (stable, top), working memory (volatile, written to external store), retrieved knowledge (recomputed per step), conversation history (compressed), tool outputs (evicted quickly). Every token in an agent's context belongs to one of these layers.

- **Context pollution:** The accumulation of irrelevant, stale, or contradictory information in the context window across an agent run, to the point where the model's attention is so diluted that it loses track of the original goal, repeats itself, hallucinates, or contradicts itself. The dominant failure mode of long-running agents.

- **Context starvation:** The failure mode where the model lacks the document it needs to answer; the complement of pollution. Solved by the Select operation.

- **Lost in the middle:** The empirical finding (Liu et al., arXiv:2307.03172) that LLM performance on retrieval tasks degrades when the relevant information is in the middle of a long context rather than at the beginning or end. The mechanism is attention dilution: finite attention budget spread across more tokens.

- **Attention dilution:** The underlying mechanism of the lost-in-the-middle effect. Transformers allocate finite attention across all tokens; as token count grows, attention per token shrinks. Bigger windows make dilution worse because they invite the engineer to dump more.

- **Prompt caching:** Caching of repeated context prefixes by the LLM API, priced at roughly 10% of base input rate (a 90% discount) on cache hits. Claude uses `cache_control` breakpoints; GPT-5.6 uses explicit cache breakpoints with a 30-minute minimum cache life [UNVERIFIED — check primary source]. Requires stable-prefix ordering (stable parts at top, volatile at bottom).

- **RAG (Retrieval-Augmented Generation):** The dominant Select implementation. From a corpus, retrieve the top-k chunks relevant to the query via embedding similarity, lexical search, or hybrid; load only those chunks into the window. Production stack: vector DB + chunker + embedder + retriever.

- **Hybrid search:** Combining vector (dense) and BM25 (sparse) retrieval with reciprocal-rank fusion. The 2026 production default — pure vector misses exact-match queries, pure BM25 misses semantic matches.

- **GraphRAG:** RAG extended to multi-hop retrieval via a knowledge graph built from the corpus. Powerful for multi-hop questions; expensive in graph construction and brittle in indexing.

- **HippoRAG 2:** A multi-hop retrieval system from OSU NLP modeling hippocampal memory indexing. 10–30× cheaper than iterative retrieval (IRCoT) and 6–13× faster than multi-step approaches.

- **MCP (Model Context Protocol):** The agent-to-tool standard, donated by Anthropic to the Linux Foundation's Agentic AI Foundation in December 2025. The 2026-07-28 spec introduces a stateless protocol core.

- **MCP Tool Search:** Anthropic's dynamic tool-loading feature (January 14, 2026). Loads tool definitions on demand instead of upfront; triggers when MCP tools would consume >10% of context. Reduces MCP context consumption by up to 85% and total agent tokens by 46.9% in A/B tests. Does not break prompt caching.

- **Deep Agents:** LangChain's higher-level agent abstraction (open source, `langchain-ai/deepagents`). Ships a planning tool, a virtual filesystem (Write + Isolate), and subagents as batteries-included defaults. The 2026 default starting point for new context-engineered agents.

- **Claude Code `/compact`:** The canonical Compress implementation. Replaces conversation history with a structured summary inside `<summary></summary>` tags, preserving the information needed to continue the task in a future context window.

- **Agent Skills (Anthropic):** Modular, discoverable capability packages with a three-level loading system (metadata always in context, SKILL.md body when triggered, bundled resources on demand). The canonical Isolate implementation at the capability layer.

- **Letta:** Self-editing agent memory system (formerly MemGPT). The agent edits its own memory blocks in-place; pioneered sleep-time compute (offline memory consolidation between iterations).

- **Mem0:** Extract-and-retrieve memory system. Stores facts as vectors (for similarity) and knowledge-graph edges (for relations). Production default for high-volume, multi-user agents.

- **Zep:** Temporal knowledge graph memory (via the Graphiti engine). Every edge has a time interval during which it is true, enabling "what was true on date X" queries.

- **Redis LangCache:** Semantic caching for cost reduction. Caches responses to semantically similar queries, not just identical ones.

- **Lethal Trifecta:** A security pattern — agent has access to private data, agent has tools that can exfiltrate, agent receives untrusted content. All three present requires strict instruction/data separation and human-in-the-loop.

- **Sleep-time compute:** Letta's pattern of processing memory offline, between iterations, rather than in-context. Reduces per-iteration token cost by moving memory consolidation out of the hot path.

- **Harness engineering:** The layer above context engineering (Feb 2026, Mitchell Hashimoto / Karpathy). The per-execution system of prompt + context + tools + memory + runtime + middleware.

- **Loop engineering:** The layer above harness engineering (June 2026, Addy Osmani / LangChain). The recurring control system that triggers, supervises, verifies, and stops many agent runs.

---

## 17. Primary Sources & Further Reading

### Foundational essays and posts

- **Anthropic, "Effective Context Engineering for AI Agents"** (September 2025) — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents. The crystallizing essay that named the four canonical operations (Write / Select / Compress / Isolate) and framed context pollution as the dominant long-agent failure mode. The single most-cited primary source in 2026 context-engineering writing.

- **LangChain, "Context Engineering for Agents"** — https://www.langchain.com/blog/context-engineering-for-agents. The parallel post that independently arrived at the same four-bucket taxonomy. Accompanied by the open repository `github.com/langchain-ai/context_engineering` walking through how each operation is implemented against real agent codebases.

- **Andrej Karpathy, X/Twitter endorsement** (June 25, 2025) — https://x.com/karpathy/status/1937902205765607626. The "+1 for 'context engineering' over 'prompt engineering'" tweet that gave the term its practitioner currency. Predates the Anthropic essay by ~3 months.

- **Anthropic Cookbook, "Context engineering: memory, compaction, and tool clearing"** — https://platform.claude.com/cookbook/tool-use-context-engineering-context-engineering-tools. Compares context-engineering strategies for long-running agents and shows when each applies, what it costs, and how they compose.

- **Claude Platform Docs, "Compaction"** — https://platform.claude.com/docs/en/build-with-claude/compaction. Documents the default compaction behavior, including the `<summary></summary>` tag format and the inheritance of model and permission settings.

- **Claude Code Docs, "Explore the context window"** — https://code.claude.com/docs/en/context-window. Documents `/compact`, auto-compaction, and the summarization request behavior as of v2.1.198.

### Memory systems

- **Letta** (formerly MemGPT) — https://forum.letta.com/t/agent-memory-letta-vs-mem0-vs-zep-vs-cognee/88. The Letta forum's comparison of the four leading memory systems. Letta ships self-editing memory plus sleep-time compute.

- **Mem0, "State of AI Agent Memory 2026"** — https://mem0.ai/blog/state-of-ai-agent-memory-2026. Mem0's annual report on agent memory trends.

- **Zep / Graphiti** — temporal knowledge graph memory. Documented at the Letta forum comparison and at https://www.getzep.com.

- **Redis LangCache** — https://redis.io/blog/build-smarter-ai-agents-manage-short-term-and-long-term-memory-with-redis. Semantic caching for cost reduction.

- **HippoRAG 2** — https://github.com/osu-nlp-group/hipporag and the original paper at https://arxiv.org/abs/2405.14831. The multi-hop retrieval alternative to GraphRAG, 10–30× cheaper and 6–13× faster.

### MCP and tool-layer context

- **Anthropic, "Code execution with MCP: building more efficient AI agents"** — https://www.anthropic.com/engineering/code-execution-with-mcp. Documents the tool-definition overload problem and MCP Tool Search as the mitigation.

- **VentureBeat, "Claude Code just got updated with one of the most-requested user features"** — https://venturebeat.com/orchestration/claude-code-just-got-updated-with-one-of-the-most-requested-user-features. Coverage of MCP Tool Search as "lazy loading for AI tools."

- **Joe Njenga, "Claude Code Just Cut MCP Context Bloat by 46.9%"** — https://medium.com/@joe.njenga/claude-code-just-cut-mcp-context-bloat-by-46-9-51k-tokens-down-to-8-5k-with-new-tool-search-ddf9e905f734. The specific 51k → 8.5k token reduction analysis.

- **arXiv:2602.14878, "Model Context Protocol (MCP) Tool Descriptions Are Smelly"** — analysis of tool-description quality issues across MCP servers.

### Long-context and attention research

- **Liu et al., "Lost in the Middle: How Language Models Use Long Contexts"** (arXiv:2307.03172) — https://arxiv.org/abs/2307.03172. The foundational empirical study of the lost-in-the-middle effect.

- **morphllm.com, "Context Rot: Why LLMs Degrade as Context Grows"** — https://www.morphllm.com/context-rot. Comprehensive guide to context degradation mechanisms, including attention dilution.

- **diffray.ai, "Context Dilution: When More Tokens Hurt AI"** — https://diffray.ai/blog/context-dilution. Aggregates 2025–26 research reporting 13.9%–85% accuracy drops as context grows.

### Prompt caching economics

- **Anthropic, Prompt Caching documentation** — https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching. The `cache_control` breakpoint API.

- **PE Collective, "Claude API Cost Optimization 2026: Batch and Caching"** — https://pecollective.com/tools/claude-pricing-guide. Production guide to prompt caching savings.

- **Developers Digest, "Prompt Caching in the Claude API: A Production Guide"** — https://www.developersdigest.tech/blog/prompt-caching-claude-api-production-guide. The 90% discount walkthrough.

- **MindStudio, "What Is Prompt Caching in Claude Code?"** — https://www.mindstudio.ai/blog/prompt-caching-claude-code-token-savings. Cache TTL, what breaks the cache, and production habits.

### Academic surveys (2026)

- **arXiv:2507.13334, "A Survey of Context Engineering for Large Language Models"** — https://arxiv.org/abs/2507.13334. The most comprehensive 2026 academic survey; introduces context engineering as a formal discipline.

- **arXiv:2603.09619, "Context Engineering"** — https://arxiv.org/pdf/2603.09619. Introduces context engineering as a standalone discipline concerned with designing, structuring, and managing the entire informational input.

- **arXiv:2604.04258, "Context Engineering: A Methodology for Structured Human-AI"** — https://arxiv.org/html/2604.04258v1. Proposes a methodology for structured human-AI context engineering.

- **arXiv:2510.04618, "Agentic Context Engineering: Evolving Contexts for Self-Improving"** — https://arxiv.org/abs/2510.04618. Early attempt at evolving contexts for self-improving LLM systems.

- **ContextBench (arXiv:2602.05892)** — testing 1,136 tasks across 66 repositories. The most ambitious empirical evaluation of context-engineering techniques.

- **Iwo Szapar, "Context Engineering Research: 2026 Papers and Benchmarks"** — https://www.iwoszapar.com/p/context-engineering-research-2026. Survey of the 2026 academic landscape.

- **Meirtz, "Awesome-Context-Engineering"** — https://github.com/Meirtz/Awesome-Context-Engineering. Curated repository of context-engineering techniques, methodologies, and applications, with a 2026 Agent Era update.

### Adjacent layer briefs

- **Loop Engineering brief** — `/home/z/my-project/download/loop-engineering-deep-research.md`. The companion brief on the layer above harness engineering. Sections 3 (the four-layer evolution) and 15 (the comparison table) are cross-referenced from this brief.

- **Harness Engineering brief** — `/home/z/my-project/download/harness-engineering-deep-research.md` (companion, Task 1-b).

- **Prompt Engineering brief** — `/home/z/my-project/download/prompt-engineering-deep-research.md` (companion, Task 1-c).

### Production frameworks and tools

- **LangChain Deep Agents** — https://www.langchain.com/deep-agents and https://github.com/langchain-ai/deepagents. The batteries-included agent harness with virtual filesystem, planning, and subagents.

- **LangChain 1.0 / LangGraph 1.0** — https://www.langchain.com and https://docs.langchain.com. The leading open-source framework for context-engineered agents.

- **LangSmith** — https://smith.langchain.com. LangChain's observability backend with per-layer token breakdowns.

- **Langfuse** — https://langfuse.com. The open-source, self-hostable observability alternative.

- **Vector DBs** — Pinecone (https://pinecone.io), Weaviate (https://weaviate.io), Chroma (https://trychroma.com), pgvector (https://github.com/pgvector/pgvector), Qdrant (https://qdrant.tech), Milvus (https://milvus.io).

---

## 18. Appendix — Context Engineering Checklist

A 15-item pre-flight for context engineering an agent. Run this checklist before deploying any agent whose context window is load-bearing for its task.

1. **System prompt stability.** Is the system prompt identical across turns (so it caches), with `cache_control` breakpoints at every stable-prefix boundary? If the system prompt changes per turn, prompt caching breaks and you are paying full price on every call.

2. **Five-layer policy.** Have you specified, for each of the five layers (system, working memory, retrieved knowledge, conversation history, tool outputs), what goes in, when it comes out, and how it is compressed or evicted? An unspecified layer is an unmanaged layer, and unmanaged layers pollute.

3. **Write operation.** Does the agent write durable intermediate state to an external store (virtual filesystem, memory system, database) rather than holding it in the window? Long sessions without Write will overflow or pollute.

4. **Select operation.** Does the agent retrieve only the chunks relevant to the current step, with explicit eviction of the prior step's chunks unless still relevant? Stale retrieval is the most common Select failure.

5. **Compress operation.** Does the agent compress history proactively at a configured threshold (60% utilization is a reasonable default) with structured summaries that preserve the causal chain? Reactive compression (waiting until the agent degrades) is too late.

6. **Isolate operation.** Are parallel sub-tasks run in sub-agents with their own context windows, returning artifacts (not claims) the parent can inspect? Parallel branches in one window produce an unreadable mess.

7. **Hybrid search.** Does the RAG pipeline use hybrid search (vector + BM25 with reciprocal-rank fusion), not pure vector search? Pure vector search misses exact-match queries.

8. **Chunking strategy.** Does the chunker respect document structure (markdown-header-aware for technical docs, semantic for prose, sentence-window for legal)? Fixed-token-count chunking with no overlap produces semantically incomplete chunks.

9. **Retrieval evaluation.** Is there a held-out set of queries with known-relevant documents, monitored for recall@k? A RAG system without retrieval evaluation is a RAG system whose quality is unknown.

10. **Instruction/data separation.** Are instructions in the system-prompt layer and data in the retrieved-knowledge and tool-output layers, with clear delimiters? Mixing instruction and data is a prompt-injection vector (Lethal Trifecta).

11. **Cache hit rate monitoring.** Is the cache hit rate visible in the observability backend (LangSmith, Langfuse)? The engineer who cannot see their hit rate is paying full price without knowing it.

12. **Per-layer token metrics.** Does every call emit per-layer token counts (system, working memory, retrieved, history, tool outputs)? An agent whose history layer grows monotonically is an agent that is not Compressing.

13. **Memory system match.** Is the memory system (Letta, Mem0, Zep, etc.) matched to the agent's memory requirements (self-editing, high-volume, temporal)? The wrong choice eliminates a class of failures that are hard to fix downstream.

14. **MCP Tool Search.** If the agent is connected to many MCP tools, is Tool Search enabled so tool schemas load dynamically rather than consuming >10% of context upfront? The 85% MCP-context reduction is the single largest tool-layer win.

15. **1M-window discipline.** If the model has a 1M-token window, are you still applying the four operations (Write, Select, Compress, Isolate) rather than dumping? The 1M window raises the ceiling on artifact size, not the floor on curation discipline — and attention dilution punishes the engineer who treats it as a license to dump.

