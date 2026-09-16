# Optional Live Model Sample

This is the original **Model + Guardrails + Evaluations** exercise, not the main [five-minute developer demo](development-comparison.md). It makes real model calls and supports a separate portal evaluation of synthetic fixtures. It does not implement Agent Service hosting, Toolbox or Memory.

## Prerequisites

- Python 3.11 and [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli).
- A Foundry resource/project with a non-production GPT-5 deployment supporting Chat Completions and `reasoning_effort="minimal"`, sufficient quota and network access. Model/judge calls incur charges.
- Inference permission such as **Cognitive Services OpenAI User**. Management permissions alone do not establish inference access.
- Separate permission to configure model Guardrails, and **Foundry User** or equivalent evaluation permission plus access to a supported judge deployment.

Check other consumers before changing a shared deployment's policy. Follow organizational identity, browser and network requirements.

## Install

Clone `https://github.com/ZahraAsadiMSFT/foundry-beyond-models.git` and run commands from the repository root.

PowerShell:

```powershell
python -m venv demo/.venv
./demo/.venv/Scripts/python.exe -m pip install -r demo/requirements.txt
```

macOS/Linux:

```bash
python3.11 -m venv demo/.venv
./demo/.venv/bin/python -m pip install -r demo/requirements.txt
```

[Requirements](../demo/requirements.txt) pin direct dependencies (`openai`, `azure-identity`, `azure-core`), not the full transitive tree. Activation is unnecessary with these interpreter paths.

## Configure

