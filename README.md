# video-gen

Generate short videos with AI models, and keep every generation reproducible:
the prompt, the model, its settings, the cost and the output, tracked and
cached so a result can be compared, repeated or thrown away on purpose.

> **Status: M0, docs only.** This commit holds the README, `AGENTS.md`, the
> licence and `.gitignore`; there is no code yet. Scope and the first build are
> set in the M1 spec, `ai_specs/s01_*.md`, on a branch off `main`.

## Roadmap

| Milestone | Builds | Done when |
|---|---|---|
| **M0** | README, `AGENTS.md`, licence, public repo | this commit |
| **M1** | the spec: what gets generated, from what inputs, with which models, and how a good clip is judged | `ai_specs/s01_*.md` is merged |

Later milestones are added by the M1 spec. Each one starts with a short spec in
`ai_specs/` and lands through a pull request.

## Working on this repo

- **`AGENTS.md` is the one instruction file for coding agents**, whatever the
  harness. Read it before changing anything. There are no tool-specific copies.
- Branch off `main` for every piece of work; merge through a pull request.
- Python, managed with `uv`; Ruff for lint and format; mypy strict; pytest.
- Experiment tracking goes to the shared MLflow server in the parent
  `nmp-ai-portfolio` workspace (`MLFLOW_TRACKING_URI`). Outside that workspace,
  point the variable at any MLflow 3 server.
- Keys live in `.env` (gitignored) or `~/.env`, never in the repo.
- Generated media stays out of git. Only small, curated samples are committed.

## Licence

MIT, see `LICENSE`. Media produced with third-party models is also subject to
that model provider's terms.
