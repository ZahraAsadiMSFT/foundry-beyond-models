# Architecture Scope and Technical Caveats

This reference explains the architecture's supported boundaries and links to product documentation. It is not a compatibility certification for a particular tenant, model, or API. Recheck current availability and preview status before adopting a capability.

## Verdict

The three architectures are **choices on an adoption spectrum**, not a maturity ladder. [A](architecture/01-model-only.md) is a legitimate composable stack; [B](architecture/02-selective-foundry.md) is the scope of this sample; [C](architecture/03-deeper-foundry.md) is an optional response to different operational needs.

The sample combines **model Guardrails + offline Evaluations** with existing model access. Supported model checks can execute on the existing inference path, and Foundry can evaluate externally produced outputs [S1-S4]. Verify policy assignment and actual outcomes on a supported deployment, and run a dataset-target evaluation without creating an agent. Follow the [getting-started guide](../README.md) for setup.

Do not claim a measured cost reduction. Removing a maintained safety adapter or scoring/reporting job is a possible reduction in integration work, conditional on coverage parity. If an existing shared platform already does this well, incremental value may be small. Fewer diagram boxes are not evidence of lower total cost.

## Capability Tradeoffs

| Candidate | Architectural value | Constraint / decision |
|---|---|---|
| Guardrails + Evaluations | A named policy attaches to an existing endpoint; external outputs become scored, reviewable evidence. App and framework remain. | **Implemented scope.** Portal configuration, synthetic data, no runtime migration. |
| Evaluations + AI Red Teaming | Measurement plus adversarial discovery; an alternative for a team already satisfied with safety middleware. | Not implemented. Check availability, execution cost, and SDK/portal compatibility. |
| Agent Service + Toolbox | Managed runtime plus reusable tool connections can reduce repeated hosting/authentication work. | Not implemented. Relevant when multiple agents share tools; Toolbox can also be adopted without moving runtime. |
| Guardrails + Memory | Cross-session recall with managed extraction, consolidation, and retrieval. | Not implemented. Memory is preview, needs chat/embedding deployments, and introduces privacy/scope/network requirements. |

## Capability and Availability Ledger

**Status discipline:** GA is stated only where the cited source establishes it. "Not marked preview" means documented mainstream functionality, not a blanket GA/SLA certification for every model, API, region, or integration. Reconfirm the exact SKU/API before use; linked documentation may change.

| Capability | Documented architecture / status | Architectural implication |
|---|---|---|
| Microsoft Foundry / Models | Current product name is Microsoft Foundry. Model guardrails cover Models sold by Azure, with audio-transcription exceptions [S1]. | Pick a supported text deployment; do not generalize to every catalog model or hosting option. |
| Model Guardrails | Named controls, deployment assignment, user-input and output interception. Standard safety and user-prompt protection are not marked preview in the overview; agent controls are preview [S1-S2]. | Model policy; default filtering exists in A already. |
| Advanced guardrails | PII, runtime groundedness, task adherence, spotlighting and agent intervention points are marked preview; applicability differs [S1]. | Leave PII/domain checks external; do not promise universal enforcement. |
| Evaluations | Portal dataset-target individual-turn evaluation scores preexisting outputs. Basic Groundedness/Relevance are not marked preview in the catalog [S3-S5]. | No Agent Service requirement for B. Judge deployment and explicit data mappings remain. |
| Advanced evaluators | Task Completion, Task Adherence, Intent Resolution, Quality Grader, custom and rubric evaluators are preview [S5-S6]. | Outside the sample's required scope. Tool evaluators need supported traces/tool schemas, not only final text. |
| Agent Service | Managed prompt agents and code-based hosted agents; current overview describes both, without a blanket preview label [S7]. | Optional runtime. Verify chosen runtime/region/version independently before making a GA claim. |
| Microsoft Agent Framework | Developer framework, successor to Semantic Kernel and AutoGen, not a managed service. Python/.NET/Go surfaces differ; Go is preview and the .NET example includes a prerelease package [S8]. | Existing frameworks may stay. Do not claim every package is GA or migration is required. |
| Toolbox | Shared MCP-compatible endpoint for external or Foundry consumers; supported auth and version management. Core overview is not marked preview; tool search and skills are preview [S9]. | Independent selective-adoption option. Client-side function calling is not a Toolbox-hosted tool. |
| Memory | Memory and Memory Store API are public preview. Managed extraction/consolidation/retrieval and item/TTL controls; chat and embedding deployments required. No VNet integration for memory stores [S10]. | Optional only; explicit scope, consent and deletion tests remain customer work. |
| AI Red Teaming | Automated adversarial discovery/reporting is documented as preview. Local SDK guide explicitly warns it is incompatible with Foundry (new) portal/SDK [S11]. | Optional architecture only; do not assume local SDK results integrate with a different portal/SDK. |
| Tracing | Tracing for prompt and hosted agents is explicitly GA; workflows and external agents are preview [S12]. Application Insights is separately connected and billed. | Retain existing telemetry in B. C shows the separate Azure resource; custom app spans still need instrumentation. |

