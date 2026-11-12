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
