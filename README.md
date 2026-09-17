# Foundry Beyond Models

Keep the same agent logic while choosing which integrations and platform operations to manage separately or consume from Foundry. Neither choice necessarily means fewer lines of code or less total system complexity; existing third-party and custom implementations remain valid options.

## Demo Structure

1. **Architecture:** the image below and [architecture choices](docs/architecture/03-deeper-foundry.md) describe the broader incremental adoption story.
2. **Code comparison:** [demo/comparison/foundry_development_comparison.py](demo/comparison/foundry_development_comparison.py) compares Tools, Guardrails, and Evaluations around a shared agent definition and runner.
3. **Optional runnable sample:** [demo/model_demo.py](demo/model_demo.py) makes a real model call. The [technical sample guide](docs/model-sample.md) covers setup, response interpretation, and a separate offline evaluation exercise.

## Architecture

[![Incremental Foundry adoption: keep business logic while transferring selected AI-platform responsibilities.](foundry.png)](foundry.png)

## What changes when I consume more of Foundry?

| Capability | Build / integrate separately | Consume from Foundry |
| --- | --- | --- |
| Agent runtime | Your hosting/runtime | Agent Service |
| Tools | Individual API/MCP connections | Toolbox |
| Knowledge | Custom RAG integration | Foundry Knowledge |
| Memory | Custom memory/store pipeline | Memory |
| Safety | External safety integration | Guardrails |
| Evaluation | External evaluation workflow | Evaluations |
| Red teaming | Separate/custom tooling | AI Red Teaming |
| Observability | Custom/separate observability integration | Foundry tracing & monitoring + Application Insights |

These are architectural choices, not a required migration path. Foundry capabilities can be adopted independently when they fit the application's requirements. The code comparison in this repo implements three representative examples—Tools, Guardrails, and Evaluations. The remaining rows illustrate additional adoption options and are not implemented as code comparisons in this demo.

## Scope and Caveats

The image is conceptual, not a deployed-resource diagram. MAF is the framework; Agent Service is the managed runtime. Configuration, application logic, backend authorization, testing, and operational accountability remain customer responsibilities.

The code comparison is architecture-as-code, not a runnable application: executing it only prints a notice. External HTTP contracts are illustrative, not vendor APIs. Guardrails require separate policy assignment and coverage tests; equivalent enforcement is not established. The code sample demonstrates offline evaluation outside the agent request path; Foundry also supports recurring/continuous evaluation scenarios. No live Toolbox integration or hosted-agent deployment is supplied.

See the [development guide](docs/development-comparison.md) and [technical assumptions and references](docs/architecture-validation.md) for details.
