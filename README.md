# Foundry Beyond Models

Why use Microsoft Foundry beyond simply calling a model API? Model access does not address every AI-specific responsibility: teams also need to configure safety controls, evaluate outputs, and operate the integrations connecting those capabilities.

**Foundry can be adopted incrementally.** Use managed capabilities where they reduce AI-specific integration and operational work without moving the entire application into Foundry. Existing frameworks, enterprise systems, data, release decisions, and business logic can remain under your control. Independently operated alternatives remain valid choices.

## Architecture

[![The same application across three choices: independently integrated services, selective Foundry adoption, and optional deeper managed capabilities. Customers retain business logic and enterprise systems while reducing selected AI-platform integration and operational work.](foundry.png)](foundry.png)

*Select the image to view it at full size.*

**Same application, less AI-platform plumbing, your choice.** This conceptual overview shows broader adoption options, not everything implemented here. This sample selects **Model + Guardrails + Evaluations**; Toolbox, Memory, Agent Service and Foundry tracing remain optional. The detailed diagrams below distinguish MAF as a developer framework from Agent Service as a managed runtime. Enterprise APIs/data and authorization remain customer-owned; Foundry trace views use separately connected Application Insights. Connections inside the illustration indicate integration, not automatic migration, universal safety coverage, or measured savings.

| Stage | Approach | What changes |
|---|---|---|
| [A: Model access](docs/architecture/01-model-only.md) | Foundry hosts the model; other AI-specific responsibilities are independently operated. | A composable baseline, not an inherently bad architecture. Model default safety already applies. |
| [B: Selective adoption](docs/architecture/02-selective-foundry.md) | Add managed Guardrails and Evaluations. | Covered safety integrations and evaluation execution/reporting may be consolidated; the application stays external. |
| [C: Optional deeper integration](docs/architecture/03-deeper-foundry.md) | Consider additional managed capabilities individually. | Future choices, not a required migration or an implemented part of this sample. |

## What This Sample Implements

**Model + Guardrails + Evaluations**, with deliberately small boundaries:

- A Python command-line harness sends one synthetic request to an existing model deployment and reports selected real filter annotations/errors.
- You configure and assign a model Guardrail in the Foundry portal. The harness performs inference only; it does not create or change policies.
- You upload ten synthetic, prewritten input/output fixtures and evaluate them in the portal. There is no evaluation SDK workflow.

No agent runtime, tools, memory, retrieval service, UI, database, infrastructure automation, or additional Foundry capability is implemented. No completed evaluation scores or saved service results are shipped.

## Prerequisites

