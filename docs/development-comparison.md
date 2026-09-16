# Why More on Foundry?

[foundry_development_comparison.py](../demo/comparison/foundry_development_comparison.py) defines a shared agent, two integration architectures, and safety/evaluation adapters. This is architecture-as-code, not a deployed application or a simulated live response. Consolidating selected integrations does not establish fewer lines of code or less total system complexity.

## Progressive Responsibility Transfer

The support-agent instructions, question and three tool contracts stay unchanged throughout. The two architecture sections show the responsibility contrast; the table below explains each capability's scope and retained ownership. For configured capabilities the source shows:

```python
# CONFIGURED / MANAGED IN FOUNDRY
# No longer implemented in this application path
```

These markers describe the named responsibility in a proposed adopted architecture. They do not claim configuration was applied or code was deleted from this sample. Step 0 is the existing managed-model baseline; steps 1-6 are further choices, not a mandatory dependency chain.

| Step | BEFORE: application/platform responsibility | AFTER: selected managed responsibility | What stays in the application/platform team |
|---|---|---|---|
| **0. MAF + Foundry Model** | Configure agent orchestration and integrations; otherwise arrange model serving too | Foundry serves the model; MAF provides developer-side orchestration | Agent runtime, configured client/identity, model choice, quota and error handling [S1,S2] |
| **1. + Toolbox** | Maintain per-service connections/auth in independently configured consumers | Curate supported connections/auth centrally behind one MCP endpoint | Toolbox client lifecycle/auth, allowlist, contracts and backend authorization [S3,S8] |
| **2. + Agent Service** | Operate agent-serving infrastructure, endpoint and scaling | Run compatible packaged code on the managed hosted-agent runtime | Serving entry point, packaging/deployment, updates, networking, identity and app recovery [S2] |
| **3. + Tracing** | Emit/correlate agent telemetry and assemble investigation views | With Application Insights connected, use supported hosted-agent server spans and trace views | Separate telemetry resource/access/retention, privacy, alerts and custom app/backend spans [S7] |
| **4. + Guardrails** | Integrate/enforce separate safety-service checks where extra controls are required | Assign supported policies and use enforcement at the applicable service intervention points | Coverage/failure tests, filter/error handling, domain/privacy checks and tool authorization [S5] |
| **5. + Evaluations** | Separately operate scoring jobs/integrations and aggregate reports | Submit prepared data for supported Foundry evaluation execution and result views | Output collection, dataset/mappings, judges, thresholds, scheduling/CI and release decisions [S6] |
| **6. + Memory, conditional** | If needed, integrate/operate long-term extraction, consolidation, storage and retrieval | Use managed Memory Store operations through a verified compatible integration | Trusted scope, lifecycle integration, consent, retention/deletion and authoritative enterprise tools [S4] |