1. In [Foundry](https://ai.azure.com/), select a resource/project and deploy GPT-5, subject to regional availability and quota. Record the deployment name, which can differ from the model name.
2. Copy the resource OpenAI endpoint, not the project URL ending in `/api/projects/...`.
3. In **Build > Guardrails**, create/select a model policy and inspect supported actions and intervention points. Preserve default protections and assign the policy to the non-production deployment. Creation, assignment and inference are separate operations.
4. Verify the assignment in the portal; its name and full coverage cannot be inferred from the response. See [Guardrail setup and permissions](https://learn.microsoft.com/azure/foundry/guardrails/how-to-create-guardrails).
5. Sign in and select the intended account/tenant/subscription in the interactive flow:

```text
az login --output none
```

Use `az login --tenant <tenant-id> --output none` for an explicit tenant, substituting locally. Do not commit identity values or sign-in output.

PowerShell:

```powershell
$env:FOUNDRY_DEMO_ENDPOINT = 'https://<resource-name>.openai.azure.com'
$env:FOUNDRY_DEMO_DEPLOYMENT = '<deployment-name>'
```

Bash:

```bash
export FOUNDRY_DEMO_ENDPOINT='https://<resource-name>.openai.azure.com'
export FOUNDRY_DEMO_DEPLOYMENT='<deployment-name>'
```

The harness uses `AzureCliCredential` and a refreshable Entra token for `https://ai.azure.com/.default`, not API keys. It does not change your CLI subscription. [The environment example](../demo/.env.example) is a reference; `.env` files are not loaded.

Accepted endpoints are HTTPS resource roots or `/openai/v1/` under `openai.azure.com` or `services.ai.azure.com`. Project URLs, custom domains and sovereign-cloud suffixes require code changes.

## Run the Model Sample

PowerShell:

```powershell
./demo/.venv/Scripts/python.exe demo/model_demo.py --help
./demo/.venv/Scripts/python.exe demo/model_demo.py --case ordinary
./demo/.venv/Scripts/python.exe demo/model_demo.py --case instruction-conflict
```

Bash:

```bash
./demo/.venv/bin/python demo/model_demo.py --help
./demo/.venv/bin/python demo/model_demo.py --case ordinary
./demo/.venv/bin/python demo/model_demo.py --case instruction-conflict
```

`ordinary` uses case-01: Item A's support term is 12 months. `instruction-conflict` uses case-10: a benign request to replace that reference fact with 999 months. This is not a guaranteed detector trigger or comprehensive security test. Only `query` and `context` are sent, never the fixture's prewritten `response`.

Each command makes one nonstreaming v1 Chat Completions request, with at most 512 completion tokens including reasoning, minimal reasoning effort, no SDK retries and `store=False`. The 25-second HTTP timeout and separate 15-second CLI token timeout are not a strict overall deadline. Cancel with Ctrl+C. `store=False` does not describe all service-side retention.

Responses is generally recommended for new applications. This harness deliberately uses supported [Chat Completions filter fields](https://learn.microsoft.com/azure/foundry/openai/how-to/reasoning) through the [v1 API](https://learn.microsoft.com/azure/foundry/openai/api-version-lifecycle).

## Interpret Results

| Interpretation | Meaning |
|---|---|
| `FILTER_ENFORCEMENT_REPORTED` | A blocking field, content-filter finish reason or documented filter error was returned. Inspect the intervention point. |
| `DETECTION_REPORTED_NOT_A_BLOCK` | Detection was reported, without blocking evidence. |
| `ANNOTATIONS_PRESENT_NO_BLOCK_REPORTED` | Annotations exist; not proof every check succeeded. |
| `NO_FILTER_EVIDENCE_RETURNED` | No evidence returned; not proof filtering passed or was disabled. |
| `REQUEST_FAILED_NOT_FILTER_EVIDENCE` | A non-filter failure such as authorization or quota. |
| `AUTHENTICATION_UNAVAILABLE` / `CONNECTION_OR_TIMEOUT` | Check CLI identity, permissions, network and deployment availability. |

A correct answer or refusal alone is not enforcement evidence. Inspect `annotation_error_present`; `finish_reason: length` indicates the token cap, not blocking. Exit zero means an observable response/filter outcome, not a passed blocking test. Offline Groundedness is not runtime enforcement. If neither case triggers a control, report that outcome rather than weakening protections.

Append `--save` to write a new timestamped capture under ignored `demo/evidence/`. It contains selected annotations, synthetic response, time and dataset hash, not raw headers or raw exceptions. This is not a general secret scrubber: review changed inputs and outputs before sharing. Keep sanitized policy-assignment evidence separately.

## Evaluate the Fixtures

The portal evaluates **existing hand-authored responses**, not measured GPT-5 outputs. The two harness calls do not populate or replace the ten fixtures.

1. Open **Evaluation**, choose target **Dataset** and scope **Individual turns**. A Model/Agent target is a different workflow.
2. Upload [demo/cases.jsonl](../demo/cases.jsonl) and select its version. Map `query`, `response`, `context` and `ground_truth` to their corresponding inputs. Groundedness needs context. Retain `case_id` and `origin` for provenance.
3. Select **Relevance** and **Groundedness**, with a supported judge deployment, quota and permissions. Inspect input requirements for any additional metrics.
4. Review mappings/criteria and submit. Dataset evaluation does not regenerate target outputs, but judge calls are billable.
5. Wait for **Completed**; inspect metric errors, row-level scores and reasons. `Partial`/`Failed` is not full success.
6. Compare case-01 (supported 12 months), case-02 (unsupported 36 months), and case-04 (context-supported but irrelevant). Cases 05/06 contrast abstention with invented information. Record actual outcomes; fixture intent does not guarantee scores.

Relevance measures relationship to the query; Groundedness measures support in supplied context, not universal truth. This small synthetic dataset is not a production benchmark. You own representativeness, reference quality, thresholds and release decisions. See [portal evaluations](https://learn.microsoft.com/azure/foundry/how-to/evaluate-generative-ai-app) and [dataset requirements](https://learn.microsoft.com/azure/foundry/observability/how-to/cloud-evaluation-datasets).

## Public Data Hygiene

[.gitignore](../.gitignore) excludes local environments, non-example environment files, caches and generated evidence. Never commit credentials, private resource identifiers/URLs, customer data or unsanitized exports. Ignoring a file does not remove existing Git history; revoke exposed credentials before history remediation.