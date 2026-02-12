# ADR 001: Hybrid AI Strategy (Edge vs. Cloud)

| Status | Proposed |
| :--- | :--- |
| **Date** | 2026-02-12 |
| **Deciders** | Engineering Team, AI Architects |
| **Tags** | #AI #Architecture #EdgeComputing #Cloud #Strategy |


---

## TL;DR (Executive Summary)

<table>
<tr>
<td width="25%"><b> Decision</b></td>
<td>Adopt <b>Hybrid AI Strategy</b> combining Edge (SLMs) and Cloud (LLMs)</td>
</tr>
<tr>
<td><b> Expected Impact</b></td>
<td><b>40-60% cost reduction</b> ($30K/month savings at 1M users)</td>
</tr>
<tr>
<td><b> Performance Goal</b></td>
<td><b>70% requests on Edge</b> with <100ms latency</td>
</tr>
<tr>
<td><b> Key Benefit</b></td>
<td><b>Enhanced privacy</b> + <b>Offline capability</b> for basic functions</td>
</tr>
<tr>
<td><b> Timeline</b></td>
<td>Q1-Q4 2026 (POC → GA in 10 months)</td>
</tr>
</table>


---

##  1. Context (Background and Problem)
Currently, relying solely on **Cloud-based LLM** (such as GPT-4, Claude) creates limitations that affect system scalability in 3 main areas:

1.  **Cost:** API costs increase exponentially as the number of users (Scale) increases, making it difficult to control the budget
2.  **Latency:** Tasks requiring immediate response, such as *Auto-complete* or *Real-time Filter*, are limited by network speed
3.  **Privacy:** Highly sensitive data should not be sent to external servers for maximum security

---

##  1.1 Alternatives Considered

We considered 3 main approaches before making the decision:

| Approach | Pros | Cons | Decision |
| :--- | :--- | :--- | :--- |
| **Pure Cloud** | • Unlimited performance<br>• Simple architecture<br>• Easy updates | • High cost ($50K/month at 1M users)<br>• No offline mode<br>• Privacy risk |  **Rejected** |
| **Pure Edge** | • $0 API cost<br>• Maximum privacy<br>• 100% offline | • Limited reasoning<br>• Requires 4GB+ RAM<br>• No support for complex tasks |  **Rejected** |
| **Hybrid**  | • 40-60% cost reduction<br>• Supports all task types<br>• Balance between Cost/Privacy/Performance | • Complex architecture<br>• Must maintain 2 models |  **Selected** |

###  Cost Projection

| User Scale | Pure Cloud | Hybrid Strategy | Savings |
| :--- | :--- | :--- | :--- |
| 100K users | $5,000/mo | $3,000/mo | **40%** |
| 500K users | $25,000/mo | $12,500/mo | **50%** |
| 1M users | $50,000/mo | $20,000/mo | **60%** |
| 5M users | $250,000/mo | $85,000/mo | **66%** |

**Assumption:** Edge handles 70% of requests, remaining 30% goes to Cloud

---


##  2. Decision

We have decided to implement the **Hybrid AI Strategy**, distributing processing load by task type (Task Orchestration) as follows:

###  Task Allocation

* **Edge (On-device):** Process on user's device using **Small Language Models (SLMs)** for simple tasks requiring high speed
* **Cloud (Remote):** Process on Cloud for tasks requiring deep reasoning (High-reasoning) or large amounts of data

---

##  3. Rationale

Why did we choose the Hybrid approach instead of either option alone?

* **Scalability & Cost:** Offloading simple tasks to Edge helps tremendously reduce the number of Tokens that need to be paid to Cloud Provider
* **User Experience (UX):** Edge processing provides **Instant UX** experience (no internet latency)
* **Security Design:** We can use Edge as a "front gate" to filter or manage confidential data before considering forwarding to Cloud

---

##  4. Comparison & Workflow

###  Workflow

| Task Type | Processing Location | Model Examples | Expected Latency | Use Case Examples |
| :--- | :--- | :--- | :--- | :--- |
| **Simple & Sensitive** |  Edge (Local) | Phi-3 Mini (1.8B)<br>Llama-3.2-3B | **< 100ms** (p95) | Auto-complete, Spell-check, PII redaction, Quick FAQ |
| **Complex & Reasoning** |  Cloud LLM | GPT-4 Turbo<br>Claude 3.5 Sonnet | **< 500ms** (p95) | Document analysis, Strategy planning, Code generation |

###  Task Routing Decision Matrix

The Orchestrator system will make decisions based on these criteria:

