# Foundry Beyond Models

**Keep the same agent logic. Consolidate selected integrations and move supported platform responsibilities to Foundry.**

The examples demonstrate integration choices, not necessarily fewer lines of code or less total system complexity.

## Code Comparison

[demo/comparison/foundry_development_comparison.py](demo/comparison/foundry_development_comparison.py) defines a shared agent and two architectures:

1. **SEPARATE SERVICES:** direct tool/auth connections and an external safety adapter registered on each model call.
2. **MORE FOUNDRY:** one Toolbox consumer connection and a model client targeting a deployment with separately assigned Guardrails. Both paths use the same agent runner and shared application checks.

The code shows connection consolidation and where a covered safety adapter can be removed. It does not establish equivalent policy enforcement: deployment assignment and coverage tests are prerequisites. The external HTTP contracts are illustrative, not vendor APIs.

Offline adapters accept the same collected-output JSONL, with explicit Foundry relevance criteria and SDK submission/result retrieval. Both retain orchestration code; Foundry supplies supported scoring execution and result views. Existing managed evaluators can offer these too. Hosting/tracing remain configuration notes, and this stateless example removes no memory pipeline. The customer keeps business logic, backend APIs, data and authorization.

This is architecture-as-code, not a deployed application. No setup is required; executing it only prints a notice. No live Toolbox/hosted-agent integration is supplied or claimed.

See the [development guide](docs/development-comparison.md) for detailed responsibility transfers and technical caveats.

## Why More on Foundry?

- **Toolbox:** reuse governed tool connections across agents, including agents hosted elsewhere.
- **Agent Service:** deploy compatible agent code to a managed runtime rather than operate its serving infrastructure yourself.
- **Observability:** connect Application Insights for supported server-side traces; keep custom application instrumentation where needed.
- **Guardrails and Evaluations:** transfer supported policy enforcement and offline scoring/reporting; retain coverage tests and release decisions.
- **Memory, where applicable:** use managed long-term memory operations through a compatible integration, not as a replacement for enterprise data or authorization.

Adopt capabilities independently; the sequence is not an automatically enabled bundle. Evaluations stay outside the synchronous request path. Platform configuration is shown as configuration, never as invented Python APIs.

## Architecture and Details

[![Incremental Foundry adoption: keep business logic while transferring selected AI-platform responsibilities.](foundry.png)](foundry.png)

The image is conceptual, not a deployed-resource diagram. MAF is the framework; Agent Service is the managed runtime.

- [Architecture choices and optional deeper integration](docs/architecture/03-deeper-foundry.md).
- [Technical assumptions and references](docs/architecture-validation.md).
- [Optional live Model + Guardrails + Evaluations exercise](docs/model-sample.md): the original working harness, setup and interpretation guidance. Separate from the main code demo.
