# Submit a ReproBoundary run

Run the frozen evaluator, then let the released pipeline package the result.
Do not manually reorganize the six files required by `TARGET_EVALUATION.json`.

## Compact package

The submitted archive has three required entries and one optional entry:

```text
<project-id>-<model>-<agent>/
├── submission.json   # metadata and the six contract-required deliverables
├── artifacts/        # referenced predictions, evaluator output, and intermediates
├── agent.log.gz      # compressed execution trace
└── source.patch      # included only when the source changed
```

`submission.json` embeds `DATASET_SELECTION.json`, `pipeline-execution.json`,
`reproduction-report.md`, `audit.json`, `metrics.json`, and
`security-attestation.json` without changing their schemas. The released task
contract is identified by project ID and SHA-256 instead of being copied into
the package.

## `submission.json` fields

The packer creates this file. Submitters should inspect it, not write a second
metadata file.

| Field | Type | Meaning |
| --- | --- | --- |
| `schema_version` | string | Compact-package schema; currently `1.0`. |
| `benchmark` | string | Must be `ReproBoundary`. |
| `project_id` | string | Exact ID from `TASK_INVENTORY_47.csv`. |
| `model` | string | Short model label used in the archive name. |
| `agent` | string | Short agent label used in the archive name. |
| `source_commit` | string | Frozen source commit from the selected task. |
| `target_contract_sha256` | string | SHA-256 of the exact released `TARGET_EVALUATION.json`. |
| `claim` | string | `complete_reproduction`, `partial_result`, or `no_metric`; derived from `metrics.json`. |
| `artifact_root` | string | Always `artifacts`; all evidence paths resolve below it. |
| `deliverables` | object | The six contract-required outputs described below. |
| `files` | array | Package manifest. Each entry contains relative `path`, integer `bytes`, and lowercase `sha256`. |

`model` and `agent` may contain letters, digits, `.`, `_`, and `-`, must start
with a letter or digit, and may be at most 80 characters.

## Embedded deliverables

All JSON deliverables repeat `schema_version`, `project_id`, and
`target_contract_sha256`. Their `schema_version` remains `4.0`; the compact
package does not replace the task contract.

### `DATASET_SELECTION.json`

| Field | Meaning |
| --- | --- |
| `target_metric_id` | Exact primary metric identifier from the contract. |
| `reduction_protocol_id` | Frozen reduced-reproduction protocol identifier. |
| `data_scopes` | Exact dataset, version, split, selection rule, sample counts, IDs, hashes, and leakage constraints from the contract. |
| `stage_budgets` | Exact per-stage resource and workload settings from the contract. |
| `no_unplanned_reductions` | Must be `true`; an infeasible run is reported as incomplete rather than silently reduced. |

### `pipeline-execution.json`

| Field | Meaning |
| --- | --- |
| `reduction_protocol_id` | Must match `DATASET_SELECTION.json`. |
| `pipeline_status` | `pipeline_complete`, `pipeline_invalid`, or `pipeline_indeterminate`. |
| `stages` | One record for every required stage, in contract order. |
| `actual_data_scopes` | Data scopes actually used; must equal the frozen scopes for a valid submission. |
| `actual_stage_budgets` | Stage settings actually used; must equal the frozen settings for a valid submission. |
| `deviations` | Disclosed protocol-deviation identifiers; use only values allowed by the task contract. |

Each `stages` entry contains `stage_id`, `status`, `commands`, `exit_codes`,
`started_at`, `finished_at`, `input_artifacts`, `output_artifacts`, and
`evidence_paths`. `status` is one of `completed`, `failed`, `timed_out`, or
`not_started`. Commands and exit codes are parallel arrays; artifact and
evidence values are safe relative paths.

### `reproduction-report.md`

UTF-8 Markdown describing the executed work, code changes, observed results,
completed scope, failures, and limitations. This is text, not a JSON object.

### `audit.json`

Contracts with an `audit.json` template use `events` for dependency resolutions
and protocol-preserving implementation fixes, and `notes` for additional audit
context. Legacy contracts without that template define only the three common
identity fields; preserve their task-produced audit content as-is. Protocol
deviations belong in `pipeline-execution.json`, not here.

### `metrics.json`

| Field | Meaning |
| --- | --- |
| `target_metric_id` | Exact primary metric identifier. |
| `target_metric_obtained` | Boolean stating whether the frozen target metric was actually computed. |
| `reproduction_status` | `completed_reproduction`, `metric_obtained_pipeline_invalid`, `partial_pipeline`, `timed_out_partial`, `timed_out_no_result`, `failed`, or `blocked_external`. |
| `comparability_status` | `comparable`, `invalid_protocol`, `insufficient_coverage`, or `no_metric`. |
| `pipeline_status` | Must match `pipeline-execution.json`. |
| `target_protocol_deviations` | Protocol deviations affecting the target result. |
| `evaluation_scope` | Dataset, version, split, languages, selection, and sample-count scope actually evaluated. |
| `metrics` | Exactly one primary metric entry when `target_metric_obtained` is `true`; otherwise an empty array. |
| `auxiliary_metrics` | Optional additional results; these never replace the target metric. |
| `no_metric_reason` | Required non-empty explanation when `target_metric_obtained` is `false`. |

The primary `metrics` entry contains `metric_id`, `name`, `value`, `unit`,
`dataset`, `dataset_version`, `split`, `languages`, `sample_count`, `evaluator`,
`evaluator_version`, and `evidence_path`.

### `security-attestation.json`

| Field | Meaning |
| --- | --- |
| `security_policy_id` | Exact policy identifier copied from the task contract. |
| `security_policy_sha256` | Exact policy hash copied from the task contract. |
| `declared_events` | Array of known security events; each event records `event_type`, `outcome`, `description`, and `evidence_paths`. |
| `no_undeclared_security_events` | Boolean attestation; it is not independent proof that the run was safe. |
| `notes` | String; use an empty string when there is no note. |

## Other package entries

- `artifacts/` preserves every path referenced by selected-ID manifests, stage
  outputs, evaluator evidence, metrics, and security events. It may contain
  nested directories, but no absolute paths, `..`, or symlinks.
- `agent.log.gz` is one gzip-compressed execution trace.
- `source.patch` is a Git binary patch generated only when tracked or new
  source files changed; generated results are excluded.

## Pack and check

From the ReproBoundary repository:

```bash
python3 pipeline.py pack-submission --run-dir "runs/$PROJECT_ID" --model "$MODEL" --agent "$AGENT" --output "$PROJECT_ID-$MODEL-$AGENT.tar.gz"
python3 pipeline.py validate-submission "$PROJECT_ID-$MODEL-$AGENT.tar.gz"
```

The packer validates all six deliverables, copies only their referenced
artifacts, compresses the agent log, records file hashes, and adds
`source.patch` when the source tree changed. It rejects missing files, unsafe
paths, symlinks, contract-hash mismatches, and invalid gzip or file hashes.

Before uploading, inspect the package for credentials, authentication files,
private host paths, datasets, model weights, checkpoints, and unrelated files.
Do not alter evaluator output or discard evidence of failed stages.

## Send the archive

Upload the single `.tar.gz` file to a stable download link. Email
`endlesssalaxy@outlook.com` with subject
`ReproBoundary submission: <project-id> / <model> / <agent>` and include only:

- the archive download URL;
- the SHA-256 printed by `pack-submission` or `validate-submission`.

The project ID, model, agent, source commit, run claim, contract hash, and file
hashes are already recorded in `submission.json`.