| Factor | Edge Condition | Cloud Condition | Weight |
| :--- | :--- | :--- | :--- |
| **Token Count** | < 500 tokens | ≥ 500 tokens | 30% |
| **Complexity Score** | < 0.5 (simple) | ≥ 0.5 (complex) | 40% |
| **Contains PII/Sensitive Data** | Yes | No | 20% |
| **Network Status** | Offline/Slow | Fast connection | 10% |

**Decision Logic:**
```python
if contains_pii == True:
    route = "Edge"  # Privacy first
elif token_count < 500 and complexity < 0.5:
    route = "Edge"  # Simple task
elif network_offline == True:
    route = "Edge"  # Fallback
else:
    route = "Cloud"  # Complex/large task
```

###  Expected Impact (Consequences)

####  Pros

* **Cost Reduction:** Reduce API costs compared to using 100% Cloud
* **Offline Capability:** Basic functions still work even without internet
* **Low Latency:** Very fast response for Real-time tasks

####  Cons/Challenges

* **Complexity:** Must develop **Orchestrator** system to decide which tasks go where
* **Device Constraint:** Speed depends on user's Hardware (RAM/CPU/NPU)
* **Maintenance:** Must maintain and update both models simultaneously

---

##  4.1 Success Metrics & KPIs

We will measure the success of the Hybrid Strategy through these KPIs:

###  Primary Metrics

| Metric | Baseline (Pure Cloud) | Target (Hybrid) | Status |
| :--- | :--- | :--- | :--- |
| **Edge Utilization Rate** | 0% | **70%** of requests |  Target |
| **Cost per 1M Requests** | $50 | **$20** (60% reduction) |  Target |
| **Average Latency (p95)** | 800ms | **< 300ms** |  Target |
| **Offline Capability** | 0% | **100%** basic functions |  Target |

###  Secondary Metrics

| Metric | Target | Measurement Method |
| :--- | :--- | :--- |
| **Model Accuracy Parity** | Edge ≥ 90% of Cloud quality | A/B testing, User feedback |
| **User Satisfaction (NPS)** | ≥ 4.5/5.0 | In-app surveys |
| **API Error Rate** | < 0.1% | Monitoring logs |
| **Model Update Frequency** | Edge: Monthly, Cloud: Weekly | Release cadence |

###  Monitoring Dashboard (Real-time tracking)

```
┌─────────────────────────────────────┐
│ Edge Requests:  ████████░░ 70%      │
│ Cloud Requests: ███░░░░░░░ 30%      │
│ Avg Latency:    180ms               │
│ Cost Savings:   $30K/month          │
│ Error Rate:     0.05%               │
└─────────────────────────────────────┘
```

---


##  5. Implementation Details

###  Orchestrator Logic

The Orchestrator system works in 3 steps:

#### Step 1: Pre-processing Analysis

```python
def analyze_request(user_input):
    return {
        'token_count': count_tokens(user_input),
        'complexity_score': calculate_complexity(user_input),
        'contains_pii': detect_sensitive_data(user_input),
        'network_speed': check_network_latency()
    }
```

#### Step 2: Routing Decision

```python
def route_request(analysis):
    # Priority 1: Privacy (if PII → always Edge)
    if analysis['contains_pii']:
        return "Edge"
    
    # Priority 2: Complexity + Size
    if analysis['token_count'] < 500 and analysis['complexity_score'] < 0.5:
        return "Edge"
    
    # Priority 3: Network availability
    if analysis['network_speed'] == "offline" or analysis['network_speed'] == "slow":
        return "Edge"
    
    # Default: Cloud for complex tasks
    return "Cloud"
```

#### Step 3: Fallback Strategy

- If Cloud unavailable → Auto fallback to Edge (degraded mode)
- If Edge insufficient memory → Queue to Cloud
- Response timeout: Edge 2s, Cloud 10s

---

###  Hardware Requirements

| Platform | Minimum Specs | Recommended | Model Size |
| :--- | :--- | :--- | :--- |
| ** Mobile** | 4GB RAM, 2GB storage | 6GB RAM, 3GB storage | Phi-3 Mini (1.8B) |
| ** Desktop** | 8GB RAM, 3GB storage | 16GB RAM, 5GB storage | Llama-3.2-3B |
| ** Browser** | N/A (Cloud only) | N/A | N/A |

**Supported OS:**
- iOS 15+ / iPadOS 15+
- Android 12+ (ARM64)
- Windows 10+, macOS 12+, Linux (x86-64)

---

###  Implementation Roadmap

