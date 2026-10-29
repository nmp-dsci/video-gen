# AGENTS.md — video-gen

## One instruction file

This is the only agent-instruction file in the repo. Every coding agent and
harness reads this file and nothing else. Do not create tool-specific
instruction files, copies or variants. If a harness does not load `AGENTS.md`
on its own, point it at this file; never fork the content. Subdirectories may
add their own `AGENTS.md` for module-specific rules, and the nearest one wins.

## What this is

A project for generating short videos with AI models, where every generation
is tracked and cached: prompt, inputs, model, settings, seed, cost and output.
`README.md` has the roadmap.

**Status: M0, docs only.** No code exists yet, and the scope is not fixed. The
M1 spec decides what gets generated and how quality is judged. Follow the
layout below until real code replaces it, then follow the code.

## Vocabulary

| Term | Meaning |
|---|---|
| **job** | one request to a generation model: prompt, input assets, model id, settings, seed |
| **generation** | the result of a job: the output file, its metadata, cost and latency |
| **provider** | where a model runs: a hosted API, or an open model on rented GPUs |
| **cache key** | a hash of everything that determines a job's output; the same key never pays twice |

## Hard rules

1. **Every generation is reproducible from its record.** A job logs its
   prompt, inputs, model id and version, settings and seed before it runs.
   A clip whose record is missing is not a result.
2. **Cache every paid call.** Generation is slow and billed per clip. Outputs
   are cached on disk under the job's cache key, and a re-run with the same
   key costs nothing.
3. **Know the cost before you spend it.** Batch runs print an estimated cost
   and job count first. Nothing loops over a paid model without a cap.
4. **One module for model calls.** Every generation call goes through one
   module, so provider choice, retries, caching, cost and tracing live in one
   place.
5. **Public repo hygiene.** Never commit keys, `.env`, caches or bulk
   generated media. Committed: code, specs, prompts, configs and a few small,
   curated sample outputs.
6. **Rights.** Do not commit input images, video or audio unless they are ours
   or licensed for redistribution. Do not generate a real person's likeness
   without their consent.

## Workflow

- `main` holds reviewed work only. Branch for every change, merge by pull
  request.
- Conventional commits: `feat(gen): …`, `fix(cache): …`, `docs: …`.
- Every milestone starts with a spec, `ai_specs/sNN_<topic>.md`, holding the
  decisions and the reasoning behind them. Read the latest spec before a
  change that touches its area.
- Review artifacts (plans, result pages, side-by-side comparisons) go in
  `.lavish/sNN_<topic>.html`. `.lavish/` is never gitignored.
- Update `README.md` whenever commands, setup or behaviour change.

## Conventions

- Python with `uv` for everything (`uv sync`, `uv add`, `uv run`). No raw `pip`.
- `ruff format .` and `ruff check . --fix`; mypy strict.
- pytest under `tests/`. Tests are deterministic and offline: mock every model
  and provider call, and use small committed fixtures.
- Secrets come from the environment (`./.env`, then `~/.env`), never from code
  or config files.

## Shared platform

In the `nmp-ai-portfolio` workspace this repo uses the shared services in
`../nmp-central-ai`. Read `../nmp-central-ai/PLATFORM.md` before touching
tracking, storage or deployment.

- **MLflow** via `MLFLOW_TRACKING_URI` only. Never start a local server or fall
  back to a `sqlite:` or `file:` store. Experiments are named
  `video-gen/<purpose>` (e.g. `video-gen/generate`, `video-gen/eval`). Never
  hardcode an experiment id. Log a generation's output as an artifact only
  when it is small; otherwise log its path and cache key.
- **Not yet onboarded.** Before the first tracked run, add this project to
  `../nmp-central-ai/registry/projects.yaml` and follow
  `../nmp-central-ai/docs/onboarding.md` (a zero-cost `smoke` command included).
- **House UI** for `.lavish/` pages: `make -C ../nmp-central-ai install-house-ui`
  once the project is in the registry. Never write your own design tokens.
- **Postgres**, if the project needs it, is one database on the shared server,
  declared in the registry. Never add a database service to this repo.
- Outside the workspace, point `MLFLOW_TRACKING_URI` at any MLflow 3 server.

## Prior art in the workspace

Reuse before rebuilding. Decide per item, in the M1 spec, whether to depend on
it, copy it, or write it fresh.

- **`../ConvFinQA-agent`**: the single model-call module pattern and MLflow
  tracking helpers.
- **`../tau-telecom-sft`**: running open models on rented GPUs (Modal), with
  its skill notes under `.claude/skills/modal/`.

## Expected layout

Guidance until code exists:

```text
src/video_gen/
  config.py          settings; reads ./.env then ~/.env
  providers/         the one module every generation call goes through, one adapter per provider
  jobs.py            job schema, cache key, cost estimate
  cache.py           on-disk output cache
  cli.py             one entry point per stage
prompts/             committed prompts and job configs
assets/samples/      small curated outputs, committed on purpose
outputs/             everything else generated; gitignored
ai_specs/            sNN_*.md specs, the plan of record
.lavish/             review artifacts
tests/
```

## Open decisions (for the M1 spec)

- What the project generates, from what inputs, and for whom.
- Providers and models: hosted APIs, open models on rented GPUs, or both.
- How a good clip is judged: human review, model-as-judge, automatic metrics.
- Budget per run and per month, and where billing goes.
- Where outputs live beyond the local cache.