Both entry points call `run_support()`, which registers shared application middleware and `model_call_checks()`. The separate path additionally supplies `external_safety_check()`. The documented [MAF chat middleware](https://learn.microsoft.com/agent-framework/agents/middleware/) runs inside the tool loop for each model call. The adapter maps text and local function exchanges, including tool results; it rejects unsupported content and streaming. It is not a complete safety product or a claim of policy equivalence. Apply privacy/minimization checks before either service receives data, and validate the actual deployment's coverage before removing external checks.

Both offline submitters accept the same versioned collected-output JSONL. Use relevance as the common evaluation goal, with `foundry_relevance_criteria()` on Foundry and an explicitly mapped external criterion/judge. Different scoring implementations are not assumed equivalent. Run IDs and the Foundry report URL are returned for later review; submission, status checks, metric-error review and release gates remain customer responsibilities. No evaluation runs inside the agent request.

## Step 0: Keep the Business Logic

Both architectures use the same support question:

> For order DEMO-1042, is Item A covered and how do I contact support?

The same MAF agent uses three approved read-only tools in both designs. The intended backend contracts are:

| Tool | Input | Returns |
|---|---|---|
| `get_order` | `order_id` | Authorized order details, including item and purchase date |
| `get_warranty` | `item_id` | Applicable warranty terms |
| `get_support_steps` | `item_id` | Approved support contact/process |

These are illustrative enterprise contracts, not provided servers or actual order data. Foundry does not implement the business APIs. Each backend must enforce the authenticated caller's access.

## Step 1 Before: Direct Connections

`separate_services()` uses `service_urls` and `auth_providers` to connect each service individually.

This version already uses MAF, MCP and a Foundry model. The framework handles tool calling; we do not inflate the baseline by hand-writing it. The application/platform team supplies separate connection and authentication configuration for each service.

## Step 1 After: The Foundry Connection

`more_foundry()` uses `toolbox_url` and `toolbox_auth_provider` for one consumer connection. Both entry points use the same `run_support()` implementation and `build_agent()` definition.

Toolbox exposes a curated collection through one MCP-compatible endpoint. Configure the same three tool names/schemas and supported backend authentication in Toolbox. The caller still authenticates to Toolbox; the backends still authorize requests. The application allowlist remains explicit [S3,S8].

### Connection Maintenance

Suppose the support service changes its endpoint or authentication configuration, keeping its contract stable:

- **Direct:** update the affected service/auth configuration in each independently configured consumer or its shared platform configuration.
- **Toolbox:** update/test the supported connection centrally and promote the intended Toolbox version. Consumers using that default can keep their endpoint and code; pinned consumers need a planned version update.

This example describes a configuration change; it does not apply one. Centralized connections can reduce per-consumer integration maintenance. Shared gateways can already offer similar benefits; assess the incremental value if one exists.

## Step 2: Hosting Responsibilities

**The Toolbox version can still run outside Foundry.** Adopting Toolbox does not require moving the runtime [S2,S3].

| Keep your runtime | Optionally use Agent Service |
|---|---|
| Operate your chosen agent-serving infrastructure and scaling | Deploy compatible packaged code to a managed endpoint/runtime |
| Configure your telemetry pipeline and investigation views | Connect Application Insights and use supported server-side agent traces |
| Maintain agent code, dependencies and backend integration | Still maintain agent code, dependencies and backend integration |

Required work has not vanished: compatible serving entry point, packaging/deployment, identity/RBAC, model configuration, network access and an Application Insights connection. Custom spans, alerts, privacy, application recovery and service costs remain yours. No hosting or telemetry implementation is included in these excerpts [S2,S7].

## Step 3: Add Observability / Tracing

**BEFORE:** configure agent telemetry emission/export and correlated investigation views. **AFTER:** for the hosted-agent path in step 2, connect Application Insights to the project and use supported server-side spans and Foundry trace views. The application does not have to emit those server spans or build their viewing experience [S7].

The agent call remains unchanged. Custom application/backend spans still require instrumentation; end-to-end correlation, alerts, privacy and telemetry operations remain yours. External/workflow agent tracing is preview. Connecting Application Insights does not instrument arbitrary code, and the resource has separate access, retention and billing. These examples configure neither hosting nor tracing.

## Step 4: Add Guardrails

**BEFORE:** where extra checks are required, maintain a separate safety-service adapter and enforce its decisions. **AFTER:** configure and assign supported controls on the chosen model/API or agent intervention point. The service can perform those covered checks/actions without that separate adapter in the application path [S5].

The baseline already has model default protections. The comparison includes an illustrative external safety adapter; its removal is conditional on equivalent coverage and tested failure behavior, not a claim that all safety code disappears. Keep domain checks, pre-transmission privacy controls, backend authorization, filter/error handling and false-positive tests. Agent/tool controls may be preview or path-specific. Policy creation, assignment and inference remain separate; the business instructions do not change.

## Step 5: Add Evaluations Outside the Request Path

**BEFORE:** separately operate scoring workers/jobs, evaluator integrations and report aggregation, or use an external provider. **AFTER:** submit prepared data to a Foundry evaluation run and use supported evaluator execution and result views [S6].

Keep this **outside the request functions and `agent.run()`**. Configure the dataset, mappings, judge and criteria in a separate evaluation workflow; review results and wire release gates separately. Dataset evaluation scores supplied responses, not newly executed agent outputs. The existing hand-authored model fixtures are not outputs from this tool-using agent: a real evaluation needs collected representative outputs with provenance. Data preparation, permissions, scheduling, judge usage and release decisions remain customer-owned.

## Step 6: Add Memory Only Where Applicable

**BEFORE:** if cross-session continuity is required, integrate/operate a scoped long-term memory pipeline. **AFTER:** use a preview Memory Store to perform the selected extraction, consolidation, persistence and retrieval operations through a verified compatible integration [S4].

The current support request is stateless: there is no memory implementation to remove, and this step can be skipped. Keep the same support instructions and tools. Any memory/context integration surrounds that logic; recalled context must not replace authoritative order/warranty facts or grant access.

For hosted MAF code, do not assume the prompt-agent memory-search tool is automatically attached. Verify a supported API/context integration and retain its lifecycle calls, explicit trusted scope, consent, retention/deletion and recall/isolation tests. Compatible chat/embedding deployments and store configuration are required. Memory stores currently lack VNet integration. Memory is not a general vector database, enterprise knowledge base or replacement for all session history. No memory integration is supplied here.

## Example Boundaries

- MAF `Agent`, `MCPStreamableHTTPTool`, `header_provider`, `allowed_tools` and async lifecycle patterns follow the current documentation [S1,S8]. The shared builder is implemented, not a fictional `configured.*` helper.
- The caller must supply a compatible, configured model client (for example `FoundryChatClient`), trusted URLs and refresh-capable auth providers. They must work for connection/discovery as well as tool calls, use the right audiences/scopes, and isolate sessions by principal. No credentials are embedded.
- Tool names and contracts must match across direct endpoints and Toolbox. The snippets do not prove identical outputs or tool selection. SDK integration, timeout/error handling, application-level limits and security tests are still needed for a real application.
- Imports are deferred so executing a source file prints only its notice with no SDK installation or cloud calls. There is no generated customer answer, benchmark or hosted deployment hiding behind the examples.
- The [original live model exercise](model-sample.md#run-the-model-sample) is separate and unchanged. It tests neither Toolbox nor Agent Service. The broader [architecture choices](architecture/03-deeper-foundry.md) remain incremental.

## Validate the Image's Assumptions

| Assumption | Verdict | Accurate wording |
|---|---|---|
| MAF + Foundry can reduce platform plumbing | **Supported, conditional** [S1-S3] | MAF organizes agent logic; selected Foundry services replace specific operated integrations where coverage matches. |
| A smaller agent script proves less total code | **Not established** | Include adapters, configuration, deployment, tests and operations in a fair comparison. No line-count, time, team-size or cost savings are measured here. |
| `from azure.ai.agents import Agent, tool` is the MAF API | **Do not use as MAF code** [S1] | The documented Python framework uses `agent_framework`, including `Agent`, and `agent_framework.foundry.FoundryChatClient`. The image mixes illustrative SDK concepts. |
| Calling `agent.run()` deploys/runs an agent in Agent Service | **Incorrect** [S1-S2] | Await the agent call in an async context. Managed hosting requires a separate compatible deployment. |
| A decorated local tool automatically uses Toolbox | **Incorrect** [S3] | A local function executes in the agent process. Toolbox requires supported connections and a client consuming its endpoint. |
| Memory replaces the external vector database | **Only for a matching memory use case** [S4] | Managed long-term memory can replace a selected memory pipeline. It is not a general database, enterprise knowledge base or automatic replacement for retrieval. |
| Memory, policies, evaluations and observability are automatically associated | **Incorrect as a blanket claim** [S4-S7] | Configure each supported integration explicitly. Evaluations are separate jobs; custom code telemetry needs instrumentation. |
| All safety checks and deployment/telemetry code disappear | **Overstated** [S2,S5,S7] | Supported server-side checks, hosting operations and trace views can be managed. Application checks, packaging, configuration, custom spans and operational accountability remain. |
| The same agent is guaranteed to behave identically | **Not demonstrated** | Preserve task, model, tools and fixtures, then test safety semantics, memory recall, authorization and output quality. A conceptual comparison is not behavioral parity evidence. |
| Everything must move together | **Incorrect** [S2-S4] | Keep external runtimes, memory or evaluation systems. Adopt only the managed capabilities that justify their dependencies. |

## Sources and Verification Boundary

The judgments above are grounded in current first-party documentation, not a deployed agent test. Python source files are syntax-checked and their notice-only entry points run locally. SDK packages were not installed or executed for these excerpts. Availability and preview behavior must be checked for the chosen deployment.

- [S1: Microsoft Agent Framework overview][S1]
- [S2: Agent Service and hosted agents][S2]
- [S3: Toolbox and supported tool types][S3]
- [S4: Memory, integration paths and limitations][S4]
- [S5: Guardrail configuration and applicability][S5]
- [S6: Dataset evaluation workflow][S6]
- [S7: Tracing setup and Application Insights][S7]
- [S8: MAF MCP tools, authentication and lifecycle][S8]

[S1]: https://learn.microsoft.com/agent-framework/overview/
[S2]: https://learn.microsoft.com/azure/foundry/agents/overview
[S3]: https://learn.microsoft.com/azure/foundry/agents/concepts/toolbox-overview
[S4]: https://learn.microsoft.com/azure/foundry/agents/concepts/what-is-memory
[S5]: https://learn.microsoft.com/azure/foundry/guardrails/how-to-create-guardrails
[S6]: https://learn.microsoft.com/azure/foundry/how-to/evaluate-generative-ai-app
[S7]: https://learn.microsoft.com/azure/foundry/observability/how-to/trace-agent-setup
[S8]: https://learn.microsoft.com/agent-framework/agents/tools/local-mcp-tools