| Phase | Timeline | Deliverables | Success Criteria |
| :--- | :--- | :--- | :--- |
| **Phase 1: POC** | Q1 2026 (Feb-Mar) | • Phi-3 on Edge<br>• Basic Orchestrator<br>• 100 beta users | Edge handles 50%+ requests |
| **Phase 2: Development** | Q2 2026 (Apr-Jun) | • Production Orchestrator<br>• Monitoring dashboard<br>• 10K users | Edge handles 65%+ requests |
| **Phase 3: Beta Launch** | Q3 2026 (Jul-Sep) | • Full rollout to 50% users<br>• A/B testing<br>• Performance tuning | Cost reduction 40%+ |
| **Phase 4: GA** | Q4 2026 (Oct-Dec) | • 100% users<br>• Auto-scaling<br>• Multi-region support | All KPIs met |

---

###  Risk Assessment

| Risk | Probability | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Device fragmentation** | High | Medium | • Support top 3 models per platform<br>• Fallback to Cloud for unsupported devices |
| **Model drift over time** | Medium | High | • Monthly accuracy testing<br>• Auto-sync Edge models from Cloud baseline |
| **Privacy breach** | Low | Critical | • On-device encryption (AES-256)<br>• No telemetry for PII data<br>• Regular security audits |
| **Edge model outdated** | Medium | Medium | • OTA updates every 2 weeks<br>• Version control with rollback |
| **Cost overrun** | Low | High | • Real-time cost monitoring<br>• Auto-throttling at budget threshold |

---

##  6. Architecture Overview

###  System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         User Request                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                      Orchestrator                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Complexity   │  │ PII Detector │  │ Network      │       │
│  │ Analyzer     │  │              │  │ Monitor      │       │
│  └──────────────┘  └──────────────┘  └──────────────┘       │
└──────────────┬─────────────────────────────┬────────────────┘
               │                             │
       ┌───────▼────────┐           ┌────────▼────────┐
       │     Edge       │           │     Cloud       │
       │  (On-device)   │           │  (Remote LLM)   │
       ├────────────────┤           ├─────────────────┤
       │ Phi-3 Mini     │           │ GPT-4 Turbo     │
       │ Llama-3.2-3B   │           │ Claude 3.5      │
       │                │           │                 │
       │ • < 100ms      │           │ • < 500ms       │
       │ • Offline OK   │           │ • Online only   │
       │ • 100% Private │           │ • High accuracy │
       └────────┬───────┘           └────────┬────────┘
                │                            │
                └────────────┬───────────────┘
                             ▼
                    ┌──────────────────┐
                    │  Response to     │
                    │  User            │
                    └──────────────────┘
```

###  Request Flow Example

**Example 1: Simple Query (→ Edge)**
```
User: "Fix grammar: I goes to school"
→ Orchestrator: tokens=8, complexity=0.2, pii=False
→ Route: Edge (Phi-3)
→ Response: "I go to school" (Latency: 45ms)
```

**Example 2: Complex Query (→ Cloud)**
```
User: "Analyze SWOT Analysis of Company X from 50-page report"
→ Orchestrator: tokens=8500, complexity=0.9, pii=False
→ Route: Cloud (GPT-4)
→ Response: [Detailed analysis] (Latency: 3200ms)
```

**Example 3: Sensitive Data (→ Edge)**
```
User: "Summarize document containing ID number 1-2345-67890-12-3"
→ Orchestrator: tokens=120, complexity=0.4, pii=True 
→ Route: Edge (Force, privacy protection)
→ Response: [Summary with PII redacted] (Latency: 280ms)
```

---

##  7. References & Resources

###  Technical Documentation

1. [Phi-3 Technical Report](https://arxiv.org/abs/2404.14219) - Microsoft Research, 2024
2. [On-Device AI: The Future of Edge Computing](https://www.anthropic.com/research/edge-ai) - Anthropic, 2025
3. [Cost Optimization Strategies for LLM Deployment](https://openai.com/research/cost-optimization) - OpenAI, 2025

###  Related ADRs

- ADR-002: Model Selection Criteria (Pending)
- ADR-003: Privacy & Security Framework (Planned)

###  Stakeholders & Decision Makers

| Role | Name | Responsibility |
| :--- | :--- | :--- |
| **Tech Lead** | [Name] | Overall architecture approval |
| **AI Architect** | [Name] | Model selection & performance |
| **Security Lead** | [Name] | Privacy & compliance review |
| **Product Manager** | [Name] | Business case & ROI validation |

---

##  Document Control

| Version | Date | Author | Changes |
| :--- | :--- | :--- | :--- |
| 1.0 | 2026-02-12 | AI Team | Initial proposal with full analysis |
| 0.9 | 2026-02-05 | AI Team | Draft version |

###  Approval Status

- [ ] **Tech Lead:** ________________ (Date: ______)
- [ ] **Security Team:** ________________ (Date: ______)
- [ ] **Product Manager:** ________________ (Date: ______)
- [ ] **Finance Team:** ________________ (Date: ______)

---