## Guardrails: What Actually Consolidates

A guardrail groups risk, intervention point and action controls. The portal can assign it to supported models/agents. REST represents it as an ARM RAI policy; creating/updating that policy and setting deployment `raiPolicyName` are separate management operations [S2]. Enforcement runs on supported inference requests. This is **integrated policy management and enforcement**, not one universal safety API.

| Control | Intervention / configuration boundary |
|---|---|
| Standard content safety | User input and output on supported model deployments; defaults already apply. App handles annotations, blocked input/output, errors and streaming behavior. |
| User prompt attack protection | User-input control. Verify assignment and observable annotation/block behavior on the actual API. A normal model refusal is not proof the guardrail fired. |
| Indirect prompt attacks | Requires compatible request/context formatting; document delimiters and API differences matter. It is not automatic inspection of all enterprise data. |
| Protected material | Applicable output controls and annotations; the app may still need to display code citations and respect licenses. |
| PII (preview) | Supported policy/API combinations can detect/filter/redact. This is not a guarantee that PII never reaches Azure or never enters logs. Keep pre-transmission minimization. |
| Runtime groundedness (preview) | Compatibility and streaming requirements apply. The configuration guide restricts this to streaming scenarios. Do not substitute it for domain validation or offline groundedness scoring. |
| Tool calls/responses | Agent-specific preview intervention points. An external orchestration loop does not receive these merely by calling a model endpoint. Toolbox governance is separately configured and must be tested for its own path. |

Request-time policy override (`x-policy-id`) is documented for supported calls, so deployment assignment alone is not a complete governance boundary. Restrict callers/configuration and test override behavior. Assigned agent guardrails can override the underlying model policy; do not draw two additive safety layers. Separate safety-service calls remain necessary for unsupported paths, pre-transmission controls, and retained specialist checks.

## Evaluations: Scope and Ownership

External/custom workflow: collect outputs, map data, invoke existing judges or rules, schedule execution, store results, compare versions, and make release decisions. Foundry can supply selected evaluators, run execution and reports; the team still supplies the test harness, data, mappings, calibration and release gate. An existing platform such as DynamoAI can remain the system of record; this is not a feature-parity comparison.

| Category | Current support and required evidence |
|---|---|
| Quality | Coherence and fluency; judge-based quality is evidence, not factual proof. |
| Relevance / groundedness | Query-response relevance and support in supplied context. Groundedness scores 1-5; it cannot establish whether the source context itself is true. Supply context or the equivalent supported message structure. |
| Safety | Standard risk categories plus specialist evaluators in the catalog. Check regional/service dependencies per evaluator; safety evaluation is not runtime enforcement. |
| Task completion / adherence | Preview; include instructions and interaction evidence. A plausible final answer is not proof of a completed backend action. |
| Tool behavior | Selection, arguments, output utilization, success and overall accuracy. Supply tool definitions and call/result evidence; verify tool support and treat errors/unsupported cases separately from passes. |
| Custom | Preview code-based, prompt-based and endpoint-based evaluators [S6]. Customer owns scoring correctness, hosting where applicable, and sensitive-data handling. |

The sample uses **Dataset**, not Model or Agent, as the evaluation target [S3]. Outputs can come from outside Foundry without rerunning or relocating the application. Judge calls still occur. Use one frozen dataset version and judge configuration for comparisons; report failures/missing scores separately and avoid statistical claims from a tiny sample.

## Complexity Comparison