- Git, Python 3.11, and [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli).
- An Azure subscription with access to [Microsoft Foundry](https://ai.azure.com/), a Foundry resource/project, model quota, and network access to its endpoint. Model and evaluation judge calls incur charges.
- A deployed **GPT-5** model supporting Chat Completions and `reasoning_effort="minimal"`. The harness uses GPT-5 parameters; do not assume any model is interchangeable. Availability varies by region and subscription.
- Inference permission on the resource, such as **Cognitive Services OpenAI User**. Management-plane access alone does not establish inference access.
- Permission to configure model Guardrails (see the [Guardrail prerequisites](https://learn.microsoft.com/azure/foundry/guardrails/how-to-create-guardrails)), and **Foundry User** or equivalent evaluation permissions on the project plus access to a supported judge deployment. Keep policy administration separate from inference permissions.

Use a non-production deployment. Check other consumers before changing a shared deployment's policy. Follow your organization's browser, network, and identity requirements.

## Repository Structure

| Path | Purpose |
|---|---|
| [demo/model_demo.py](demo/model_demo.py) | Single-request model harness and filter-evidence interpretation. |
| [demo/cases.jsonl](demo/cases.jsonl) | Ten synthetic evaluation fixtures, including deliberate negatives. |
| [demo/requirements.txt](demo/requirements.txt) | Pinned direct Python dependencies. |
| [demo/.env.example](demo/.env.example) | Nonsecret configuration reference; not loaded automatically. |
| [docs/architecture/01-model-only.md](docs/architecture/01-model-only.md) | Architecture A and retained ownership. |
| [docs/architecture/02-selective-foundry.md](docs/architecture/02-selective-foundry.md) | Architecture B and conditional integration reductions. |
| [docs/architecture/03-deeper-foundry.md](docs/architecture/03-deeper-foundry.md) | Optional Architecture C choices. |
| [docs/architecture-validation.md](docs/architecture-validation.md) | Technical scope, capability caveats, and product references. |

The harness creates `demo/evidence/` only when `--save` is used. Generated captures and local environments are ignored by Git.

## 1. Clone and Install

Clone this repository, then run subsequent commands from its root. For a fork, substitute your fork's clone URL.

### PowerShell

```powershell
git clone https://github.com/ZahraAsadiMSFT/foundry-beyond-models.git
cd foundry-beyond-models
python -m venv demo/.venv
./demo/.venv/Scripts/python.exe -m pip install -r demo/requirements.txt
```

### macOS / Linux (Bash)

```bash
git clone https://github.com/ZahraAsadiMSFT/foundry-beyond-models.git
cd foundry-beyond-models
python3.11 -m venv demo/.venv
./demo/.venv/bin/python -m pip install -r demo/requirements.txt
```

Activation is unnecessary when using these interpreter paths. Dependencies are `openai`, `azure-identity`, and `azure-core`; the latter is directly imported for authentication errors. Direct versions are pinned, not the full transitive dependency tree.

## 2. Set Up the Model and Guardrail

1. Open [Microsoft Foundry](https://ai.azure.com/) and create or select your resource and project.
2. Deploy GPT-5 from the model catalog, subject to availability and quota. Record the **deployment name**, which may differ from the model name.
3. Copy the resource's OpenAI endpoint from the deployment details. Use `https://<resource-name>.openai.azure.com`, not a project URL ending in `/api/projects/<project-name>`.
4. Open **Build > Guardrails** (portal labels may change). Create or select a model policy, review standard content safety and user-prompt protection, and inspect the available actions, thresholds, and intervention points for your model/API.
5. Preserve default protections and assign the policy to your non-production model deployment. Policy creation, deployment assignment, and inference are separate operations. A policy name alone does not enable a detector.
6. Verify the assignment in the portal. Do not infer its name or complete coverage from an inference response. [Guardrail setup documentation](https://learn.microsoft.com/azure/foundry/guardrails/how-to-create-guardrails) describes current permissions and supported controls.

This sample does not configure agent-tool controls or promise every Guardrail is available on every model API. Offline Groundedness evaluation is not runtime groundedness enforcement.

## 3. Authenticate and Configure

Sign in using Azure CLI and select the appropriate account, tenant, and subscription in the interactive flow:

```text
az login --output none
```

For an explicit tenant, use `az login --tenant <tenant-id> --output none`, replacing the placeholder locally. Do not commit your tenant ID or sign-in output.

The harness uses `AzureCliCredential` and a refreshable Entra token for `https://ai.azure.com/.default`. It does not use API keys or change your CLI subscription. Ensure this identity has inference permission on the chosen resource.

Set the two required environment variables in the terminal where you will run Python. Replace both placeholders with your deployment values.

### PowerShell

```powershell
$env:FOUNDRY_DEMO_ENDPOINT = 'https://<resource-name>.openai.azure.com'
$env:FOUNDRY_DEMO_DEPLOYMENT = '<deployment-name>'
```

### Bash

```bash
export FOUNDRY_DEMO_ENDPOINT='https://<resource-name>.openai.azure.com'
export FOUNDRY_DEMO_DEPLOYMENT='<deployment-name>'
```

[demo/.env.example](demo/.env.example) is a reference only. The code reads process environment variables and **does not load `.env` files**. No API key or dotenv package is required.

The endpoint validator accepts HTTPS resource roots or `/openai/v1/` URLs under `openai.azure.com` or `services.ai.azure.com`. Project endpoints, custom domains, and sovereign-cloud suffixes are not supported by this sample without a code review.

## 4. Run the Model Sample

### PowerShell

```powershell
./demo/.venv/Scripts/python.exe demo/model_demo.py --help
./demo/.venv/Scripts/python.exe demo/model_demo.py --case ordinary
./demo/.venv/Scripts/python.exe demo/model_demo.py --case instruction-conflict
```

### Bash

```bash
./demo/.venv/bin/python demo/model_demo.py --help
./demo/.venv/bin/python demo/model_demo.py --case ordinary
./demo/.venv/bin/python demo/model_demo.py --case instruction-conflict
```

- **Ordinary (`case-01`):** asks for Item A's support term from the reference. The intended answer is 12 months. It checks model access and lets you inspect returned filter annotations.
- **Instruction conflict (`case-10`):** adds a benign request to replace that reference fact with 999 months. It illustrates the difference between model instruction following and service-reported safety enforcement. It is not a guaranteed detector trigger or a comprehensive security test.

Only the fixture's `query` and `context` are sent. Its prewritten `response` is never supplied as an answer to imitate, and the script does not run all ten rows as a batch.

Each command makes one nonstreaming v1 Chat Completions request: at most 512 completion tokens including reasoning, minimal reasoning effort, no automatic SDK retries, and `store=False`. The HTTP timeout is 25 seconds per transport operation and CLI token acquisition has its own 15-second limit, not a strict overall deadline. Cancel with Ctrl+C as needed. `store=False` is not a claim about all service-side retention policies.

Microsoft generally recommends Responses for new applications. This sample deliberately uses supported [Chat Completions filter-response fields](https://learn.microsoft.com/azure/foundry/openai/how-to/reasoning) with the [v1 API](https://learn.microsoft.com/azure/foundry/openai/api-version-lifecycle); no dated inference API version is required.

## 5. Interpret Guardrail Results

| Interpretation | Meaning |
|---|---|
| `FILTER_ENFORCEMENT_REPORTED` | `filtered: true`, `finish_reason: content_filter`, or a documented filter error was returned. Inspect the field and intervention point. |
| `DETECTION_REPORTED_NOT_A_BLOCK` | A detector reported a risk, but there is no blocking evidence. |
| `ANNOTATIONS_PRESENT_NO_BLOCK_REPORTED` | Annotations were returned; no block was reported. This does not establish that every check succeeded. |
| `NO_FILTER_EVIDENCE_RETURNED` | No filter evidence was returned. Do not infer checks passed or filtering was disabled. |
| `REQUEST_FAILED_NOT_FILTER_EVIDENCE` | A non-filter HTTP failure, such as authorization or quota, is not safety evidence. |
| `AUTHENTICATION_UNAVAILABLE` / `CONNECTION_OR_TIMEOUT` | Check CLI sign-in, inference permissions, network access, and deployment availability. |

A correct answer or model refusal alone is **not** Guardrail enforcement. `detected: true` alone is detection, not blocking. Inspect nested `annotation_error_present` markers; a capture containing one does not show that all controls passed. `finish_reason: length` means the token cap was reached, not a safety block. Exit zero means an observable response/filter outcome, not that a blocking test passed.

Append `--save` to either command to write a new timestamped JSON capture under `demo/evidence/`. The capture includes selected annotations, the synthetic response, time, and dataset hash. It omits raw headers and raw exception messages, but this is **not a general-purpose secret scrubber**: changed inputs/model output may contain sensitive content. Review before sharing. Keep policy assignment and configuration records separately, without credentials or resource IDs. If neither case triggers a control, report that outcome; do not weaken default protections to manufacture a contrast.

## 6. Run Foundry Evaluations

The portal evaluates **existing responses**, independently of the Python harness. Running the two commands above does not replace or populate the ten fixture responses.

1. Open your project's **Evaluation** page and create a new evaluation with target **Dataset** and scope **Individual turns**. A Model or Agent target would be a different workflow that generates target outputs.
2. Upload [demo/cases.jsonl](demo/cases.jsonl) as a JSONL dataset and select its actual version. All ten rows contain synthetic `query`, `context`, `response`, `ground_truth`, `case_id`, and `origin` fields.
3. Map `query -> query`, `response -> response`, `context -> context`, and `ground_truth -> ground_truth` where requested. Groundedness requires the supplied context even if a generic mapper labels context optional. Keep `case_id` and `origin` for provenance, not as scoring criteria.
4. Select **Relevance** and **Groundedness**. Additional quality metrics are optional; inspect their input requirements. Choose a supported judge deployment with sufficient quota and permissions. Do not assume every model or metric combination is supported.
5. Review the dataset, mappings, judge, and criteria, then submit. Dataset evaluation does not rerun a target application to generate these responses, but model-based judges still make billable calls.
6. Wait for **Completed** and inspect per-metric errors as well as summary scores. `Partial` or `Failed` is not a completely successful run. Inspect row-level scores and reasons, not only averages.
7. Compare `case-01` and `case-02`, then `case-04`. Record actual results; scores are not guaranteed to match the fixture intent. Keep downloaded results local and sanitize anything you choose to publish.

See [portal evaluations](https://learn.microsoft.com/azure/foundry/how-to/evaluate-generative-ai-app) and [evaluation datasets](https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-datasets) for current UI and input requirements.

### Understand the Fixtures

The `response` values are **hand-authored examples, not measured GPT-5 outputs**. Negative examples deliberately contain mistakes. Scores on this dataset illustrate evaluation behavior, not model performance or a production benchmark.

| Example | Intended distinction |
|---|---|
| `case-01` / `case-02` | Supported 12-month answer versus unsupported 36-month answer. |
| `case-04` | An answer can be supported by context but irrelevant to the question. |
| `case-05` / `case-06` | Abstaining on missing information versus inventing a support term. |

Relevance concerns the response's relationship to the question. Groundedness concerns support in the supplied context, not universal factual truth. Evaluator reasons and disagreements require human review. You still own dataset representativeness, reference quality, thresholds, regression policy, and release decisions.

## Extending the Sample

Add representative synthetic fixtures, compare policy configurations on isolated deployments, or evaluate outputs produced by your own application using the same dataset approach. Keep generated responses distinct from hand-authored fixtures and retain provenance. Expanding the dataset alone does not add new harness CLI cases.

Consider [Architecture C](docs/architecture/03-deeper-foundry.md) only where a specific managed capability reduces work your team wants to transfer. Runtime, tools, memory, tracing, and adversarial testing remain separate choices, not prerequisites for this sample.

## Public Data Hygiene

[.gitignore](.gitignore) excludes local environment files (except the placeholder example), virtual environments, Python caches, and generated evidence. Keep credentials, subscription/tenant IDs, full resource IDs, private URLs, customer data, and unsanitized exports out of commits. Ignoring a file does not remove it from existing Git history; review staged changes and history before publishing. Revoke exposed credentials before any history-remediation work.
