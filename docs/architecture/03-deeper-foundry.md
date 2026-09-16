# C. Optional Deeper Integration

**Message:** After B's selected Guardrails + Evaluations, consume more Foundry capabilities only where agent hosting, repeated tool connections, long-term memory, trace investigation or adversarial testing becomes an operational bottleneck. Each is an independent adoption decision. C is not a target state and is not automatically better than B.

**Read left to right:** Independently operated responsibility -> optional Foundry capability and potential simplification -> ownership that stays outside Foundry. The left column shows work considered for consolidation, not a mandatory duplicate stack running alongside Foundry. Adopt only the rows whose coverage, cost and constraints justify the change; unselected responsibilities stay where they are in B.

```mermaid
flowchart LR
    subgraph EXT["BEFORE | THIRD-PARTY / CUSTOM"]
        Hosting["Agent hosting + scaling<br/>Runtime lifecycle operations"]
        Connections["Repeated tool adapters<br/>Per-consumer connection setup"]
        Store["Long-term memory store<br/>Extraction + retrieval pipeline"]
        Views["Custom AI trace views<br/>Span exploration + correlation"]
        Attacks["Custom / external adversarial tests<br/>Attack generation + run reports"]
    end
    subgraph FOUNDRY["MICROSOFT FOUNDRY | CHOOSE INDEPENDENTLY"]
        Baseline["ALREADY IN B<br/>Models + Guardrails + Evaluations"]
        Runtime["1. Agent Service<br/>Managed agent hosting + lifecycle"]
        Toolbox["2. Toolbox<br/>Reusable supported tool connections"]
        Memory["3. Memory: preview<br/>Managed extraction + recall"]
        Trace["4. Foundry tracing<br/>Ready-made AI trace exploration"]
        Red["5. AI Red Teaming: preview<br/>Automated attacks + reporting"]
    end
    subgraph APP["APPLICATION / CUSTOMER-OWNED | RETAINED"]
        App["UI + business logic + agent code<br/>Approvals + failure recovery"]
        Data["PII + domain controls + consent<br/>Memory scope + retention<br/>Conversation-history requirements"]
        Review["Test scope + human security testing<br/>Remediation + release decisions"]
    end
    subgraph ENT["ENTERPRISE SYSTEMS / DATA | RETAINED"]
        Tools["Tools + APIs + source data<br/>Enterprise authorization"]
    end
    subgraph AZURE["SEPARATE AZURE RESOURCE / CUSTOMER OPERATIONS"]
        Monitor["Application Insights + Azure Monitor<br/>Instrumentation + data controls<br/>Existing observability can stay"]
    end
    Hosting -->|"supported runtime operations"| Runtime
    Connections -->|"supported shared connections"| Toolbox
    Store -->|"scoped memory pipeline"| Memory
    Views -->|"AI investigation experience"| Trace
    Attacks -->|"supported automated test scope"| Red
    Runtime -.->|"customer still owns"| App
    Toolbox -.->|"customer still owns"| Tools
    Memory -.->|"customer still owns"| Data
    Trace -.->|"still requires"| Monitor
    Red -.->|"customer still owns"| Review
    classDef app fill:#EAF2FC,stroke:#245A91,color:#172B4D,stroke-width:2px
    classDef external fill:#F3F4F6,stroke:#667085,color:#202939
    classDef foundry fill:#DDF3EF,stroke:#087E8B,color:#123B40,stroke-width:2px
    classDef optional fill:#FFFFFF,stroke:#087E8B,color:#123B40,stroke-width:2px,stroke-dasharray:5 4
    classDef enterprise fill:#FFF2D8,stroke:#96711C,color:#493A16
    class App,Data,Review app
    class Hosting,Connections,Store,Views,Attacks,Monitor external
    class Baseline foundry
    class Runtime,Toolbox,Memory,Trace,Red optional
    class Tools enterprise
    style APP fill:#FFFFFF,stroke:#245A91,stroke-width:2px
    style EXT fill:#FFFFFF,stroke:#667085,stroke-dasharray:5 4
    style FOUNDRY fill:#F3FBF9,stroke:#087E8B,stroke-width:2px
    style ENT fill:#FFFFFF,stroke:#96711C,stroke-width:2px
    style AZURE fill:#FFFFFF,stroke:#667085,stroke-dasharray:5 4
```

