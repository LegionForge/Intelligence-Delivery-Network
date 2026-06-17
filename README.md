# 🧠 Intelligence Delivery Network (IDN) - Development on Hold see [LegionForge](https://github.com/LegionForge) or https://legionforge.org
https://github.com/LegionForge/Intelligence-Delivery-Network
> *"CDN for Intelligence" — Route AI workloads to the right compute tier based on what the request actually needs.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: Prototype](https://img.shields.io/badge/Status-Prototype-blue.svg)](ROADMAP.md)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## The Big Idea

Today, most LLM applications route every request — whether it\'s "what\'s the capital of France?" or "design a distributed database from scratch" — to the same large, expensive, slow frontier model. This is like sending every HTTP request to a data center on the other side of the world instead of using a CDN.

**IDN fixes this.** Inspired by how Content Delivery Networks (CDNs) cache and route content at the optimal network node, IDN routes intelligence workloads to the optimal compute tier based on a real-time analysis of the request: its complexity, size, domain, latency requirements, privacy constraints, and which tools or agents are needed.

```
CDN:  [Browser Cache] → [Edge PoP]    → [Regional Cache] → [Origin Server]
IDN:  [On-Device L0] → [Edge L1]     → [Regional L2]    → [Global L3]
```

The result: **the vast majority of requests handled at L0/L1/L2 cost and speed**, with L3 reserved only for genuinely hard problems — estimated 10x cost reduction, 10x latency improvement for typical workloads with no quality loss.

---

## The Four Tiers

| Tier | Name | Location | Latency | Cost | Model Size | Hardware | Primary Use Cases |
|------|------|----------|---------|------|------------|----------|-------------------|
| **L0** | **On-Device** | Phone / PC / IoT | ~5ms | $0.00000 | 0.5–3B | Apple ANE, Qualcomm Hexagon NPU, AMD Ryzen NPU | Offline, privacy-forced, instant chat, autocomplete, PII preprocessing |
| **L1** | **Edge** | ISP / local server / CDN node | ~15–50ms | ~$0.000001 | 3–8B | Cloudflare Workers AI, Groq LPU, local GPU | Fast RAG, extraction, classification, real-time tools |
| **L2** | **Regional** | Nearest cloud region | ~150–500ms | ~$0.001 | 30–70B | AWS/GCP/Azure regional, Groq clusters | Code generation, summaries, document workflows, tool agents |
| **L3** | **Global** | Centralized supercluster | ~1–5s | ~$0.05–0.10 | 200B–2T | GPT-4o, Claude Opus, Gemini Ultra, DeepSeek-R1 | Deep reasoning, research synthesis, system design, multi-agent planning |

### Why Four Tiers?
The 4-tier structure maps precisely to real CDN topology (client cache → edge PoP → regional mid-tier → origin) and is validated by IEEE research on tiered LLM inference continuum architectures. Fewer than 4 tiers loses the qualitative distinctions between on-device, edge, regional, and global compute. More than 4 tiers introduces routing overhead and policy complexity that exceeds the performance gains.

### L0 Is Not Just a Cheap L1
L0 (on-device) has four unique roles no other tier can fulfill:
1. **Privacy firewall** — data physically cannot leave the device (HIPAA, GDPR, attorney-client)
2. **Offline fallback** — the only tier that works with zero connectivity
3. **PII preprocessor** — sanitizes prompts before escalating to cloud tiers
4. **Escalation source** — partial results and context travel upward gracefully if complexity is exceeded mid-generation

---

## Core Architecture

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                        IDN PIPELINE                              │
  │                                                                  │
  │  [Prompt + Context + User Prefs]                                 │
  │          ↓                                                       │
  │  ┌───────────────────────────────┐                              │
  │  │       IDN PROFILER            │  ← the core innovation       │
  │  │  • token count                │                              │
  │  │  • complexity score (0–1)     │                              │
  │  │  • reasoning hops             │                              │
  │  │  • domain classification      │                              │
  │  │  • output volume estimate     │                              │
  │  │  • latency/quality sensitivity│                              │
  │  │  • PII / compliance flags     │                              │
  │  │  • task multiplicity          │                              │
  │  │  • L0 eligibility block       │                              │
  │  └───────────────────────────────┘                              │
  │          ↓                                                       │
  │  ┌───────────────────────────────┐                              │
  │  │       IDN ROUTER              │                              │
  │  │  • maps profile → tier        │                              │
  │  │  • selects expert models      │                              │
  │  │  • builds tool/agent plan     │                              │
  │  │  • plans async subtasks       │                              │
  │  │  • sets fallback paths        │                              │
  │  └───────────────────────────────┘                              │
  │          ↓                                                       │
  │  [Execution Layer]                                               │
  │   L0 → on-device (NPU/ANE, INT4 quantized, offline-capable)     │
  │   L1 → edge      (Cloudflare/Groq, fast, ~1-8B)                 │
  │   L2 → regional  (nearest cloud GPU cluster, ~30-70B)           │
  │   L3 → global    (frontier superclusters, 200B–2T)              │
  └──────────────────────────────────────────────────────────────────┘
```

---

## The Workload Profiler

The profiler is a fast pre-inference analysis function (<100ms for heuristic mode, <10ms for rules-only) that runs on every request and outputs structured routing metadata. It is **model-agnostic, gateway-agnostic, and designed as an open standard** any LLM app or gateway can consume.

### Input
```json
{
  "prompt": "string",
  "context": "string (optional)",
  "user_history": ["array of recent turns"],
  "user_preferences": {
    "max_latency_ms": 500,
    "max_cost_usd": 0.01,
    "quality_preference": "fast | balanced | thorough"
  }
}
```

### Output — Core Routing Metadata
```json
{
  "tokens_input": 1200,
  "tokens_context": 800,
  "workload_size_score": 0.72,
  "complexity_score": 0.81,
  "reasoning_hops": 3,
  "output_volume_estimate": 1500,
  "latency_sensitivity": "medium",
  "quality_sensitivity": "high",
  "domain_tags": ["coding", "architecture"],
  "pii_risk": "low",
  "task_multiplicity": 3,
  "candidate_tiers": ["L2", "L3"],
  "recommended_tier": "L2",
  "expert_selection": [
    {
      "model": "deepseek-coder-70b",
      "provider": "groq",
      "cost_estimate": 0.002,
      "latency_estimate": 0.25
    }
  ],
  "tool_plan": ["repo_search", "code_llm", "unit_test_agent"],
  "parallelism_plan": {
    "split": true,
    "subtasks": [
      "analyze current module structure",
      "design new scalable architecture",
      "generate refactored code"
    ],
    "merge_strategy": "reduce"
  },
  "routing_decision": {
    "primary_path": "L2:deepseek-coder-70b",
    "fallback_paths": ["L3:claude-opus"]
  }
}
```

### Output — L0 Eligibility Block
```json
{
  "l0": {
    "eligible": true,
    "forced": false,
    "offline_mode": false,
    "connectivity": "good",
    "available_ram_gb": 3.2,
    "model_fits": true,
    "thermal_state": "nominal",
    "battery_pct": 72,
    "power_mode": "normal",
    "tokens_per_second": 55,
    "quantization": "int4",
    "quality_degradation_estimate": 0.12,
    "quality_floor_met": true,
    "compliance_tags": ["GDPR"],
    "pii_classes": ["location"],
    "data_egress_permitted": true,
    "preprocessing_done": false,
    "redacted_fields": [],
    "confidence_score": 0.84,
    "verification_required": false,
    "context_sync_state": "synced",
    "escalation_ready": true,
    "escalation_context_payload": null,
    "escalation_reason": null
  }
}
```

### Scoring Methods
- **`workload_size_score`**: composite of token count, reasoning hops, and estimated output volume normalized to 0–1
- **`complexity_score`**: weighted average of abstraction level, instruction density, ambiguity, and domain knowledge requirement
- **`reasoning_hops`**: count of distinct logical steps implied (e.g., "analyze → compare → synthesize → recommend" = 3)
- **`domain_tags`**: lightweight NER/regex classifier across: `general, coding, research, legal, health, finance, creative, agentic`
- **`quality_degradation_estimate`** (L0): expected quality loss vs L2 for this domain/task — legal/medical tasks degrade more than casual chat under INT4 quantization

---

## Routing Policy

| Signal | L0 On-Device | L1 Edge | L2 Regional | L3 Global |
|--------|-------------|---------|-------------|-----------|
| Input tokens | < 200 | < 500 | 500–4,000 | > 4,000 |
| Complexity score | < 0.25 | 0.25–0.45 | 0.45–0.72 | > 0.72 |
| Reasoning hops | 0 | 0–1 | 2–3 | 4+ |
| Domain | general / autocomplete | RAG / extraction | coding / tools / summaries | research / legal / multi-agent |
| PII / offline / compliance | **force L0** | consider | escalate | specialized policy |
| Latency preference | real-time / offline | fast | balanced | thorough |
| `data_egress_permitted` | only valid tier if false | — | — | blocked |
| Battery / thermal | check constraints | — | — | — |
| `quality_floor_met` | route up if false | — | — | — |

---

## L0 Decision Issues That Echo Into All Tiers

### 1. Tier Escalation Mid-Generation
A request may start at L0 but exceed capacity mid-response due to thermal throttling, RAM pressure, or unexpected complexity. IDN defines an **escalation handoff protocol**:
- L0 stores `partial_response` and `escalation_context_payload`
- Router selects escalation target (not always L1 — complexity escalations may jump to L2)
- Higher tier receives context and resumes or regenerates from the partial

### 2. Context Sync & Cache Invalidation
When L0 runs offline, higher tiers don\'t have its context. Like CDN cache invalidation, IDN tracks:
- `context_sync_state`: `synced | dirty | conflict`
- Dirty context must be pushed before escalating to cloud tiers
- Cached L0 responses carry TTL; stale responses trigger freshness escalation

### 3. L0 as PII Preprocessor for Cloud Tiers
Even when escalation to L2/L3 is needed, L0 can **redact PII from the prompt** before it leaves the device — sending only a sanitized version to cloud tiers. This is a unique architectural role: L0 acts as a compliance firewall even for requests it cannot fully answer.

### 4. Trust & Verification Propagation
L0 INT4-quantized models can hallucinate more than larger models. For moderate-confidence responses, IDN can route the L0 output to L1 for fast spot-checking before returning to the user — a cheap verification pass rather than a full re-generation.

### 5. Quality Degradation Awareness Across Tiers
L0\'s quantization-induced quality loss (`quality_degradation_estimate`) informs router decisions at *all* tiers:
- Legal, medical, financial domains: degradation penalty is high → prefer L1+ even for short prompts
- General chat, autocomplete, classification: degradation is low → L0 is safe

---

## Async Decomposition Planner

Many requests naturally split into parallel subtasks. The planner detects:
- **Task multiplicity**: multiple distinct instructions in one prompt
- **Fan-out patterns**: "for each of these N items, do X"
- **Sequential dependencies**: step A must complete before step B

It builds a DAG of subtasks, routes each to the appropriate tier independently, then merges via configurable strategy (`concat | reduce | vote`). This mirrors how CDNs chunk and parallelize large content delivery — and allows expensive L3 calls to be reserved for synthesis while cheaper tiers handle parallel sub-tasks.

---

## Competitive Landscape (Feb 2026)

### Model Providers (Tiered Internally)
- **Google**: Gemini Nano (L0/L1) → Flash (L1/L2) → Pro/Ultra (L3)
- **Anthropic**: Haiku → Sonnet → Opus with model router preview
- **OpenAI**: o1-mini → o1-preview → o4 with "thinking tokens"
- **xAI**: Grok tiered inference

### Production Routing & Gateway Tools
- **LiteLLM**: multi-model proxy with basic cost/model routing
- **Portkey**: AI gateway — multi-provider routing, observability, guardrails, fallbacks
- **TrueFoundry**: enterprise LLM control plane — governance, agents, multi-region
- **OpenRouter.ai**: 100+ models tiered by cost/latency
- **Cloudflare Workers AI**: edge-native inference (L1 target)
- **Groq LPU**: ultra-low latency inference (L1 target)
- **MLC-LLM**: on-device runtime (e.g., Llama-3.2-1B on iPhone ~100ms)

### What IDN Uniquely Adds

| Feature | Portkey / TrueFoundry | DynaRoute | **IDN** |
|---------|----------------------|-----------|---------|
| Multi-provider routing | ✅ | ❌ | ✅ |
| Explicit L0–L3 CDN tiers | ❌ | ❌ | ✅ |
| On-device (L0) as first-class tier | ❌ | ❌ | ✅ |
| Standard open profiler schema | ❌ | ❌ | ✅ |
| L0 privacy / compliance firewall | ❌ | ❌ | ✅ |
| PII preprocessing before cloud egress | ❌ | ❌ | ✅ |
| Async subtask decomposition | ❌ | ❌ | ✅ |
| Tool + agent pipeline planning | ❌ | ❌ | ✅ |
| Mid-generation escalation protocol | ❌ | ❌ | ✅ |

---

## Project Goals

1. **Open workload profiler** implementing the analysis schema — embeddable in any app or gateway
2. **Reference router** mapping profiler output to tier, expert, and tool decisions
3. **CDN-style tier spec** — clear definitions for L0–L3 nodes (capabilities, latency, cost, region, constraints)
4. **Benchmarks** showing real cost and latency savings vs always-L3 and simple rule-based baselines
5. **Personal growth** — routing algorithms, MoE gating, distributed systems, open-source stewardship

---

## Current Status

🚧 **Early Prototype (v0.1)**

- [x] Vision, tier model, and architecture designed
- [x] Full workload analysis schema drafted (core + L0 block)
- [x] Routing policy tabl
