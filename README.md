# ReproBoundary website

This repository publishes the ReproBoundary benchmark website at
<https://reproboundary.github.io/>.

[Website](https://reproboundary.github.io/) ·
[Benchmark code](https://github.com/ReproBoundary/ReproBoundary) ·
[Execution traces](https://huggingface.co/datasets/SupingSun/ReproBoundary-Execution-Trajectories) ·
[Submit a run](SUBMIT_A_RUN.md)

The site presents the paper abstract and results, original construction figure,
failure analysis, 47-task explorer, and compact submission instructions. The
benchmark runner, frozen task contracts, and evaluators are maintained in the
[main repository](https://github.com/ReproBoundary/ReproBoundary); large paper
and trajectory files remain on Hugging Face.

## Repository contents

```text
ReproBoundary.github.io/
├── index.html
├── app.js
├── styles.css
├── assets/
├── data/
│   ├── leaderboard.json
│   └── tasks.json
└── SUBMIT_A_RUN.md
```

Use [`SUBMIT_A_RUN.md`](SUBMIT_A_RUN.md) for the package layout, field
reference, generated-file example, validation command, and delivery details.
