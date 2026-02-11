# Excel vs. Web App: AI Memory Demand Model Comparison

> **Date:** February 2026
> **Models compared:**
> - **Excel:** `Tiered Memory Usage Calculator_2026.02.10 1725.xlsx`
> - **Web App:** `ai-memory-demand-model.html`

---

## Table of Contents

1. [Structural Comparison](#structural-comparison)
2. [Assumption-by-Assumption Comparison](#assumption-by-assumption-comparison)
3. [Critical Errors & Red Flags](#critical-errors--red-flags)
4. [What Each Model Gets Right](#what-each-model-gets-right)
5. [What Each Model Is Missing](#what-each-model-is-missing)
6. [Net Assessment](#net-assessment)
7. [Recommended Fixes](#recommended-fixes)
8. [Memory Supply Reality Check](#memory-supply-reality-check)
9. [Frontier Model KV Cache Reference](#frontier-model-kv-cache-reference)
10. [Batching Methodology](#batching-methodology)
11. [Sources](#sources)

---

## Structural Comparison

| Dimension | Excel Model | Web App |
|---|---|---|
| **Architecture** | Single monolithic model (4.5T params, DeepSeek-V3 "scale") | Multi-segment (Consumer/Enterprise/Agentic), model-agnostic |
| **Scenario handling** | Single point estimate + sensitivity tab | Bear/Base/Bull + Custom editable mode |
| **Time horizon** | 2026 snapshot only | 2026, 2028, 2031 projections |
| **KV cache approach** | Explicit attention head math (GQA formula) | Empirical bytes/token declining over time |
| **Memory tiers** | HBM / DRAM / Rack SSD / Datalake NAND (4 tiers) | HBM / DRAM / NAND (3 tiers) |
| **Supply-demand** | Compares to annual production | Compares to annual production with allocation % |
| **User segmentation** | Non-Agentic vs Agentic (2 types) | Consumer / Enterprise / Agentic (3 types) |
| **RAG modeling** | Embedded in datalake calc | Separate Layer 5 with hot-tier DRAM % |
| **Replication** | 3x fixed | 2.5x-3.5x (scenario-dependent) |
| **Methodology** | Single formula chain from users to peak QPS to KV to tiers | 9-layer waterfall: Users -> Tokens -> KV -> Tiers -> RAG -> Bulk -> Replication -> Totals -> Supply |

---

## Assumption-by-Assumption Comparison

| Parameter | Excel | Web App (Base 2026) | Real-World Evidence | Verdict |
|---|---|---|---|---|
| **Consumer/Non-Agentic MAU** | 1B WAU | 600M MAU | ChatGPT ~800M WAU alone; Gemini ~750M MAU; total market ~1.5-2B | Excel closer but still low; **Web app ~2-3x too low** |
| **DAU/MAU (or DAU/WAU)** | 30% (non-agentic) | 30% | ChatGPT observed ~68% DAU/MAU | **Both ~2x too low** |
| **Queries/user/day** | 10 (non-agentic) | 8 (consumer 2026) | ChatGPT: 2.5B prompts/day / ~120M DAU = ~20/day | **Both ~2-2.5x too low** |
| **Tokens/consumer query** | 1,667 (500 in + 1K RAG + 167 out) | 1,500 | Epoch AI: avg output 269 tokens; OpenRouter: avg prompt 6K (skewed by agentic); median consumer 100-400 tokens | **Both ~4-10x too high** for median consumer; reasonable if including full context window |
| **Agentic users** | 1.5B WAU (labeled "15% of regular") | 2M instances (2026) | Salesforce Agentforce: 12K customers; still very early | Excel internally inconsistent (15% of 1B != 1.5B); **Web app more conservative and plausible** |
| **Agentic tokens/task** | 90,000 (35K in + 50K RAG + 5K out) | 50,000 (2026) | Claude Code ~80K avg/thread; SWE-bench agents 100K-300K; simple agents 5K-20K | Both reasonable for complex tasks; **Excel's 50K RAG tokens is extreme** |
| **KV bytes/token** | 393,216 (384 KiB) -- GQA formula | 163,840 (0.16 MB) declining to 61,440 | Llama 3.1 405B GQA FP16: 504 KiB; FP8: 252 KiB; Llama 3.1 70B FP16: 320 KiB; FP8: 160 KiB; DeepSeek-V3 MLA: ~656 bytes (but only ~1-5% of traffic) | **Excel ~2-4x too high** (correct architecture but missing FP8/FP4 quantization); **Web app reasonable** as industry average |
| **Context length** | 1,024 "live window" | 2,000-4,000 (consumer 2026) | OpenRouter avg sequence: 5,400 tokens (late 2025), growing rapidly | **Excel too low**; Web app slightly low but reasonable for consumer median |
| **Model architecture** | Single 4.5T MoE, 96 layers, 8 KV heads | Model-agnostic (no architecture assumed) | DeepSeek-V3: 671B, 61 layers, MLA; Llama 3.1 405B: 126 layers, GQA; no 4.5T model exists in production | **Excel's hypothetical 4.5T model is speculative but GQA architecture is representative of ~95% of production traffic** |
| **Quantization** | None (FP16 throughout) | Implicitly modeled via declining bytes/token | Industry standard: FP8/FP4 KV cache, INT4/INT8 weights | **Excel misses 2-4x reduction**; Web app handles implicitly |
| **Batching/concurrency** | ~0.23-0.79 QPS/replica (no batching) | Peak factor + session duration model | Production systems: 50-500+ concurrent sequences/replica | **Excel ignores batching -> 10-100x replica overcount** |
| **Persistent KV %** | 85% reuse fraction (agentic) | 5% (consumer) / 40% (enterprise) / 80% (agentic) | Production prefix caching: 20-40% hit rate | **Excel too high**; Web app's differentiated approach is better |
| **NAND supply 2026** | ~1,114 EB | 1,200 EB (base) | ~2,800-3,000 EB (2024 was ~2,100 EB + ~20% annual bit growth) | **Both ~2-2.5x too low** |
| **DRAM supply 2026** | ~358 EiB (unit unclear) | 155 EB | Q2 2025: 76 EB -> annualized ~300+ EB; 2026 ~310-350 EB | Excel unclear (unit confusion); **Web app ~2x too low** |
| **Enterprise orgs** | N/A (not separately modeled) | 500K (2026) | OpenAI alone: 1M+ business customers; Gartner: 80%+ enterprises use GenAI by 2026 | **Web app ~4-6x too low** |
| **Replication** | 3x | 3.0x (base) | 2x-3x for data; erasure coding 1.3-1.5x for bulk | Both at upper end; **reasonable but slightly high for bulk** |
| **Memory tier split** | 50/50/0 (HBM/DRAM/SSD) for live KV | 70/25/5 (HBM/DRAM/NAND) evolving to 40/35/25 | NVIDIA ICMSP + Samsung KV offload SSDs validate tiering trend | **Web app more forward-looking and nuanced** |

---

## Critical Errors & Red Flags

### Excel: Significant (Corrected Assessment)

> **Note:** An earlier version of this analysis claimed a ~600x KV cache overestimate based on comparing the Excel's GQA architecture to DeepSeek-V3's MLA. This was misleading. MLA is used exclusively by DeepSeek (~1-5% of production traffic). The Excel's GQA with 8 KV heads is actually representative of ~95% of frontier models (Llama 3, Claude, Gemini, Qwen, Mistral). The corrected assessment follows.

| Issue | Impact | Explanation |
|---|---|---|
| **No KV cache quantization (FP16 instead of FP8/FP4)** | **~2-4x overestimate of KV cache** | The GQA architecture and 384 KiB/token formula are arithmetically correct and within range of real frontier models (Llama 3.1 405B is 504 KiB in FP16). However, production systems universally use FP8 KV cache (2x reduction, standard on H100) or FP4 (4x, emerging on Blackwell). The real KV bytes/token should be ~96-192 KiB. |
| **No continuous batching** | **~10-60x overestimate of replicas** | Implies ~0.23-0.79 QPS/replica. With continuous batching, one 8xH100 node can serve ~72-290 concurrent sequences (depending on context length), yielding ~14-58 QPS/replica at typical latencies. See [Batching Methodology](#batching-methodology) below. |
| **Agentic user count inconsistency** | **Internal contradiction** | Labels 1.5B as "15% of regular users" but 15% of 1B = 150M, not 1.5B. |
| **Headline: 131.6% of global DRAM supply** | **~1-2 orders of magnitude too high** | Compounding the 2-4x KV overestimate with 10-60x batching overestimate yields 20-240x total inflation. Own sensitivity analysis (0.35-10.7% of DRAM supply) is more realistic. |

#### Why 384 KiB/token Is Not "Wrong"

The Excel's KV formula (`2 x 96 layers x 8 KV heads x 128 head_dim x 2 bytes = 384 KiB`) uses Grouped Query Attention (GQA). This is the dominant attention mechanism in production:

| Model | Attention | KV/token (FP16) | KV/token (FP8) | Market Share |
|---|---|---|---|---|
| Llama 3.1 405B | GQA | 504 KiB | 252 KiB | ~9% |
| Llama 3.1 70B | GQA | 320 KiB | 160 KiB | (included above) |
| GPT-4 (leaked) | MQA (1 head) | ~60 KiB | ~30 KiB | ~25% |
| Claude | Unknown (likely GQA) | Unknown | Unknown | ~32% |
| Gemini | MQA | Unknown | Unknown | ~20% |
| DeepSeek V3 | **MLA** | ~69 KiB | ~34 KiB | **~1-5%** |
| **Excel's 4.5T model** | GQA | **384 KiB** | **192 KiB** | N/A (hypothetical) |

The 384 KiB figure sits between Llama 70B (320 KiB) and 405B (504 KiB) in FP16. The issue is not the architecture -- it's the missing quantization (2-4x) and the missing batching model (10-60x).

### Web App: Moderate

| Issue | Impact | Explanation |
|---|---|---|
| **NAND/DRAM supply ~2x too low** | **Overstates shortfall by 2x** | 1,200 EB NAND vs ~2,800-3,000 EB actual; 155 EB DRAM vs ~310-350 EB actual. |
| **Consumer MAU 600M** | **~2-3x too low** | Below ChatGPT alone (~800M WAU). |
| **DAU/MAU 30%** | **~2x too low** | ChatGPT observed ~68%. |
| **Queries/day 8** | **~2.5x too low** | Observed ~20+ for ChatGPT users. |
| **Tokens/query 1.5K** | **~4-10x too high** | Median consumer interaction 100-400 tokens. |
| **Enterprise orgs 500K** | **~4-6x too low** | OpenAI alone has 1M+ business customers. |
| **Counterbalancing errors** | **Demand total closer than individual params suggest** | Low users x high tokens/query partially cancel out, but accidentally. |

---

## What Each Model Gets Right

### Excel Strengths

- **Tiered memory hierarchy concept** (HBM/DRAM/SSD/NAND) is architecturally sound and reflects real industry practice
- **MoE active parameter calculation** is correct: `dense_params + (expert_params x active_experts / total_experts)` -- math checks out for 4.5T spec
- **Peak QPS derivation funnel** (WAU -> DAU -> queries -> avg QPS -> peak QPS) is methodologically clean and internally consistent
- **Datalake storage** with embedding inflation (3.5-4.5x) is in reasonable range
- **The fundamental thesis** -- that AI at scale strains memory supply -- is correct and validated by industry data

### Web App Strengths

- **Three-segment user model** (Consumer/Enterprise/Agentic) is more realistic than two
- **KV bytes/token as empirical, declining parameter** is more robust than architecture-specific calculation that can be wrong
- **Persistent KV % differentiated by segment** is architecturally insightful (5% consumer / 40% enterprise / 80% agentic)
- **Memory tier split evolving over time** (HBM->DRAM->NAND shift) reflects real trends (NVIDIA ICMSP, Samsung KV offload SSDs)
- **Scenario-based approach** captures uncertainty better than point estimates
- **No single catastrophic architectural error** -- errors are moderate and partially offsetting

---

## What Each Model Is Missing

### Both Models Miss

| Gap | Impact | Notes |
|---|---|---|
| **Multi-modal inference** | Different memory footprint per token type | Image/video/audio tokens have substantially different KV characteristics than text |
| **Edge/on-device inference** | Shifts demand from server to client devices | By 2028-2031, significant consumer inference moves on-device |
| **CXL-attached memory** | Emerging 4th tier between DRAM and NAND | Expected to be major inference infrastructure component by 2028 |
| **Geographic distribution** | Different adoption curves by region | US, EU, China, India have very different dynamics |
| **Inference efficiency improvements** | 5-10x reduction per query over 5 years | Distillation, pruning, architecture improvements, hardware FLOPS/watt |

### Excel-Specific Gaps

| Gap | Impact |
|---|---|
| **No KV cache quantization (FP8/FP4)** | Misses 2-4x memory reduction that is standard in production on H100/Blackwell |
| **No continuous batching model** | Overestimates replica count by 10-60x at typical context lengths |
| **Single monolithic model assumption** | Real providers use model routing (small/medium/large) |
| **No MoE/speculative decoding** | Changes memory-per-token calculus dramatically |
| **No KV cache compression (PagedAttention, eviction, sliding window)** | Production systems reduce fragmentation from ~70% to <4% |

### Web App-Specific Gaps

| Gap | Impact |
|---|---|
| **sessionTurnover = 10 seems too low** | If sessions last 150-270s, theoretical turnover is 320-576/day; 10 implies >97% idle |
| **DRAM overhead 1.2x too simple** | Real overhead varies 1.3-2.0x (OS, networking, framework, page tables) |
| **No model weights in DRAM/NAND** | Inference clusters replicate weights across many nodes |
| **Missing multi-tenant efficiency** | Shared model weights, prefix caches, dynamic allocation |

---

## Net Assessment

| Dimension | Excel | Web App |
|---|---|---|
| **Demand estimate reliability** | Off by ~1-2 orders of magnitude (missing quantization x missing batching = 20-240x compounded) | Within ~1-2 orders of magnitude (counterbalancing errors) |
| **Supply estimate reliability** | Possibly off by unit confusion (EiB vs EB) | ~2x too low (fixable) |
| **Architectural sophistication** | More detailed (explicit model architecture), GQA is representative | Less detailed but more robust (model-agnostic) |
| **Usability for decisions** | Sensitivity tab is more realistic than headline numbers | Scenario toggle + Custom mode enables exploration |
| **Forward-looking value** | Locked to 2026 snapshot | Projects to 2031 with evolving parameters |
| **Biggest single error** | No batching model (~10-60x replica overcount) | Supply figures ~2x underestimate |

**Bottom line:** Both models have meaningful errors but in different places. The Excel's KV architecture (GQA, 384 KiB/token) is actually representative of ~95% of production inference -- the real problems are missing KV quantization (2-4x) and no batching model (10-60x), which compound to inflate demand by 20-240x. The web app avoids the batching problem by using a concurrent-sessions model but understates supply by ~2x. Both would benefit from a proper GPU-capacity layer that models the KV budget per replica to derive total infrastructure needs.

---

## Recommended Fixes

### For the Excel

| Fix | Priority | Details |
|---|---|---|
| Add KV cache quantization | **Critical** | GQA architecture is fine, but apply FP8 (2x reduction) or FP4 (4x). Change 384 KiB -> 192 KiB (FP8) or 96 KiB (FP4) per token |
| Add continuous batching model | **Critical** | Replace naive `replicas = peak_QPS / 1` with KV-budget-aware formula: `replicas = peak_concurrent / sessions_per_replica`. See [Batching Methodology](#batching-methodology) |
| Fix agentic user inconsistency | **High** | 15% of 1B = 150M, not 1.5B |
| Add model heterogeneity | **Medium** | Not all traffic goes to the largest model |
| Use the sensitivity analysis range as headline | **Quick win** | The 0.35-10.7% of DRAM range is far more realistic |

### For the Web App

| Fix | Priority | Details |
|---|---|---|
| Double NAND supply to ~2,800 EB | **Critical** | Current 1,200 EB is ~2.5x below actual 2026 production |
| Double DRAM supply to ~310 EB | **Critical** | Current 155 EB is ~2x below actual 2026 production |
| Bump consumer MAU to ~1.5B+ | **High** | ChatGPT alone exceeds current 600M figure |
| Lower tokens/query to ~500 | **High** | Median consumer interaction is 100-400 tokens |
| Raise queries/day to ~15-20 | **High** | Observed ~20+ for engaged ChatGPT users |
| Bump enterprise orgs to ~2M+ | **Medium** | OpenAI alone has 1M+ business customers |
| Raise DAU/MAU to ~50% | **Medium** | 30% is below observed engagement for AI chatbots |
| Add CXL memory tier | **Low** | Emerging tier between DRAM and NAND for 2028+ |
| Add edge/on-device category | **Low** | Relevant for 2028-2031 consumer projections |

---

## Memory Supply Reality Check

The directional thesis -- that AI inference is creating serious memory supply pressure -- is **validated by industry data**, even though both models get the magnitude wrong.

| Metric | Value | Source |
|---|---|---|
| Data centers' share of high-end DRAM in 2026 | **70%** | TrendForce / Tom's Hardware |
| AI's share of DRAM wafer capacity in 2026 | **20%** | TrendForce |
| OpenAI Stargate DRAM demand | Up to **40% of global DRAM output** | Tom's Hardware |
| HBM sold out through 2026 | Confirmed by Micron, SK Hynix | NotebookCheck |
| Micron can meet customer demand | Only **55-60%** | TrendForce |
| HBM consumes vs DDR5 per GB | **3x the wafer capacity** | Tom's Hardware |
| NAND bits going to AI by 2026 | **20%**, contributing 34% of market value | Tom's Hardware |
| Relief expected | **Not before 2027-2028** | TrendForce / Micron |
| HBM market growth | $35B (2025) -> $54.6B (2026) -> $100B (2028) | SK Hynix / Micron |
| KV cache offloading to SSD | **Production-mature** (LMCache + NVIDIA Dynamo) | LMCache / NVIDIA |

The question isn't "if" there's a memory crunch -- it's "how big." The web app, with corrected supply figures and demand recalibration, would be a much more useful tool for answering that question than the Excel in its current state.

---

## Frontier Model KV Cache Reference

The KV cache formula for any transformer is: `bytes/token = 2 x layers x kv_heads x head_dim x precision_bytes`

| Model | Layers | KV Heads | Head Dim | Attention | FP16 (bytes/tok) | FP8 (bytes/tok) |
|---|---|---|---|---|---|---|
| **Llama 3.1 405B** | 126 | 8 | 128 | GQA | 516,096 (504 KiB) | 258,048 (252 KiB) |
| **Llama 3.1 70B** | 80 | 8 | 128 | GQA | 327,680 (320 KiB) | 163,840 (160 KiB) |
| **GPT-4 (leaked, unconfirmed)** | 120 | 1 | ~128 | MQA | ~61,440 (60 KiB) | ~30,720 (30 KiB) |
| **Excel's 4.5T model** | 96 | 8 | 128 | GQA | 393,216 (384 KiB) | 196,608 (192 KiB) |
| **DeepSeek V3** | 61 | N/A | N/A | MLA | ~70,272 (69 KiB) | ~35,136 (34 KiB) |
| **Mixtral 8x7B** | 32 | 8 | 128 | GQA | 131,072 (128 KiB) | 65,536 (64 KiB) |

**Key observations:**
- The Excel's 384 KiB/token is correct arithmetic and sits between real frontier GQA models (320-504 KiB in FP16)
- With FP8 quantization (production standard on H100), all GQA models drop by 2x
- MLA (DeepSeek only, ~1-5% of traffic) is ~5-11x more efficient than GQA, but is not yet widely adopted
- If GPT-4's leaked MQA architecture is accurate, it achieves similar efficiency to MLA through a different mechanism (1 KV head vs latent compression)

---

## Batching Methodology

### The Problem with the Excel's Approach

The Excel derives replica counts from `replicas = peak_QPS / throughput_per_replica`, where implied throughput is ~0.23-0.79 QPS/replica. This assumes each replica handles **one request at a time**.

In reality, production LLM inference uses **continuous batching**: multiple requests are processed simultaneously on the same GPUs. During the decode phase, the GPU processes one forward pass per step but generates one token for *each* active sequence in the batch. The batch size is limited by **KV cache memory**, not compute.

### The Correct Formula Chain

```
Step 1: Peak Concurrent Sessions
──────────────────────────────────────
peak_concurrent = peak_QPS x avg_latency_seconds

  Example: 100K peak QPS x 5s avg latency = 500K concurrent sessions


Step 2: GPU Configuration per Replica
──────────────────────────────────────
GPUs_per_replica = enough to hold model weights + KV budget

  Example (Llama 405B FP8): 405 GB weights → 8x H100 (640 GB total)


Step 3: KV Cache Budget per Replica
──────────────────────────────────────
KV_budget = (GPUs x HBM_per_GPU) - model_weights - activations - overhead

  Example: 640 - 405 - 50 - 40 = ~145 GB available for KV cache


Step 4: Concurrent Sessions per Replica
──────────────────────────────────────
KV_per_sequence = KV_bytes_per_token x avg_context_length

sessions_per_replica = floor(KV_budget / KV_per_sequence)

  Example (FP8, 4K ctx): 258,048 bytes/tok x 4,096 = ~1.0 GB/seq
  sessions_per_replica = 145 / 1.0 = ~145 concurrent sequences


Step 5: Number of Replicas
──────────────────────────────────────
num_replicas = ceil(peak_concurrent / sessions_per_replica)

  Example: ceil(500K / 145) = ~3,449 replicas


Step 6: Total HBM
──────────────────────────────────────
total_HBM = num_replicas x GPUs_per_replica x HBM_per_GPU

  Example: 3,449 x 8 x 80 GB = ~2.2 PB of HBM
```

### How Context Length Changes Everything

For a Llama 3.1 405B on 8x H100 (FP8 weights, FP8 KV, ~145 GB KV budget):

| Avg Context | KV/Sequence | Max Concurrent/Replica | QPS (at 5s latency) |
|---|---|---|---|
| 2K tokens | 0.5 GB | ~290 | ~58 |
| 4K tokens | 1.0 GB | ~145 | ~29 |
| 8K tokens | 2.0 GB | ~72 | ~14 |
| 32K tokens | 8.0 GB | ~18 | ~3.6 |
| 128K tokens | 32 GB | ~4 | ~0.8 |

At typical consumer context (2-4K), one replica serves 145-290 concurrent sequences. At agentic context (32-128K), it drops to 4-18. This is why context length is the critical variable for infrastructure sizing -- and why the Excel's ~0.5 QPS/replica is only realistic for 128K+ context workloads.

### Overflow to DRAM and SSD Tiers

When KV cache exceeds HBM capacity, modern systems offload to lower tiers:

| Tier | Bandwidth | Latency | Role |
|---|---|---|---|
| **HBM** (GPU) | 2,000-3,350 GB/s | Nanoseconds | Active KV for current token generation |
| **DRAM** (Host CPU) | 25-50 GB/s | ~100 ns + PCIe transfer | Recently evicted KV, prefix caches |
| **NVMe SSD** | 7-13 GB/s | ~10-100 us | Cold KV blocks, long-idle sessions |
| **Network flash (ICMSP)** | Low GB/s | Low milliseconds | Pod-level shared KV cache (NVIDIA BlueField-4) |

Production systems (vLLM + LMCache, NVIDIA Dynamo) manage this hierarchy automatically. The DRAM tier is the most important for memory demand modeling because it absorbs the overflow that doesn't fit in HBM -- this is where the DRAM supply pressure comes from.

### Production Throughput Benchmarks

| Model | Hardware | Throughput | Source |
|---|---|---|---|
| DeepSeek-R1 671B | 8x H200 | 2,200 tok/s per H200 | vLLM Dec 2025 |
| Llama 3.1 405B | 8x MI300X | ~5.76 req/s (ShareGPT) | vLLM AMD |
| Mixtral 8x7B | 2x H100 SXM (FP8) | 38.4 req/s streaming | NVIDIA TensorRT-LLM |
| Llama 405B | GCP Trillium TPU | 1,703 tok/s | Google Cloud |
| DeepSeek-R1 | Blackwell Ultra | 5,842 tok/s/GPU (offline) | NVIDIA MLPerf |

---

## Sources

### User & Usage Data
- [ChatGPT 800M WAU - Backlinko](https://backlinko.com/chatgpt-stats)
- [ChatGPT Statistics - DemandSage](https://www.demandsage.com/chatgpt-statistics/)
- [ChatGPT 2.5B Prompts/Day - TechCrunch](https://techcrunch.com/2025/07/21/chatgpt-users-send-2-5-billion-prompts-a-day/)
- [Google Gemini 750M MAU - TechCrunch](https://techcrunch.com/2026/02/04/googles-gemini-app-has-surpassed-750m-monthly-active-users/)
- [OpenAI 1M Business Customers](https://openai.com/index/1-million-businesses-putting-ai-to-work/)
- [Gartner 80% Enterprise GenAI by 2026](https://www.gartner.com/en/newsroom/press-releases/2023-10-11-gartner-says-more-than-80-percent-of-enterprises-will-have-used-generative-ai-apis-or-deployed-generative-ai-enabled-applications-by-2026)

### Token Consumption
- [OpenRouter State of AI 2025: 100T Token Study](https://openrouter.ai/state-of-ai)
- [a16z / OpenRouter State of AI](https://a16z.com/state-of-ai/)
- [Epoch AI: ChatGPT Energy Use](https://epoch.ai/gradient-updates/how-much-energy-does-chatgpt-use)
- [SWE-bench Token Consumption - OpenReview](https://openreview.net/forum?id=1bUeVB3fov)
- [Claude Code Pricing & Usage](https://code.claude.com/docs/en/costs)
- [Anthropic Agentic Coding Trends 2026](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)

### KV Cache Architecture & Frontier Model Specs
- [DeepSeek-V3 Technical Report (arXiv)](https://arxiv.org/html/2412.19437v1)
- [DeepSeek-V3 MLA Explained - McCormick ML](https://mccormickml.com/2025/02/12/the-inner-workings-of-deep-seek-v3/)
- [DeepSeek-V3 MLA Analysis - Lior Sinai](https://liorsinai.github.io/machine-learning/2025/02/22/mla.html)
- [MLA Explanation - HuggingFace](https://huggingface.co/blog/NormalUhr/mla-explanation)
- [Llama 3.1 405B Config - HuggingFace](https://huggingface.co/meta-llama/Llama-3.1-405B)
- [GPT-4 Architecture Leak - KDnuggets](https://www.kdnuggets.com/2023/07/gpt4-details-leaked.html)
- [TransMLA: Converting GQA to MLA (NeurIPS 2025)](https://arxiv.org/abs/2502.07864)
- [NVIDIA NVFP4 KV Cache](https://developer.nvidia.com/blog/optimizing-inference-for-long-context-and-large-batch-sizes-with-nvfp4-kv-cache/)
- [NVIDIA Dynamo KV Cache](https://developer.nvidia.com/blog/how-to-reduce-kv-cache-bottlenecks-with-nvidia-dynamo/)
- [Samsung KV Cache Offloading](https://download.semiconductor.samsung.com/resources/white-paper/scaling_ai_inference_with_kv_cache_offloading.pdf)
- [NVIDIA BlueField-4 ICMSP](https://developer.nvidia.com/blog/introducing-nvidia-bluefield-4-powered-inference-context-memory-storage-platform-for-the-next-frontier-of-ai/)
- [KV Cache Optimization Techniques - Omri Mallis](https://www.omrimallis.com/posts/techniques-for-kv-cache-optimization/)
- [HuggingFace KV Cache Quantization](https://huggingface.co/blog/kv-cache-quantization)

### Batching & Inference Throughput
- [Continuous Batching from First Principles - HuggingFace](https://huggingface.co/blog/continuous_batching)
- [Orca: Continuous Batching (OSDI 2022)](https://www.usenix.org/conference/osdi22/presentation/yu)
- [Inside vLLM: Anatomy of High-Throughput Inference](https://blog.vllm.ai/2025/09/05/anatomy-of-vllm.html)
- [vLLM Large-Scale Serving: DeepSeek R1](https://blog.vllm.ai/2025/12/17/large-scale-serving.html)
- [NVIDIA Mixtral TensorRT-LLM Benchmarks](https://developer.nvidia.com/blog/achieving-high-mixtral-8x7b-performance-with-nvidia-h100-tensor-core-gpus-and-tensorrt-llm/)
- [NVIDIA Blackwell DeepSeek-R1 Record](https://developer.nvidia.com/blog/nvidia-blackwell-delivers-world-record-deepseek-r1-inference-performance/)
- [VMware LLM Inference Sizing Guide](https://blogs.vmware.com/cloud-foundation/2024/09/25/llm-inference-sizing-and-performance-guidance/)
- [NVIDIA Mastering LLM Inference Optimization](https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/)
- [FlexGen: High-Throughput Generative Inference](https://arxiv.org/pdf/2303.06865)

### Market Share
- [Menlo Ventures 2025 Mid-Year LLM Market Update](https://menlovc.com/perspective/2025-mid-year-llm-market-update/)

### Memory Supply
- [TrendForce: AI Consumes 20% DRAM Wafer Capacity 2026](https://www.trendforce.com/news/2025/12/26/news-ai-reportedly-to-consume-20-of-global-dram-wafer-capacity-in-2026-hbm-gddr7-lead-demand/)
- [TrendForce: Memory CapEx 2026](https://www.trendforce.com/presscenter/news/20251113-12780.html)
- [Tom's Hardware: Data Centers 70% of Memory 2026](https://www.tomshardware.com/pc-components/ram/data-centers-will-consume-70-percent-of-memory-chips-made-in-2026-supply-shortfall-will-cause-the-chip-shortage-to-spread-to-other-segments)
- [Tom's Hardware: OpenAI Stargate 40% of DRAM](https://www.tomshardware.com/pc-components/dram/openais-stargate-project-to-consume-up-to-40-percent-of-global-dram-output-inks-deal-with-samsung-and-sk-hynix-to-the-tune-of-up-to-900-000-wafers-per-month)
- [HBM Supply Curve - Next Platform](https://www.nextplatform.com/2025/12/19/hbm-supply-curve-gets-steeper-but-still-cant-meet-demand/)
- [SK Hynix 2026 Market Outlook](https://news.skhynix.com/2026-market-outlook-focus-on-the-hbm-led-memory-supercycle/)
- [Micron: Memory Crunch Won't Ease Soon](https://www.trendforce.com/news/2025/12/18/news-micron-reveals-three-culprits-behind-memory-crunch-and-why-it-wont-ease-soon/)
- [IDC Global Memory Shortage Crisis 2026](https://www.idc.com/resource-center/blog/global-memory-shortage-crisis-market-analysis-and-the-potential-impact-on-the-smartphone-and-pc-markets-in-2026/)

### Agentic AI
- [Salesforce Agentforce 12K Customers](https://www.salesforce.com/news/press-releases/2025/10/13/agentic-enterprise-announcement/)
- [Gartner: 40% Enterprise Apps with Agents by 2026](https://www.gartner.com/en/newsroom/press-releases/2025-08-26-gartner-predicts-40-percent-of-enterprise-apps-will-feature-task-specific-ai-agents-by-2026-up-from-less-than-5-percent-in-2025)
- [Microsoft: 1.3B AI Agents by 2028](https://www.demandsage.com/ai-agents-market-size/)
- [Menlo Ventures: Enterprise LLM Spend](https://menlovc.com/perspective/2025-mid-year-llm-market-update/)

### RAG Storage
- [Pure Storage: Vector DB Storage](https://blog.purestorage.com/purely-technical/vector-database-and-storage/)
- [Microsoft Azure: RAG Embeddings Architecture](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/rag/rag-generate-embeddings)