**Legend:** This is a consolidation map, not a runtime sequence or network topology. Solid arrows identify work potentially transferred to a supported Foundry capability; dotted arrows identify retained ownership or dependencies, not API calls or telemetry direction. Dashed numbered boxes are independent options; the filled box is B's existing adoption. Customer-authored agent code remains customer-owned even when executed in Agent Service. See [availability and integration caveats](../architecture-validation.md); this is not a tested deployment template.

**What does not move:** The whole application, business process, enterprise APIs and authoritative data do not move into Foundry. Authorization, approvals/consent, PII/domain controls, data governance and human decisions remain customer responsibilities. Application Insights remains a separate resource; Foundry provides the AI tracing experience over collected telemetry, not a replacement telemetry backend.

## ASCII Fallback

```text
ALREADY IN B: Foundry models + Guardrails + offline Evaluations

BEFORE / INDEPENDENTLY OPERATED   OPTIONAL FOUNDRY CHOICE    STILL OWNED OUTSIDE
Agent hosting + lifecycle -----> 1. Agent Service ........ Business logic/code,
                                                                 managed runtime           approvals + recovery
Repeated tool connections -----> 2. Toolbox .............. APIs + authorization
                                                                 shared connections        [ENTERPRISE SYSTEMS]
Long-term memory pipeline -----> 3. Memory (preview) ..... Consent + scope,
                                                                 extraction + recall       retention + PII
Custom AI trace exploration ---> 4. Foundry tracing ...... Instrumentation,
                                                                 AI trace experience       Application Insights
                                                                                                                     [SEPARATE RESOURCE]
Adversarial test automation ----> 5. AI Red Teaming ....... Human security tests,
                                                                 (preview)                 remediation + gates

Arrows show potential consolidation, NOT runtime calls or a required bundle.
Unselected responsibilities stay in B. Authoritative enterprise data and
conversation-history requirements remain distinct from long-term memory.
```

## Potential Consolidation

| Capability | Before / independently operated | With deeper Foundry adoption | What becomes simpler | Customer still owns |
|---|---|---|---|---|
| **1. Agent Service** | Operate agent compute/hosting, scaling and runtime lifecycle infrastructure around the chosen framework | Use managed prompt-agent execution or hosted code-based agents where the execution model fits | Reduce operation of a separate agent-serving platform for the supported runtime scope; no need to move the UI or entire application | Business logic, agent instructions/code, workflow semantics, dependency maintenance, compatibility tests, deployment/release choices, approvals and failure recovery |
| **2. Toolbox** | Repeat tool definitions, adapters and supported authentication/connection setup across agent consumers | Configure shared tools and supported connections exposed through an MCP-compatible endpoint | Reuse supported tool integrations instead of maintaining each consumer's connection plumbing independently; external runtimes can use it too | Tool implementations, API contracts, identity/connection configuration, least privilege, backend authorization, consent, network policy and unsupported adapters |
| **3. Memory (preview)** | Build and operate a long-term memory store plus extraction/update/retrieval pipeline | Use managed memory extraction and scoped recall where supported, including direct Memory Store APIs | Avoid maintaining that custom memory pipeline/store for the covered use case; not a replacement for all data infrastructure | Chat/embedding dependencies and cost, memory scope/isolation, consent, retention/deletion, PII controls and recall correctness; authoritative data and conversation-history requirements remain separate |
| **4. Foundry tracing + Application Insights** | Build custom AI trace exploration views and investigation workflows over instrumented spans | Use the Foundry AI trace experience backed by connected Application Insights | Reduce custom AI-specific trace-view development and investigation work for supported spans; this does not eliminate telemetry collection or every existing dashboard | Instrumentation/custom spans, context propagation, Application Insights configuration, redaction, access/retention, costs, alerts, existing observability and end-to-end operations |
| **5. AI Red Teaming (preview)** | Maintain automated adversarial prompt generation, attack orchestration, scoring and report aggregation, or integrate an external workflow | Use supported Foundry automated attacks and test reporting through a compatible workflow | Reduce custom adversarial test-harness and report plumbing for covered targets/strategies; testing remains separate from runtime Guardrails and B's evaluations | Target adapter/access, test authorization and scope, datasets, compatibility checks, human security testing, findings triage, remediation and release gates |

