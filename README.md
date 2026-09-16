# Foundry Beyond Models

**Keep your agent logic. Let Foundry manage more of the AI platform.**

A 1-2 minute code comparison: keep the same support agent while moving selected AI-platform integration and operations to Foundry.

## Start With the Code

Open **[demo/comparison/foundry_development_comparison.py](demo/comparison/foundry_development_comparison.py)**. One file, two architectures:

1. **SEPARATE SERVICES:** individual tool/auth connections plus application-owned memory, safety, telemetry, hosting and offline evaluation responsibilities.
2. **MORE FOUNDRY:** the same MAF agent with Toolbox connections and explicit notes showing what Agent Service, Tracing, Guardrails, Evaluations and conditional Memory can manage.

The file defines the agent/business logic once. Real MAF excerpts show tool connections; labeled comments show configuration and operations without inventing platform APIs. The customer keeps business logic, backend APIs, data and authorization.

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

- [Architecture A: model access](docs/architecture/01-model-only.md), [B: selective adoption](docs/architecture/02-selective-foundry.md), [C: deeper adoption](docs/architecture/03-deeper-foundry.md).
- [Technical assumptions and references](docs/architecture-validation.md).
- [Optional live Model + Guardrails + Evaluations exercise](docs/model-sample.md): the original working harness, setup and interpretation guidance. Separate from the main code demo.