| Capability | Before / external approach | With Foundry | What gets simpler | What customer still owns |
|---|---|---|---|---|
| Model injection/safety checks | Selected middleware calls, auth, result adapters | Supported endpoint policy enforcement | Selected calls/adapters can retire after parity testing | Policy design, access, false positives, error UX, residual controls |
| Evaluations | Operated scoring jobs and report aggregation, or existing vendor platform | Dataset evaluation, managed evaluators and results | Selected execution/reporting infrastructure | Data, mappings, judge cost, calibration, gates, remediation |
| AI Red Teaming (preview) | Attack corpus, strategy orchestration, target adapter, result aggregation | Curated attack strategies and scorecards | Selected test generation/report plumbing | Scope, safe target, adapter, compatibility, human review |
| Agent runtime (optional) | Host/scale runtime and conversation lifecycle | Prompt or hosted Agent Service | Runtime hosting and lifecycle operations | Workflow semantics, code/dependencies for hosted agents, recovery |
| Toolbox (optional) | Repeated tool definitions, credentials, updates per consumer | Shared MCP endpoint and configured connections | Reuse and supported credential lifecycle | Tool API contracts, permissions, approval, compatibility tests |
| Memory (preview) | Extraction, storage, consolidation and retrieval pipeline | Managed store and memory APIs/tool | Selected long-term context pipeline | Scope/consent, model dependencies, retention/deletion, correctness |
| Tracing (optional) | Instrumentation and existing collector/backend | Foundry views over Application Insights | Managed-agent visibility after setup | External instrumentation, telemetry privacy, retention, cost |
| PII / enterprise authorization | Local minimization and backend access enforcement | Retained in B | No consolidation claimed | All relevant data and action authorization decisions |

## Red Teaming Decision

Evaluation measures selected behavior; red teaming searches for failures; guardrails mitigate selected risks at runtime. Findings can become regression cases and policy changes, but there is no automatic proof-of-safety loop. The local guide contains both a single-turn-only statement and multi-turn strategy entries: treat multi-turn support as unresolved for that path. A callback-based test does not imply compatibility with the new portal.

No red-team runner is included in this repository. When evaluating that optional capability, preserve report provenance and supported target scope. Illustrative scores are not measured results, and a zero attack-success rate is not comprehensive safety assurance.

## Adoption Checklist

- [ ] Confirm supported text model, region, quota, endpoint/API version and current policy assignment in a non-production project.
- [ ] Verify control availability, caller override exposure, input/output annotations and blocked/error handling. Keep default safety enabled.
- [ ] Resolve documentation inconsistencies: S1's low/high restrictiveness descriptions conflict with its threshold definitions; S2 mixes older API examples with newer product guidance. Do not repeat those labels or its latency estimate as a guarantee.
- [ ] Confirm Dataset target, judge connection, Groundedness/Relevance input mappings, completed run and row-level reasons in the actual tenant.
- [ ] Record evaluator/model versions, sample count, missing/error cases and dataset provenance. Do not attribute hand-edited output improvements to guardrails.
- [ ] Confirm current GA/preview terms for any optional feature mentioned. Treat C as conceptual until its combined dependencies are tested, especially Memory networking and Toolbox tool/auth support.
- [ ] For optional red-team reports, verify the producing portal/SDK, target, date, risk scope and limitations. S11 does not establish new-portal compatibility.
- [ ] Measure cost/latency on the actual workload before making numeric claims.

## Sources

First-party documentation supports the design; it does not certify tenant availability. Check the linked pages for current requirements and preview status.

- **S1:** [Guardrails overview](https://learn.microsoft.com/azure/foundry/guardrails/guardrails-overview).
- **S2:** [Configure guardrails: portal, REST and annotations](https://learn.microsoft.com/azure/foundry/guardrails/how-to-create-guardrails).
- **S3:** [Run evaluations from the portal](https://learn.microsoft.com/azure/foundry/how-to/evaluate-generative-ai-app).
- **S4:** [Evaluate existing datasets](https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-datasets).
- **S5:** [Built-in evaluator catalog](https://learn.microsoft.com/azure/foundry/concepts/built-in-evaluators).
- **S6:** [Custom evaluators](https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/custom-evaluators) and [agent evaluators](https://learn.microsoft.com/azure/foundry/concepts/evaluation-evaluators/agent-evaluators).
- **S7:** [Foundry Agent Service](https://learn.microsoft.com/azure/foundry/agents/overview).
- **S8:** [Microsoft Agent Framework](https://learn.microsoft.com/agent-framework/overview/).
- **S9:** [Toolbox](https://learn.microsoft.com/azure/foundry/agents/concepts/toolbox-overview).
- **S10:** [Memory](https://learn.microsoft.com/azure/foundry/agents/concepts/what-is-memory).
- **S11:** [Local AI Red Teaming Agent: preview and compatibility warning](https://learn.microsoft.com/azure/foundry/how-to/develop/run-scans-ai-red-teaming-agent).
- **S12:** [Tracing setup and availability](https://learn.microsoft.com/azure/foundry/observability/how-to/trace-agent-setup).