**Scope of the claim:** These are potential reductions for a customer whose requirements match the supported capability. Existing shared services may already handle the work effectively. Fewer custom integrations do not establish lower total cost, complete feature parity or better architecture; account for migration, service configuration, preview limitations and ongoing governance before retiring anything.

**Evaluation continuity:** B's hypothetical scenario already selected Foundry Evaluations, so C does not add another evaluation platform or count evaluation consolidation again. A customer with an existing evaluation investment can retain it; migration or dual operation is not required. AI Red Teaming is a separate optional adversarial testing decision.

## When This Is Justified

- **Runtime:** The team wants managed hosting, scaling, conversations, and lifecycle. Choose a configuration-based prompt agent or a hosted code-based agent according to the required execution model. Neither implements the business process for you.
- **Microsoft Agent Framework:** A developer framework, not another managed service. Customer-authored hosted code can use it or another supported framework. It can also run outside Foundry; do not draw MAF itself as a mandatory cloud boundary.
- **Toolbox:** Reuse approved tools and supported authentication connections across consumers. It can also be adopted directly from B by external MCP-compatible runtimes; moving runtime is not a prerequisite. Tool definitions and authentication need configuration, and enterprise APIs still enforce permissions. Client-side function calling is not automatically hosted by Toolbox.
- **Memory (preview):** Consider managed long-term memory only when cross-session context creates measurable value. Direct Memory Store APIs also support selective adoption without migrating the runtime. Compatible chat and embedding deployments are required; memory stores currently do not support VNet integration. Do not confuse memory with conversation history or an authoritative enterprise database.
- **Evidence and tracing:** Evaluation and preview AI Red Teaming are separate test workflows. The local red-team SDK guide explicitly warns it is incompatible with the new Foundry portal/SDK; this box is a capability option, not a claim that that local runner feeds the new portal. Prompt/hosted-agent tracing is GA after connecting Application Insights; custom application spans require instrumentation, and external/workflow agent tracing is preview.

Agent guardrails can override the underlying model guardrail rather than add another cumulative layer. Explicitly configure applicable intervention points. Tool scanning is not authorization, and hosted code's arbitrary outbound actions must not be assumed intercepted.

## Customer Still Owns

- Agent instructions/code, workflow semantics, model choice, compatibility tests, failure recovery, and dependency maintenance for hosted code.
- Tool implementations, API contracts, consent, approval UX, least-privilege identities, backend authorization, and network policy.
- Memory scope selection, consent, deletion/retention policy, isolation tests, and correctness of recalled information.
- Evaluation and red-team scope, datasets, judge calibration, human review, remediation, and release gates.
- Telemetry instrumentation, sensitive-data redaction, retention, access control, costs, and end-to-end operations.

## Adoption Boundary

Remaining at B, or selecting only one C capability while keeping an external runtime, is a valid outcome. For each potential addition, compare the work transferred with its constraints, migration effort, and ongoing cost. The five options are not a migration checklist. This repository implements only the Model + Guardrails + Evaluations sample; C is conceptual.

Sources: [Agent Service](https://learn.microsoft.com/azure/foundry/agents/overview), [Agent Framework](https://learn.microsoft.com/agent-framework/overview/), [Toolbox](https://learn.microsoft.com/azure/foundry/agents/concepts/toolbox-overview), [tracing](https://learn.microsoft.com/azure/foundry/observability/concepts/trace-agent-concept).