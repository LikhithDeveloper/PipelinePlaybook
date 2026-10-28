# Pipeline Playbook

CI/CD, YAML & GitHub Actions — zero to advanced

Start here — no experience needed

## What is CI/CD?

CI/CD is the practice of automating the steps between "I wrote some code" and "it's running for users" — building, testing, and shipping it without doing those steps by hand each time. This guide takes you from this idea all the way to writing production pipelines. Read the tabs in order; don't skip.

### The problem it solves

Imagine a team of five developers all changing the same project. Without automation, every time someone wants to ship a change they'd have to manually pull everyone's code together, run all the tests on their own laptop, build the app, copy files to a server, and hope nothing broke. This is slow, error-prone, and gets worse as the team grows — bugs are found late, "it works on my machine" becomes a daily phrase, and releases become scary events that happen once a month.

CI/CD replaces that manual ritual with an automated assembly line. Every time anyone changes the code, a machine automatically checks it, tests it, and (when you want) ships it — the same way, every time.

### The pipeline: an assembly line for code

A **pipeline** is a sequence of automated stages. Code enters at one end, and each stage does one job before passing it to the next — exactly like a factory line. If any stage fails, the line stops and nobody downstream is affected.

Code push→ Build→ Test→ Scan→ Package→ Deploy

That's the whole idea. Everything else in this guide is detail about how to describe those stages and make a real tool run them for you.

### CI and CD are two halves

| Term | What it automates | Who presses "go" |
| --- | --- | --- |
| **CI** — Continuous Integration | Every push builds the code and runs tests automatically, so broken changes are caught in minutes | Nobody — automatic |
| **CD** — Continuous Delivery | Every passing build is packaged and made *ready* to release at the click of a button | A human clicks "release" |
| **CD** — Continuous Deployment | Every passing build goes *all the way to production* with no human step | Nobody — automatic |

In plain words **CI** = "is this change good?" (build + test on every push). **CD** = "ship the good changes" (package + release). Delivery stops at a human approval; Deployment removes even that.

### Why it matters

#### Catch bugs early

- Tests run on every change, so problems surface in minutes — while the author still remembers the code — not weeks later.

#### Ship faster & safer

- Releasing becomes a boring, routine, automated event instead of a risky monthly scramble.

#### Consistency

- The same steps run the same way every time. No more "it works on my machine."

#### Easy rollback

- Every release is a versioned artifact, so undoing a bad deploy is just redeploying the previous one.

### Where YAML and GitHub Actions fit in

To make a tool run your pipeline, you have to *describe* the stages somewhere. You write that description in a file using a format called **YAML**, and a tool called **GitHub Actions** reads it and runs the stages for you on GitHub's machines. So there are two things to learn, and keeping them separate is the whole game:

#### Layer 1 — YAML

- A general text format for structured data
- Rules about colons, dashes and indentation
- The same everywhere: GitHub, GitLab, Docker, Kubernetes

#### Layer 2 — GitHub Actions keywords

- Words like `jobs`, `steps`, `runs-on`
- Defined by GitHub Actions, not by YAML
- GitLab, Jenkins etc. use different words for the same ideas

The one idea to hold onto YAML is the **grammar**; GitHub's keywords are the **vocabulary**. A file can be perfect YAML and still be rejected by GitHub if it uses words GitHub doesn't know. Learn the grammar once, then learn each tool's dictionary.

### Words you'll meet (mini-glossary)

| Term | Plain meaning |
| --- | --- |
| **Pipeline** | The whole automated sequence of stages |
| **Workflow** | GitHub's word for a pipeline — one YAML file |
| **Job** | A group of steps that run on one machine |
| **Step** | A single action or command inside a job |
| **Runner** | The machine that executes the steps |
| **Action** | A reusable, prebuilt step someone published |
| **Trigger** | The event that starts a workflow (a push, a PR…) |
| **Artifact** | A file the pipeline produces (a build, a report) |

### Your path through this guide

1. **01 YAML syntax** — learn the file format first; everything is written in it.
2. **02 GitHub Actions** — the keywords and how a workflow is structured.
3. **03 Real workflows** — complete pipelines you can copy and run.
4. **04 Cheat sheet** — keep it open as a quick reference.
5. **05 Beyond the basics** — artifacts, caching, reusable workflows, and more.
6. **06 Interview Q&A** — 28 questions to test yourself.
7. **07 Enterprise patterns** — how large companies run CI/CD at scale.
8. **08 Practicals** — hands-on projects and real-world recipes to build.
9. **09 Everything** — every official doc and course to go further.

New to all this? Don't try to memorize. Read 01 and 02 slowly, then open the **08 Everything** tab, pick one free course, and make a tiny test repo. You'll learn more from one workflow that actually runs than from reading ten.

01 — The grammar

## YAML syntax

YAML stores data as a mix of key-value pairs, lists, and nesting. Structure is shown entirely by indentation — there are no braces or brackets holding things together.

### Rule zero: indentation is the structure

Two spaces per level is the convention. There's one rule you can never break:

Hard rule Use **spaces, never tabs**. A single tab character is a syntax error in YAML. Set your editor to insert spaces when you press Tab.

### The three building blocks

building-blocks.yml

```
# 1 — Key-value pair (a "mapping"). Note the space after the colon.
name: CI
runs-on: ubuntu-latest

# 2 — List (a "sequence"). Each item starts with a dash.
branches:
  - main
  - develop

# 3 — Nesting. Indent to show something lives inside something.
jobs:
  test:
    runs-on: ubuntu-latest
```

In that last block, `test` lives under `jobs`, and `runs-on` lives under `test` — purely because of how far they're indented.

### The pattern you'll use constantly: a list of mappings

Build steps are a *list* where each item is itself a *mapping*. The dash sits at the item level; the keys indent underneath it.

steps.yml

```
steps:
  - name: Check out code     # item 1 starts
    uses: actions/checkout@v4  # same item (aligned with name)
  - name: Install deps      # item 2 starts
    run: pip install -r requirements.txt
```

Each `-` begins a new step. The keys `name` and `uses` belong to the same step *because they're indented equally*, and they attach to the dash above them because they're indented past it.

### The colon needs a space

The single most common beginner error: a key-value pair must have a space after the colon.

broken

```
version:3.12
# read as one string "version:3.12"
```

correct

```
version: 3.12
# key "version", value 3.12
```

### Watch the type guessing

YAML guesses whether a value is a number, string, or boolean. That occasionally bites you.

types.yml

```
version: 3.12        # a float → becomes 3.12 (the trailing zero is lost!)
version: "3.12"      # a string → stays exactly "3.12"
retries: 3          # an integer
enabled: true      # a boolean
region: "on"        # QUOTE this — bare on/off/yes/no read as booleans
message: "Build: done" # QUOTE when the value contains a colon
```

Rule of thumb Quote anything that could be misread: version numbers, values with colons, leading zeros, and the words `on/off/yes/no`. This is exactly why you see Python versions written as `"3.12"` with quotes.

### Long values and multiple lines

multiline.yml

```
run: |                 # "|" keeps the newlines — each line runs separately
  pip install -r requirements.txt
  ruff check .
  pytest

description: >         # ">" folds everything into a single line
  this becomes one
  long sentence
```

### Block style vs flow style

The same list can be written two ways. Both are valid — pick whichever reads better on the line.

block style

```
python-version:
  - "3.11"
  - "3.12"
```

flow style (JSON-like)

```
python-version: ["3.11", "3.12"]
```

Save yourself hours Before committing, run a YAML linter — the `yamllint` CLI, or the YAML / GitHub Actions extension in VS Code. It flags indentation errors instantly, so you don't push a broken file and wait for the pipeline to fail just to find a misplaced space.

02 — The vocabulary

## GitHub Actions syntax

GitHub Actions gives meaning to a fixed set of YAML keys. Learn this small vocabulary and the file structure falls into place. Workflow files live in `.github/workflows/` in your repo.

### The anatomy of a workflow

.github/workflows/ci.yml

```
name: CI                        # label shown in the Actions tab

on:                             # WHEN the workflow runs (the trigger)
  push:
    branches: [main]
  pull_request:

jobs:                           # WHAT runs — one or more jobs
  test:                          # job id (you choose this name)
    runs-on: ubuntu-latest       # which machine (runner) to use
    steps:                       # ordered list of actions in this job
      - uses: actions/checkout@v4  # run a prebuilt action
      - run: echo "Hello pipeline" # run a shell command
```

### The core keywords

| Keyword | Means | Lives under |
| --- | --- | --- |
| `name` | Display label for the workflow | top level |
| `on` | The event(s) that trigger it | top level |
| `jobs` | Container for all jobs | top level |
| `runs-on` | The runner OS (ubuntu/windows/macos) | a job |
| `steps` | Ordered list of things to do | a job |
| `uses` | Run a reusable published action | a step |
| `run` | Run a shell command | a step |
| `with` | Inputs passed to a `uses` action | a step |
| `env` | Environment variables | step / job / top |
| `needs` | Wait for another job to finish first | a job |
| `if` | Run only when a condition is true | job or step |
| `strategy` / `matrix` | Run the same job across many versions | a job |

YAML-valid ≠ Actions-valid A step only accepts GitHub's known keys (`name`, `uses`, `run`, `with`, `env`, `if`, `id`…). Invented keys like `x1` are perfect YAML but GitHub ignores or rejects them. To pass custom values into an action, use `with:`, which takes arbitrary keys.

### uses vs run — the two kinds of step

uses — a prebuilt action

```
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.12"
```

run — your own commands

```
- name: Run tests
  run: |
    pip install -r requirements.txt
    pytest
```

`uses` pulls in a packaged action someone published (always pin a version like `@v4`). `run` executes shell commands directly on the runner.

### Triggers — the `on` key

triggers.yml

```
on:
  push:
    branches: [main, develop]   # on push to these branches
    tags: ["v*"]                 # or when a version tag is pushed
  pull_request:                      # on any PR
  schedule:
    - cron: "0 2 * * *"            # daily at 02:00 UTC
  workflow_dispatch:                 # a manual "Run" button
```

### Variables & secrets — GitHub's expression syntax

This is *not* YAML — it's GitHub's own `${{ }}` syntax, interpreted at runtime. YAML just carries it as a string.

secrets.yml

```
- name: Deploy
  run: ./deploy.sh
  env:
    API_KEY: ${{ secrets.DEPLOY_KEY }}   # from repo Settings → Secrets
    BRANCH:  ${{ github.ref_name }}      # built-in context variable
```

Security basics Secrets are stored in **Settings → Secrets and variables** and injected at runtime — **never commit them to the repo**. Reference them only through `${{ secrets.NAME }}`.

### Matrix builds — test many versions at once

matrix.yml

```
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
```

This single job runs three times in parallel — once per Python version. If any one fails, that cell shows red.

03 — Putting it together

## Real-world workflows

Complete, copy-ready pipelines you'd actually commit. Each is annotated so you can see where YAML ends and GitHub's keywords begin.

### ① A full Python CI + deploy pipeline

Runs tests on every PR; deploys only when `main` changes and tests pass.

.github/workflows/python-ci.yml

```
name: Python CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4          # get the code onto the runner
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip                      # reuse installed deps between runs
      - run: pip install -r requirements.txt
      - run: ruff check .                  # fast lint first (fail cheap)
      - run: pytest --cov=src --cov-fail-under=80

  deploy:
    needs: test                          # wait for "test" to pass
    if: github.ref == 'refs/heads/main'  # only on main, not on PRs
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t myapp:${{ github.sha }} .
      - run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.DEPLOY_KEY }}
```

What to notice `needs: test` chains the jobs. The `if:` means PRs get tested but only `main` deploys. The image is tagged with `github.sha` so every deploy is traceable to the exact commit.

### ② A Node.js build with caching

.github/workflows/node.yml

```
name: Node CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
      - run: npm ci                     # clean, reproducible install
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

### ③ Build & push a Docker image to a registry

.github/workflows/docker.yml

```
name: Docker publish
on:
  push:
    tags: ["v*"]                   # only on version tags

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      packages: write               # allow pushing to GHCR
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

Handy built-in `secrets.GITHUB_TOKEN` is created automatically for every run — you don't have to set it. It's scoped to the current repo, which is why the `permissions` block grants it `packages: write`.

04 — Quick reference

## Cheat sheet

Everything above, condensed. Keep this section open while you write your first few workflows.

#### YAML gotchas

- Spaces only — `never tabs`
- Always a space after `:`
- 2 spaces per indent level
- Quote `on/off/yes/no`
- Quote versions: `"3.12"`
- Quote values with a `:` inside
- `#` starts a comment

#### Structure keywords

- `name` — workflow label
- `on` — trigger
- `jobs` — the work
- `runs-on` — the machine
- `steps` — ordered list

#### Step keywords

- `uses` — prebuilt action
- `run` — shell command
- `with` — inputs to an action
- `env` — env variables
- `if` — conditional
- `id` — name for referencing

#### Flow control

- `needs` — run after another job
- `if` — run only when true
- `strategy.matrix` — many versions
- `continue-on-error` — don't fail the run

#### Multiline strings

- `|` — keep newlines (block)
- `>` — fold into one line
- `|-` — keep newlines, strip trailing

#### Common contexts

- `secrets.NAME` — a secret
- `github.sha` — commit hash
- `github.ref_name` — branch/tag
- `github.actor` — who triggered it
- `matrix.x` — current matrix value

#### Pipeline best practice

- Cheap stages first (lint before tests)
- Build the artifact once; promote it
- Pin action versions (`@v4`)
- Never commit secrets
- Keep the workflow file in the repo

#### Runners

- `ubuntu-latest` — most common
- `windows-latest`
- `macos-latest`
- self-hosted — your own machine

05 — Job-ready

## Beyond the basics

Everything between "my first workflow runs" and "I can run production pipelines." Pick a sub-tab. Work through them in order the first time — each builds toward being genuinely interview-ready.

### Artifacts — pass files between jobs

Each job runs on a fresh machine, so files don't carry over automatically. Upload an artifact in one job, download it in another.

artifacts.yml

```
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build      # produces ./dist
      - uses: actions/upload-artifact@v4
        with:
          name: site                     # a label to download by
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: site                     # same label → ./dist reappears
      - run: ./publish.sh dist/
```

Artifact vs cache An **artifact** is a build output you want to keep or move between jobs (losing it loses a result). A **cache** is just a speed optimization that can always be rebuilt. Don't use one for the other's job.

### Caching — stop reinstalling everything

The language setup actions have caching built in (`cache: pip`), but `actions/cache` gives you full control with a custom key.

cache.yml

```
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip                 # what to cache
    # key changes only when the lockfile changes → cache rebuilds then
    key: pip-${{ hashFiles('requirements.txt') }}
    restore-keys: pip-              # fall back to any older pip- cache
```

The `hashFiles()` function builds a key from your lockfile's contents. Change a dependency and the hash changes, so the cache is rebuilt; otherwise the old cache is restored instantly.

### Job outputs — pass values (not files) between jobs

Use `id` on a step, write to `$GITHUB_OUTPUT`, expose it as a job output, then read it in a later job.

outputs.yml

```
jobs:
  version:
    runs-on: ubuntu-latest
    outputs:
      tag: ${{ steps.v.outputs.tag }}     # expose the step's output
    steps:
      - id: v                            # give the step an id
        run: echo "tag=v1.2.3" >> "$GITHUB_OUTPUT"

  release:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Releasing ${{ needs.version.outputs.tag }}"
```

### Reusable workflows — call one workflow from another

Define a workflow with `workflow_call`, then call it from others. This kills copy-paste across many repos or many pipelines.

reusable: tests.yml

```
on:
  workflow_call:                  # makes it callable
    inputs:
      python-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
      - run: pytest
```

caller: ci.yml

```
on: [push]

jobs:
  run-tests:
    # point at the reusable file
    uses: ./.github/workflows/tests.yml
    with:
      python-version: "3.12"
    # pass secrets explicitly if needed:
    # secrets: inherit
```

Reusable vs composite A **reusable workflow** is a whole workflow (its own jobs) you call with `uses` at the *job* level. A **composite action** bundles several *steps* into one action you call at the *step* level — see below.

### Composite actions — bundle repeated steps

When the same 3–4 steps repeat across jobs, package them as a composite action in `.github/actions/setup/action.yml`.

.github/actions/setup/action.yml

```
name: Setup project
runs:
  using: composite               # marks it as a composite action
  steps:
    - uses: actions/setup-python@v5
      with: { python-version: "3.12" }
    - run: pip install -r requirements.txt
      shell: bash                   # composite run steps must name a shell
```

using it in any job

```
steps:
  - uses: actions/checkout@v4
  - uses: ./.github/actions/setup      # your local composite action
  - run: pytest
```

### Concurrency — cancel superseded runs

Without this, pushing twice quickly runs the pipeline twice. This cancels the older run when a new one starts on the same branch.

concurrency.yml

```
concurrency:
  group: ci-${{ github.ref }}        # one group per branch
  cancel-in-progress: true           # kill the older run
```

### Manual runs with inputs — `workflow_dispatch`

manual.yml

```
on:
  workflow_dispatch:                 # shows a "Run workflow" button
    inputs:
      environment:
        description: "Where to deploy"
        type: choice
        options: [staging, production]
        default: staging

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh ${{ inputs.environment }}
```

### Environments & approvals

Attach a job to a named environment to get required reviewers, wait timers, and environment-specific secrets. This is how Continuous Delivery enforces its "human approves production" gate.

environment.yml

```
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production          # reviewers set in repo settings must approve
    steps:
      - run: ./deploy.sh
        env:
          KEY: ${{ secrets.PROD_KEY }}        # scoped to the "production" environment
```

### More knobs worth knowing

| Keyword | What it does |
| --- | --- |
| `timeout-minutes` | Kill a job/step that hangs past this limit (default 360) |
| `continue-on-error` | Let a step fail without failing the whole job |
| `paths` / `paths-ignore` | Only trigger when certain files change |
| `permissions` | Narrow the `GITHUB_TOKEN`'s scope (least privilege) |
| `services` | Spin up sidecar containers (Postgres, Redis) for tests |
| `fail-fast: false` | In a matrix, keep other cells running when one fails |
| `defaults.run` | Set a default shell / working directory for all run steps |

service-container.yml

```
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:                      # a throwaway DB for the test run
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
        ports: ["5432:5432"]
    steps:
      - uses: actions/checkout@v4
      - run: pytest   # connects to localhost:5432
```

### Status badge

Drop this in your README to show the pipeline's pass/fail state:

README.md

```
![CI](https://github.com/OWNER/REPO/actions/workflows/ci.yml/badge.svg)
```

### YAML power-features you'll see in real repos

Anchors (`&`) name a block; aliases (`*`) reuse it; the merge key (`<<`) pulls one mapping's keys into another. They remove repetition.

anchors.yml

```
defaults: &defaults            # &defaults names this block
  runs-on: ubuntu-latest
  timeout-minutes: 10

job-a:
  <<: *defaults                 # merge in everything from &defaults
  name: A

job-b:
  <<: *defaults
  name: B                       # both jobs inherit runs-on + timeout
```

One caveat GitHub Actions supports standard YAML anchors in a workflow file, but **not** across files, and some other CI tools disable them. They're great for cutting repetition — just don't over-use them, since they can make a file harder to read for beginners.

### Multi-document & nulls (round-out)

misc.yml

```
name: doc one
---                           # "---" separates multiple docs in one file
name: doc two

empty: null                  # explicit null (also: ~ or just blank)
blank:                        # this is also null
```

Next: when it breaks You can now write a real multi-job pipeline. Your first one *will* fail — so the next sub-tab, **Debugging**, is the one you'll use most in practice.

### Debugging & troubleshooting

Every engineer spends more time reading failed runs than writing new ones. Here's the toolkit, roughly in the order you'll reach for it.

### 1 · Read the run, re-run the job

In the repo's **Actions** tab, click the failed run then the failed job, and expand the red step. The error is almost always in the last few lines. You can **Re-run failed jobs** (not the whole workflow) to retry a flaky failure, and **Re-run with debug logging** for more detail.

### 2 · Turn on debug logging

GitHub hides verbose logs by default. Enable them by setting two repository *secrets* (Settings then Secrets and variables then Actions):

| Secret name | Value | Shows |
| --- | --- | --- |
| `ACTIONS_STEP_DEBUG` | `true` | Detailed per-step diagnostic output |
| `ACTIONS_RUNNER_DEBUG` | `true` | Low-level runner/job setup logs |

### 3 · Print the context when confused

Unsure what a variable holds? Dump it into the log and look.

debug-dump.yml

```
- name: Dump contexts
  run: |
    echo "event: ${{ github.event_name }}"
    echo "ref:   ${{ github.ref }}"
    echo "sha:   ${{ github.sha }}"
  env:
    FULL_CTX: ${{ toJSON(github) }}   # whole context as JSON
- run: echo "$FULL_CTX"
```

### 4 · SSH into the runner (the power move)

When logs aren't enough, drop in a step that opens a live SSH session *into the runner mid-run*, so you can inspect the actual filesystem. Add it just before the failing step.

tmate-debug.yml

```
- name: Debug over SSH
  if: failure()                       # only open a session if something failed
  uses: mxschmitt/action-tmate@v3
```

Use with care The session holds the runner open and prints a public SSH address in the log. Only use it on your own repos, and remove it when done. Never leave a tmate step in a shared or production workflow.

### 5 · Conditional functions for failure handling

| Function | Runs the step when... |
| --- | --- |
| `success()` | all previous steps passed (the default) |
| `failure()` | a previous step failed — use for alerts, debug |
| `always()` | no matter what — use for cleanup, uploading logs |
| `cancelled()` | the run was cancelled |

always-upload-logs.yml

```
- name: Upload test logs
  if: always()                        # even if tests failed, grab the report
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: reports/
```

### Common errors & what they mean

| Symptom | Usual cause |
| --- | --- |
| Workflow doesn't run at all | File not in `.github/workflows/`, or `on:` trigger doesn't match |
| `bad indentation` / parse error | Tabs instead of spaces, or misaligned keys |
| Secret is empty | Not defined in repo settings, or it's a fork PR (secrets withheld) |
| `Resource not accessible` | `GITHUB_TOKEN` lacks a permission — add a `permissions` block |
| Step passes but did nothing | Command failed silently — add `set -e` or check exit codes |

### Workflow commands & special files

Steps talk to the runner by writing to special files and printing special log lines. This is how you set variables, build summaries, and control the log.

### Set a variable for later steps

A plain `export` lasts one step. To share a value with *later* steps, append it to `$GITHUB_ENV`.

github-env.yml

```
- run: echo "VERSION=1.4.0" >> "$GITHUB_ENV"
- run: echo "Building $VERSION"       # available in all later steps
```

### Write a rich run summary

Anything you append to `$GITHUB_STEP_SUMMARY` (as Markdown) shows up as a formatted panel on the run's page — great for test results, coverage, or a deploy link.

step-summary.yml

```
- name: Publish summary
  run: |
    echo "## Build succeeded" >> "$GITHUB_STEP_SUMMARY"
    echo "- Version: $VERSION" >> "$GITHUB_STEP_SUMMARY"
    echo "- Tests: 142 passed" >> "$GITHUB_STEP_SUMMARY"
```

### Log groups, masking & annotations

log-commands.yml

```
- run: |
    echo "::group::Install step"        # collapsible log group
    npm ci
    echo "::endgroup::"

    echo "::add-mask::$MY_TOKEN"          # hide a value from the log

    echo "::warning file=app.js,line=10::Deprecated call"
    echo "::error::Build failed on the main bundle"   # annotations on the run
```

Why these matter `::warning` and `::error` annotations appear right on the pull request's "Files changed" view, so reviewers see problems inline. The step summary becomes your pipeline's dashboard. These touches separate a basic workflow from a polished one.

### The old `set-output` syntax

Deprecated — don't use Older tutorials show `::set-output name=x::value` and `::set-env`. These were removed for security reasons. The modern equivalents are `echo "x=value" >> "$GITHUB_OUTPUT"` and `$GITHUB_ENV`. If you see the old form, it's out of date.

### Security & OIDC

Pipelines hold the keys to your production systems, so they're a prime attack target. These are the practices that keep them safe, and the ones interviewers love to ask about.

### Least privilege for the token

The automatic `GITHUB_TOKEN` is powerful. Scope it down to only what a job needs.

permissions.yml

```
permissions:
  contents: read            # default everything to read-only...

jobs:
  release:
    permissions:
      contents: write        # ...then grant write only where needed
      packages: write
```

### OIDC — deploy to the cloud with no stored secrets

**The problem.** To deploy to AWS, your workflow needs AWS credentials. The old way was to create a permanent AWS access key and paste it into GitHub secrets. That key never expires, so if it ever leaks (a log, a compromised action, a screenshot) an attacker has standing access until someone notices and rotates it.

**The OIDC way.** Instead of a stored key, GitHub and AWS set up a *trust relationship* once. Then, at the moment a workflow runs, GitHub hands it a signed identity token that says "I am the `main` branch of `your-org/your-repo`." AWS checks that token against the trust it was told to expect, and if it matches, issues **temporary** credentials that expire in minutes. Nothing long-lived is ever stored anywhere.

The analogy A stored access key is like giving the pipeline a **copy of your house key** that works forever. OIDC is like a **doorman** who recognizes the pipeline by its GitHub ID badge each time and lets it in with a visitor pass that expires the same day. No permanent key to lose.

### How the handshake works

1\. Workflow asks GitHub for an ID token-> 2. Presents it to AWS STS-> 3. AWS checks the trust & claims-> 4. Temp creds (expire in \~1h)

### Setting it up in AWS — step by step

You do this **once** per AWS account. Two pieces: register GitHub as an identity provider, then create a role that trusts it.

1

**Add GitHub as an OIDC identity provider.** In the AWS Console go to *IAM → Identity providers → Add provider → OpenID Connect*. Fill in:

| Field | Value |
| --- | --- |
| Provider URL | `https://token.actions.githubusercontent.com` |
| Audience | `sts.amazonaws.com` |

This tells AWS "tokens signed by GitHub's Actions service are something I recognize."

2

**Create an IAM role with a trust policy.** Go to *IAM → Roles → Create role → Web identity*, pick the provider you just made. The key part is the *trust policy*, which says *exactly which repo and branch* may assume this role — this is what stops any other GitHub repo in the world from using your provider.

trust-policy.json

```
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
    },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
      },
      "StringLike": {
        "token.actions.githubusercontent.com:sub": "repo:your-org/your-repo:ref:refs/heads/main"
      }
    }
  }]
}
```

The `sub` condition is the whole security That `sub` line is what locks the role to your repo. Get it wrong (e.g. leave a wildcard `repo:*`) and *any* GitHub repo could assume your role. Common patterns: `...:ref:refs/heads/main` (one branch), `...:environment:production` (one environment), or `...:pull_request` (any PR).

3

**Attach a permissions policy** to the role — what it's actually allowed to do once assumed (e.g. write to one S3 bucket). Keep it least-privilege; the trust policy says *who*, this says *what*.

4

**Copy the role's ARN** (looks like `arn:aws:iam::123456789012:role/github-ci-deploy`). That's the only thing your workflow needs — and it's not a secret, so it can live in plain YAML.

### Using it in the workflow

.github/workflows/deploy.yml

```
permissions:
  id-token: write           # REQUIRED — lets the job request the OIDC token
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Get temporary AWS creds via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          # the role ARN from step 4 — not a secret, safe in plain text
          role-to-assume: arn:aws:iam::123456789012:role/github-ci-deploy
          aws-region: ap-south-1
      - name: Deploy
        run: aws s3 sync ./dist s3://my-bucket   # works — no access keys anywhere
```

What just happened No `AWS_ACCESS_KEY_ID`, no `AWS_SECRET_ACCESS_KEY` anywhere. The `configure-aws-credentials` action requested the GitHub token, handed it to AWS, got back temporary creds, and set them for the rest of the job. They vanish when the job ends. Azure and GCP work the same way with their own login actions.

### Pin actions to a commit SHA

For the highest security, pin third-party actions to a full commit SHA instead of a tag. A tag can be moved to point at malicious code; a SHA can't.

sha-pin.yml

```
- uses: actions/checkout@08c6903cd8c0fde910a37f88322edcfb5dd907a8 # v4.3.0
```

### Secret-handling rules

#### Do

- Store in repo/org secrets
- Reference via `${{ secrets.X }}`
- Scope to environments
- Rotate regularly

#### Don't

- Commit secrets to the repo
- Print them to logs
- Pass them to untrusted actions
- Expose them to fork PRs

The classic trap Secrets are **not** passed to workflows triggered by pull requests from forks, otherwise anyone could open a PR to steal them. A fork PR needing them is a deliberate, carefully-reviewed exception, not a default.

### Automation beyond CI

Actions isn't only for build-and-test. It's a general automation engine triggered by repo events or a clock.

### Scheduled jobs (cron)

scheduled.yml

```
on:
  schedule:
    - cron: "0 2 * * 1"     # 02:00 UTC every Monday
# fields: minute hour day-of-month month day-of-week
```

Good for nightly builds, weekly dependency checks, stale-issue cleanup, or scheduled reports.

### Dependabot — keep dependencies & actions current

Dependabot opens automatic pull requests when your dependencies (or pinned action versions) have updates. Configure it with a file — no workflow needed.

.github/dependabot.yml

```
version: 2
updates:
  - package-ecosystem: "github-actions"   # update your action versions
    directory: "/"
    schedule: { interval: "weekly" }
  - package-ecosystem: "pip"              # and your app's deps
    directory: "/"
    schedule: { interval: "weekly" }
```

### Notify a chat channel on failure

notify.yml

```
- name: Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v2
  with:
    webhook: ${{ secrets.SLACK_WEBHOOK }}
    webhook-type: incoming-webhook
    payload: '{"text":"CI failed on ${{ github.ref_name }}"}'
```

### Other event triggers worth knowing

| Trigger | Fires when... |
| --- | --- |
| `issues` | an issue is opened/labeled — auto-triage, auto-reply |
| `release` | a release is published — build & attach assets |
| `issue_comment` | someone comments — "/deploy" ChatOps commands |
| `workflow_run` | another workflow finishes — chain pipelines |
| `repository_dispatch` | an external system calls the API — trigger from outside |

### Runners, cost & operations

The practical side of running pipelines day to day: which machines run your jobs, and what they cost.

### Runner types

| Type | When to use |
| --- | --- |
| **GitHub-hosted** (`ubuntu-latest` etc.) | Default. Fresh VM each run, zero maintenance. Fine for most work. |
| **Larger hosted runners** | More CPU/RAM or GPU for heavy builds — paid, set in org settings. |
| **Self-hosted** | Your own hardware — for special hardware, internal network access, or heavy sustained use. |

### Cost & minutes

GitHub-hosted runners are billed in **minutes**. Public repos get generous free usage; private repos get a monthly free allowance, then pay per minute. Windows and macOS runners cost a multiple of Linux minutes, so default to `ubuntu-latest` unless you need the others.

#### Keep bills down

- Cache dependencies
- Use `concurrency` to cancel superseded runs
- Filter with `paths`
- Prefer Linux over macOS/Windows
- Set `timeout-minutes`

#### Speed = savings

- Run cheap checks first
- Parallelize independent jobs
- Shallow-clone when you can
- Cache Docker layers

### Monorepo tip: only build what changed

paths-filter.yml

```
on:
  push:
    paths:
      - "services/api/**"        # only run when the API code changes
      - ".github/workflows/**"
```

Next One more essential sub-tab: **Rollback** — what you do when a deploy goes wrong. It's the pattern that makes shipping safe.

### Rollback with tags

Deploys go wrong. The measure of a good pipeline isn't that it never fails — it's that recovering takes seconds. The whole trick is one discipline: **every release is an immutable, tagged artifact**, so "undo" simply means "point back at the previous tag." No rebuilding, no reverting source in a panic.

### Why tags make rollback trivial

When you build, you tag the artifact with something unique and permanent — usually the Git commit SHA or a version number. That tagged image is frozen forever; it never changes. So at any moment you have a shelf of past releases, each labelled, each ready to redeploy.

app:v1.0· app:v1.1· app:v1.2 (live)· app:v1.3 (bad!)

If `v1.3` turns out broken, rollback is just: deploy `v1.2` again. That exact image already passed tests and already ran in production, so you know it works.

### The golden rule that enables it

Build once, never rebuild for rollback You must deploy the *same* artifact you originally built and tested. If you rebuild from the old source to roll back, you might pull in different dependency versions and ship something you never tested. The tagged image is the guarantee — redeploy it, don't recreate it.

### Tagging on build

Tag with both the commit SHA (always unique) and a human version. The SHA ties every release back to the exact line of code that produced it.

tag-on-build.yml

```
- name: Build & tag the image
  run: |
    IMAGE=registry/app
    docker build -t $IMAGE:${{ github.sha }} .
    docker tag  $IMAGE:${{ github.sha }} $IMAGE:latest
    docker push $IMAGE:${{ github.sha }}
    docker push $IMAGE:latest
```

### Rolling back — manually, in seconds

Add a `workflow_dispatch` rollback workflow that takes the tag you want to go back to and redeploys it. One click, pick the tag, done.

.github/workflows/rollback.yml

```
name: Rollback
on:
  workflow_dispatch:
    inputs:
      tag:
        description: "Image tag (SHA or version) to roll back to"
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: production          # still goes through approval
    steps:
      - name: Redeploy the chosen tag
        run: ./deploy.sh registry/app:${{ inputs.tag }}
```

### Git tags for versioned releases

Alongside image tags, teams tag the Git commit itself so a version maps to an exact point in history. Pushing a tag can even trigger the release workflow (remember the `on: push: tags` trigger).

tagging a release locally

```
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin v1.2.0          # can trigger the release pipeline

# rollback in Git terms: re-point or redeploy the previous tag
git checkout v1.1.0
```

### Rollback strategies at a glance

| Strategy | How rollback works |
| --- | --- |
| **Re-tag / redeploy** | Redeploy the previous image tag — simplest, works everywhere |
| **Blue-green** | Keep the old version running; switch traffic back instantly if the new one fails |
| **Canary** | Send a small % of traffic to the new version first; if it misbehaves, stop before full rollout |
| **GitOps revert** | The deployed version lives in Git — rollback is `git revert`, and Argo/Flux syncs it back |

Interview-ready summary "Because every release is an immutable artifact tagged by commit SHA, rolling back is redeploying the last known-good tag — not rebuilding or reverting code under pressure. That's why the build-once discipline matters." Say that and you've shown you understand safe deployment.

06 — Prepare

## Interview questions

The questions that actually come up, grouped by depth. Think through your answer first, then click to reveal. Filter by level with the chips below.

Basic What is CI/CD in one sentence?›

CI (Continuous Integration) automatically builds and tests code on every change so problems are caught early; CD (Continuous Delivery/Deployment) automatically packages and releases that code so it reaches users quickly and reliably. Together they turn "a change" into "a tested, shipped change" with little or no manual work.

Basic What's the difference between Continuous Delivery and Continuous Deployment?›

Both automate everything up to production. In Continuous *Delivery*, the release to production is gated by a human clicking a button. In Continuous *Deployment*, even that final step is automatic — every passing build goes straight to production with no human gate.

Basic Why does YAML use indentation instead of braces?›

YAML is designed to be human-readable, so it represents structure visually through indentation rather than with punctuation like `{}` or `[]`. The trade-off is that whitespace becomes significant: wrong indentation changes the meaning or breaks the file. That's why you must use spaces (never tabs) and stay consistent.

Basic Where do GitHub Actions workflow files live?›

In the `.github/workflows/` directory at the root of the repository. Any `.yml` or `.yaml` file there is picked up as a workflow. You can have multiple workflow files, each with its own triggers.

Basic What is the difference between `uses` and `run` in a step?›

`uses` runs a reusable, prebuilt action published by someone else (e.g. `actions/checkout@v4`). `run` executes shell commands directly on the runner. A step uses one or the other, not both.

Basic What is a "runner"?›

A runner is the machine (virtual or physical) that executes your job's steps. GitHub provides hosted runners (`ubuntu-latest`, `windows-latest`, `macos-latest`), or you can register self-hosted runners on your own hardware for special needs like GPUs or internal network access.

Intermediate How do you make one job wait for another?›

Use the `needs` keyword. `deploy: { needs: test }` means the `deploy` job won't start until `test` completes successfully. Jobs without a `needs` relationship run in parallel by default. You can list several: `needs: [build, test]`.

Intermediate How do you handle secrets, and why not just put them in the YAML?›

Secrets go in the repo's **Settings → Secrets and variables**, then are referenced with `${{ secrets.NAME }}`. They're encrypted, masked in logs, and never exposed to workflows triggered from forks. Hardcoding them in YAML would commit them to version history where anyone with repo access — or anyone who sees a public repo — could read them.

Intermediate What is a matrix build and when would you use it?›

A matrix (`strategy.matrix`) runs the same job multiple times with different parameter combinations — for example across Python 3.10 / 3.11 / 3.12, or across three operating systems. Use it to verify your code works on every supported version/platform without duplicating the job. Each combination runs in parallel as its own job.

Intermediate Why run linting before the test suite?›

"Fail fast, fail cheap." Linting takes seconds; a full test suite can take minutes. Ordering cheap checks first means an obvious error (a formatting or import mistake) stops the pipeline almost immediately, giving the developer feedback faster and freeing the runner. The same logic puts unit tests before slower integration tests.

Intermediate What does `${{ }}` do, and is it YAML?›

It's GitHub Actions' *expression syntax*, not YAML. YAML treats the whole thing as a plain string; GitHub evaluates it at runtime, substituting context values like `secrets.X`, `github.sha`, or `matrix.version`, and supports operators and functions (`contains()`, `==`, etc.) inside it.

Intermediate How do you run a job only on the main branch?›

Use an `if` condition: `if: github.ref == 'refs/heads/main'`. A common pattern is a `test` job that runs on all pushes and PRs, plus a `deploy` job that `needs: test` and carries that `if` so deployment only happens on main.

Intermediate What's the difference between `|` and `>` in YAML?›

`|` (literal block) preserves newlines — each line is kept as written, which is what you want for a sequence of shell commands. `>` (folded block) collapses line breaks into spaces, producing one long line — useful for wrapping a long single value across multiple source lines.

Advanced What does "build once, deploy many" mean and why does it matter?›

You build the deployable artifact (e.g. a Docker image) a single time, then promote that *identical* artifact through environments: staging → production. Rebuilding separately for each environment risks subtle differences (dependency versions, build flags) so that what you tested isn't what you shipped. Tagging the artifact with the commit SHA makes every deployment traceable and rollbacks trivial.

Advanced How would you speed up a slow pipeline?›

Several levers: cache dependencies (`cache: pip/npm`) so they aren't reinstalled every run; parallelize independent jobs instead of chaining them with `needs`; run cheap checks first to fail early; use a matrix to parallelize across versions; cache Docker layers; and only run expensive jobs when relevant files change (`paths` filters). Measure first — optimize the slowest stage.

Advanced Why pin action versions (`@v4`) instead of using a floating tag?›

Reproducibility and security. A floating reference means the action's code can change under you, breaking builds or — worse — introducing malicious code if the action is compromised. Pinning to a major tag like `@v4` is the common balance; for maximum safety you can pin to a full commit SHA, which is immutable.

Advanced How do you implement a rollback?›

Because every deploy is an immutable, versioned artifact (tagged by commit SHA or release version), rolling back is simply redeploying the previous known-good tag — not rebuilding or reverting code. This is why the "build once, immutable artifact" discipline matters: it makes rollback a fast, boring operation rather than an emergency.

Advanced What are GitHub Actions "environments" and why use them?›

Environments (defined in repo settings and referenced with `environment: production` in a job) let you attach protection rules to a deployment target: required reviewers who must approve, wait timers, and environment-specific secrets. They turn a deploy job into a gated release, which is how Continuous Delivery enforces its "human approves production" step.

Advanced How does GitHub Actions differ from Jenkins or GitLab CI conceptually?›

All three run pipelines defined as stages, but they differ in hosting and vocabulary. GitHub Actions is tightly integrated with GitHub, config in `.github/workflows/`, with a marketplace of reusable actions. GitLab CI uses a single `.gitlab-ci.yml` with `stages` and `script` keys. Jenkins is self-hosted and historically uses a `Jenkinsfile` (Groovy) with a large plugin ecosystem. The underlying concepts — triggers, stages, artifacts, runners/agents — map across all of them, so learning one transfers.

Advanced What is an artifact in a pipeline, and how is it different from caching?›

An **artifact** is a build output you want to keep or pass between jobs — a compiled binary, a test report, a Docker image. A **cache** is a performance optimization: reusable intermediate data (like downloaded dependencies) that speeds up future runs but can be rebuilt if lost. Losing a cache is harmless; losing an artifact means losing a build result. In Actions you use `upload-artifact`/`download-artifact` for the former and `actions/cache` for the latter.

Advanced Reusable workflow vs composite action — when do you reach for each?›

A **reusable workflow** (`on: workflow_call`) is an entire workflow with its own jobs, called at the job level with `uses` — good for sharing a whole CI process across repos. A **composite action** bundles several *steps* into one unit, called at the step level — good for a repeated setup sequence inside a job. Rule of thumb: sharing jobs → reusable workflow; sharing steps → composite action.

Advanced How do you pass data between jobs?›

Two mechanisms depending on what you're passing. For **files**, use `upload-artifact` in the producer job and `download-artifact` in the consumer (linked by `needs`). For **values**, write to `$GITHUB_OUTPUT` in a step with an `id`, expose it under the job's `outputs`, then read it as `needs.<job>.outputs.<name>`.

Advanced What does the `concurrency` key solve?›

It prevents wasteful or conflicting parallel runs. By setting a `group` (often per branch) with `cancel-in-progress: true`, a new push cancels the still-running pipeline for that branch instead of running both. It's also used to serialize deployments so two deploys can't race.

Advanced Why set `permissions` on the `GITHUB_TOKEN`?›

Least privilege. The auto-generated `GITHUB_TOKEN` can be scoped down (or up) per workflow/job with the `permissions` block. Granting only what a job needs — say `packages: write` for a publish job and read-only elsewhere — limits the blast radius if a step or a third-party action is compromised.

Advanced What is OIDC in CI/CD and why is it better than stored cloud keys?›

OIDC (OpenID Connect) lets a workflow authenticate to a cloud provider by proving its identity and receiving a *short-lived* token at runtime, instead of storing long-lived access keys as secrets. Nothing durable is kept in GitHub, so there's no key to leak or rotate. It requires `permissions: id-token: write` and a trust relationship configured on the cloud side. It's now the recommended way to deploy to AWS/Azure/GCP.

Intermediate How do you debug a failing workflow?›

Start with the run logs in the Actions tab (the error is usually in the last lines of the failed step). Re-run failed jobs for flaky failures. Enable verbose output with the `ACTIONS_STEP_DEBUG` secret. Print context values with `toJSON(github)`. For the hardest cases, add an SSH-into-the-runner step (e.g. action-tmate) to inspect the live environment, and use `if: always()` to upload logs even when a step fails.

Intermediate How do you set a variable that later steps can use?›

Append it to the `$GITHUB_ENV` file: `echo "KEY=value" >> "$GITHUB_ENV"`. A plain shell `export` only lasts for the current step. Similarly, `$GITHUB_OUTPUT` exposes step outputs, `$GITHUB_PATH` adds to PATH, and `$GITHUB_STEP_SUMMARY` writes a Markdown summary to the run page. (The old `::set-output` command is deprecated.)

Tip: in a real interview, back each answer with a concrete example from a pipeline you've built — "in my project I ran lint before pytest so a formatting slip failed in ten seconds instead of three minutes." Specifics beat definitions.

07 — At scale

## Enterprise patterns

A five-person team writes a workflow per repo. A company with 500 repos can't — nobody wants to copy-paste and maintain the same pipeline 500 times. Large organizations solve this with **templates**: write the pipeline once, reuse it everywhere, enforce it centrally. Here's how.

### The core problem: don't repeat the pipeline

If every repo owns its own copy of the CI workflow, then fixing a bug or tightening a security rule means editing hundreds of files. The whole game at scale is to keep **one source of truth** for the pipeline and have every repo point at it. There are three building blocks, from loosest to strictest.

#### 1 · Starter templates

- A suggested workflow new repos can *copy* with one click. A starting point, not enforced.

#### 2 · Centralized reusable workflows

- One repo *owns* the real pipeline; every other repo *calls* it. The workhorse pattern.

#### 3 · Rulesets (required workflows)

- The org *enforces* that certain workflows must pass before any merge. Not optional.

### ① Starter workflow templates

An organization creates a special repo named `.github` and puts workflow templates in a `workflow-templates/` folder. Then, whenever anyone in the org clicks "New workflow" in a repo's Actions tab, these company-approved templates appear as options to start from.

.github/workflow-templates/node-ci.yml

```
name: Node CI (company standard)
on:
  push:
    branches: [ $default-branch ]   # placeholder filled in on copy
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci && npm test
```

What this buys you Every new project starts from the same vetted baseline instead of a blank file. But it's only a *copy* — once copied, each repo's version drifts independently. That's why big orgs layer the next pattern on top.

### ② Centralized reusable workflows — the real workhorse

This is how most large organizations actually operate. One dedicated repo (say `acme/ci-workflows`) holds the canonical pipeline as a reusable workflow. Every other repo contains a tiny file that just *calls* it. Fix the pipeline once in the central repo, and every repo gets the fix.

central: acme/ci-workflows

```
# .github/workflows/standard-ci.yml
on:
  workflow_call:
    inputs:
      language:
        type: string
        default: python

jobs:
  lint-test-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./lint.sh
      - run: ./test.sh
      - run: ./security-scan.sh
```

every other repo (3 lines!)

```
# .github/workflows/ci.yml
on: [push, pull_request]

jobs:
  ci:
    # call the central pipeline, pinned to a version tag
    uses: acme/ci-workflows/.github/workflows/standard-ci.yml@v2
    with:
      language: python
    secrets: inherit        # pass the repo's secrets through
```

Why the `@v2` matters The caller pins to a **version tag** of the central repo. The platform team ships `v3` with improvements; repos upgrade when ready by changing one character. This gives central control *and* per-repo safety — nobody's pipeline changes under them without a deliberate bump.

### ③ Rulesets — enforce it across the org

Templates and reusable workflows are opt-in. To *require* that a workflow runs and passes before anyone can merge, organizations use **repository rulesets** (this replaced the older "required workflows" feature). An org admin configures, in settings, a rule like: "On every repo matching this pattern, the central security workflow must pass before merge to `main`." Developers can't bypass it.

The enforcement layer Rulesets are set in the GitHub UI (Organization → Settings → Rulesets), not in YAML. They can require specific status checks, require the central reusable workflow to run, block force-pushes, and demand pull-request reviews — all enforced org-wide.

### Shared internal actions

Alongside workflows, big orgs publish their own **composite or JavaScript actions** in an internal repo — a standard "set up our toolchain" action, a "deploy to our cloud" action — so steps are reused the same way workflows are.

using an internal action

```
steps:
  - uses: acme/actions/setup-toolchain@v1   # company's own action
  - uses: acme/actions/deploy@v1
    with:
      environment: staging
```

### Environment promotion — build once, promote everywhere

The classic enterprise deployment shape: the *same* artifact is built once, then promoted through a chain of environments, each with its own approval gate. Nothing is ever rebuilt for production — what you tested is exactly what ships.

Build once→ Dev→ Staging→ Production

promotion.yml

```
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: docker build -t app:${{ github.sha }} .
      - run: docker push registry/app:${{ github.sha }}

  deploy-staging:
    needs: build
    environment: staging              # auto-deploys
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh staging registry/app:${{ github.sha }}

  deploy-prod:
    needs: deploy-staging
    environment: production           # waits for reviewer approval
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh prod registry/app:${{ github.sha }}  # SAME image
```

The golden rule, restated Notice `${{ github.sha }}` is identical in all three deploys. The image built in job one is the exact image that reaches production — never rebuilt. Rollback = redeploy the previous SHA.

### GitOps — the modern deployment style

Many large orgs go one step further: the pipeline doesn't deploy directly. Instead it updates a **Git repository that describes the desired state** (Kubernetes manifests), and a tool like **Argo CD** or **Flux** continuously makes the cluster match what's in Git. Git becomes the single source of truth; a rollback is a `git revert`.

CI builds image→ Updates manifest in Git→ Argo CD syncs cluster

### How it all fits together

| Layer | Pattern | Who owns it |
| --- | --- | --- |
| New-repo baseline | Starter workflow templates | Platform team (in `.github` repo) |
| The actual pipeline | Centralized reusable workflows | Platform team (versioned repo) |
| Reused steps | Internal composite/JS actions | Platform team |
| Enforcement | Repository rulesets | Org admins |
| Releasing | Environment promotion + approvals | Each product team |
| Deploying | GitOps (Argo CD / Flux) | Platform / SRE team |

Takeaway for a beginner You don't need to build all this on day one. But in an interview, saying "at scale you'd use a centralized reusable workflow pinned to a version tag, enforced by org rulesets, deploying one immutable artifact through environment gates" signals you understand how real companies operate — not just toy pipelines.

08 — Do it, don't just read it

## Practicals

You've read the theory. This is where it becomes real. Start with the first project — a repo you actually push and watch run — then work through the hands-on topics. Three full project ideas at the end turn everything into portfolio-worthy work.

### Your first pipeline — push it today

The fastest way to learn. In about ten minutes you'll have a repo whose tests run automatically on every push, with a green checkmark you earned. Follow these exactly.

1

**Create a repo** on GitHub (e.g. `ci-hello`), cloned to your machine, or make it locally and push.

2

**Add a tiny Python app and a test** (two small files below).

3

**Add the workflow file** at `.github/workflows/ci.yml`.

4

**Commit and push.** Open the *Actions* tab and watch it run green.

5

**Break the test on purpose**, push again, and watch it go red — then practise the Debugging sub-tab.

app.py

```
def add(a, b):
    return a + b
```

test_app.py

```
from app import add

def test_add():
    assert add(2, 3) == 5
```

.github/workflows/ci.yml

```
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install pytest
      - run: pytest -v
```

the git commands

```
git init
git add .
git commit -m "first pipeline"
git branch -M main
git remote add origin https://github.com/YOU/ci-hello.git
git push -u origin main
```

That's the whole loop Push → GitHub reads `ci.yml` → spins up a runner → installs pytest → runs your test → shows green or red. Everything else in this guide is a richer version of this one loop.

### Three real projects to build

Each one is portfolio-worthy and teaches a different slice of CI/CD. They go from beginner to ambitious. Build at least one end to end — it's the best possible interview talking point.

Beginner

#### 1 · Auto-deployed personal website

**What you build:** a static site (portfolio, blog, docs) that rebuilds and redeploys to GitHub Pages every time you push to `main`.

**What it teaches:** the core CI→CD loop, build steps, and real deployment — all free, no cloud account needed.

**Pipeline shape:** checkout → build (or just copy files) → deploy to Pages.

pages-deploy.yml (sketch)

```
on: { push: { branches: [main] } }
permissions: { pages: write, id-token: write }
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: github-pages
    steps:
      - uses: actions/checkout@v4
      - uses: actions/upload-pages-artifact@v3
        with: { path: "." }
      - uses: actions/deploy-pages@v4
```

**Stretch:** add an HTML-link checker step that fails the build if a link is broken.

Intermediate

#### 2 · Dockerized API with full CI

**What you build:** a small REST API (Flask/FastAPI or Express) that, on every push, is linted, tested, built into a Docker image, and pushed to GitHub Container Registry (GHCR) — tagged by commit SHA.

**What it teaches:** multi-stage pipelines, `needs`, Docker build + push, image tagging (the foundation of rollback), and the built-in `GITHUB_TOKEN`.

**Pipeline shape:** *job 1* lint + test → *job 2* (needs job 1, only on main) build image → push to GHCR as `ghcr.io/you/api:<sha>`.

structure

```
jobs:
  test:    # ruff + pytest
  publish:
    needs: test
    if: github.ref == 'refs/heads/main'
    # docker/login-action → docker/build-push-action
    # tag: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

**Stretch:** add a `services:` Postgres container and write one integration test against it.

Advanced

#### 3 · Deploy to AWS with OIDC + environment gates

**What you build:** the Dockerized API from project 2, now deployed to AWS (ECS, Lambda, or an S3-hosted frontend) using **OIDC** — no stored keys — promoted through `staging` → `production` with a manual approval, and a one-click rollback workflow.

**What it teaches:** everything that makes you look senior — keyless cloud auth, environment promotion, build-once/deploy-many, approval gates, and tag-based rollback.

**Pipeline shape:** build & push image → deploy-staging (auto) → deploy-prod (needs approval) → separate `workflow_dispatch` rollback that redeploys a chosen tag.

the senior-signal bits

```
permissions: { id-token: write, contents: read }
# aws-actions/configure-aws-credentials@v4 (OIDC role)
# environment: production  → reviewer must approve
# same image SHA promoted staging → prod (never rebuilt)
```

**Stretch:** add a Slack notification on prod deploy and a canary step that deploys to 10% first.

How to use these Don't just read them — pick project 1 today, finish it, then climb. By project 3 you'll have a repo that demonstrates the exact patterns real companies use. "I built a pipeline that deploys to AWS with OIDC and tag-based rollback" is a sentence that gets interviews.

### Testing strategies in CI

CI is only as good as its tests. These are the practices that turn "tests run" into "tests protect you."

### Coverage gates

Fail the build if test coverage drops below a threshold — stops untested code sneaking in.

coverage-gate.yml

```
- run: pytest --cov=src --cov-fail-under=80   # fails under 80%
```

### Fast feedback: order and parallelism

Run the quick, broad checks before the slow, narrow ones, so failures surface early.

Lint (secs)-> Unit tests (secs)-> Integration (mins)

### Flaky tests — the quiet killer

A flaky test passes and fails randomly, eroding trust in the whole pipeline. Handle them deliberately:

#### Contain

- Retry only known-flaky tests (e.g. pytest-rerunfailures), never hide real failures
- Quarantine a flaky test into a separate non-blocking job while you fix it

#### Fix the cause

- Usually: timing/sleeps, shared state, network, random data without a seed
- Make tests isolated and deterministic

### Matrix testing + fail-fast off

matrix-test.yml

```
strategy:
  fail-fast: false            # see ALL version failures, not just the first
  matrix:
    python: ["3.10", "3.11", "3.12"]
```

Interview note "How do you handle a flaky test?" is common. Answer: isolate the root cause (timing/shared state), make it deterministic, and if you must ship meanwhile, quarantine it to a non-blocking job — never blanket-retry everything, because that masks real regressions.

### Docker layer caching

Docker builds in layers, and unchanged layers can be reused instead of rebuilt. Without caching, CI rebuilds every layer every time — slow and wasteful. With it, only the layers you actually changed rebuild.

### The two things that make it fast

#### 1 · Order your Dockerfile well

- Copy dependency files and install *before* copying your source
- So a code change doesn't invalidate the (slow) dependency layer

#### 2 · Enable build cache in CI

- Use Buildx with GitHub Actions cache
- Layers persist between runs

Dockerfile (cache-friendly order)

```
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .        # deps first…
RUN pip install -r requirements.txt  # …this layer is cached
COPY . .                       # code last — changes don't bust the deps layer
CMD ["python", "app.py"]
```

docker-cache.yml

```
- uses: docker/setup-buildx-action@v3
- uses: docker/build-push-action@v6
  with:
    push: true
    tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
    cache-from: type=gha        # read cache from GitHub Actions
    cache-to: type=gha,mode=max   # write all layers back to cache
```

The payoff A first build might take 3 minutes; subsequent builds where only code changed can drop to 20–30 seconds, because the dependency layers come straight from cache.

### Release automation

Instead of manually deciding version numbers and writing changelogs, let the pipeline do it based on your commit messages.

### Conventional Commits — the foundation

Write commit messages in a small structured format, and tools can infer the right version bump automatically.

| Commit prefix | Means | Version bump |
| --- | --- | --- |
| `fix:` | a bug fix | patch (1.2.0 → 1.2.1) |
| `feat:` | a new feature | minor (1.2.0 → 1.3.0) |
| `feat!:` / `BREAKING CHANGE` | incompatible change | major (1.2.0 → 2.0.0) |

### Auto-version, tag & changelog

Tools like **semantic-release** (JS) or **release-please** (Google, multi-language) read those commits, compute the next version, create the Git tag and GitHub Release, and generate the changelog — all in CI.

release-please.yml (sketch)

```
on: { push: { branches: [main] } }
permissions: { contents: write, pull-requests: write }
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        # opens a "release PR"; merging it tags + publishes the release
```

Why teams love it No human argues about version numbers, the changelog is always accurate, and releases become a merge of an auto-generated PR. It pairs perfectly with the tag-based rollback you learned: every release is a clean, labelled tag.

### Language recipes

Every language's CI follows the same three beats: **set up the toolchain → install deps (cached) → test**. Only the commands change. Here are the common ones beyond Python and Node.

Java (Maven)

```
- uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: "21"
    cache: maven
- run: mvn -B verify
```

Go

```
- uses: actions/setup-go@v5
  with: { go-version: "1.23" }
- run: go test ./...
```

Rust

```
- uses: actions/checkout@v4
- run: rustup toolchain install stable
- run: cargo test
```

The transferable insight Once you've written one pipeline, every other language is just swapping the setup action and the test command. The structure — triggers, jobs, steps, caching — is identical.

### Writing your own actions

When you repeat the same logic across workflows, package it as an action. There are three kinds; composite is the easiest and most common.

| Type | What it is | Use when |
| --- | --- | --- |
| **Composite** | A bundle of steps in YAML | You're combining existing shell/action steps (most cases) |
| **JavaScript** | Node code that runs directly on the runner | You need logic, the GitHub API, or speed |
| **Docker** | Runs inside a container you define | You need a specific OS/toolchain |

composite — action.yml

```
name: Greet
description: "Print a greeting"
inputs:
  who:
    description: "name to greet"
    required: true
runs:
  using: composite
  steps:
    - run: echo "Hello, ${{ inputs.who }}!"
      shell: bash
```

Drop that in `.github/actions/greet/action.yml` and use it with `uses: ./.github/actions/greet`. To share it across repos, put it in its own repo and tag a version.

Start here 90% of "I need a custom action" moments are solved by a composite action. Reach for JavaScript or Docker only when you genuinely need code or a special environment — and when you do, the official `actions/javascript-action` template is the fastest start (linked in Everything).

### Using Actions beyond CI/CD

Actions reacts to *any* repo event, so it automates project chores too — no pipeline involved.

| Automation | What it does | Popular action |
| --- | --- | --- |
| Auto-label PRs | Tags PRs by which files changed | `actions/labeler` |
| Release notes draft | Builds release notes from merged PRs | `release-drafter` |
| Stale cleanup | Closes inactive issues/PRs | `actions/stale` |
| Welcome newcomers | Comments on a first-time contributor's PR | `actions/first-interaction` |
| Assign reviewers | Auto-requests review from a team | built-in CODEOWNERS |

auto-label.yml

```
on: [pull_request_target]
jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/labeler@v5
```

Why it matters Knowing Actions is a general automation engine — not just a CI tool — is a mark of fluency. It's the same triggers/jobs/steps model you already know, pointed at housekeeping instead of builds.

### Pipeline observability

Once pipelines matter, you want to *see* how they behave: what's slow, what fails often, what's costing minutes. You can't improve what you don't measure.

### What to watch

#### Speed

- Total run duration over time
- Slowest jobs/steps
- Cache hit rate

#### Reliability

- Pass/fail rate per workflow
- Flaky-test frequency
- Mean time to green after a failure

#### Cost

- Runner minutes used
- Minutes by OS (Linux vs mac/Win)
- Spend trend

### Where to see it

| Source | Gives you |
| --- | --- |
| Repo **Insights → Actions** | Built-in usage & run metrics |
| Org **billing** page | Minutes consumed and remaining |
| `$GITHUB_STEP_SUMMARY` | Your own per-run dashboard (timings, results) |
| Export to Datadog/Grafana | Long-term trends across all repos (advanced) |

Practical first step Before reaching for external tooling, write key timings and results to `$GITHUB_STEP_SUMMARY` (from the Workflow commands sub-tab). You get a per-run dashboard for free.

### GitHub Actions vs GitLab CI vs Jenkins

Same concepts, different words. Here's the same simple pipeline in all three, so you can read any of them.

GitHub Actions

```
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pytest
```

GitLab CI (.gitlab-ci.yml)

```
test:
  image: python:3.12
  stage: test
  script:
    - pytest
```

Jenkins (Jenkinsfile, declarative)

```
pipeline {
  agent any
  stages {
    stage('Test') {
      steps { sh 'pytest' }
    }
  }
}
```

| Concept | GitHub Actions | GitLab CI | Jenkins |
| --- | --- | --- | --- |
| Config file | `.github/workflows/*.yml` | `.gitlab-ci.yml` | `Jenkinsfile` |
| Unit of work | job → steps | job → script | stage → steps |
| Machine | `runs-on` (runner) | `tags` (runner) | `agent` |
| Reuse | actions, reusable workflows | `include`, templates | shared libraries |
| Hosting | GitHub-hosted | GitLab-hosted/self | self-hosted |

The takeaway Triggers, jobs, steps, runners, artifacts, caching — every tool has all of them under different names. Learn the concepts (which you now have) and switching tools is just learning new keywords.

09 — The link library

## Everything

Official docs and trusted references for every topic — both the ones taught in this guide and the deeper or vendor-specific ones a single page can't fully hold. Every link opens the canonical source in a new tab.

Covered in this guide

### YAML foundations

The grammar: indentation, mappings, lists, quoting, multiline strings, anchors.

[YAML 1.2 SpecificationThe authoritative language spec — every rule, in full.yaml.org](https://yaml.org/spec/1.2.2/) [Learn YAML in Y MinutesThe entire syntax as one annotated example file.learnxinyminutes.com](https://learnxinyminutes.com/docs/yaml/) [YAML reference guideReadable walkthrough of scalars, collections, anchors and tags.yaml.info](https://www.yaml.info/learn/index.html) [YAML Lint (online)Paste a file, instantly check it's valid before committing.yamllint.com](https://www.yamllint.com/)

Covered in this guide

### GitHub Actions core

Workflow anatomy, triggers, jobs, steps, `uses` vs `run`, secrets, matrix.

[GitHub Actions — docs homeThe hub for everything; start here if a deep link ever moves.docs.github.com](https://docs.github.com/en/actions) [Workflow syntax referenceEvery keyword and its exact meaning — the canonical reference.docs.github.com](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) [QuickstartYour first workflow, step by step.docs.github.com](https://docs.github.com/en/actions/writing-workflows/quickstart) [Events that trigger workflowsThe full list of `on:` events and their payloads.docs.github.com](https://docs.github.com/en/actions/reference/events-that-trigger-workflows) [Using secretsHow to store and reference secrets safely.docs.github.com](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions) [Matrix strategyRun a job across many versions or OSes in parallel.docs.github.com](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/running-variations-of-jobs-in-a-workflow)

Covered in this guide

### Expressions, contexts & the marketplace

The `${{ }}` syntax, context variables, and where prebuilt actions come from.

[ExpressionsOperators and functions usable inside `${{ }}`.docs.github.com](https://docs.github.com/en/actions/learn-github-actions/expressions) [ContextsEvery `github.*`, `secrets.*`, `matrix.*` value available.docs.github.com](https://docs.github.com/en/actions/learn-github-actions/contexts) [Actions MarketplaceSearch prebuilt actions to drop into `uses:`.github.com](https://github.com/marketplace?type=actions) [actions/checkoutThe action every workflow starts with — its full options.github.com](https://github.com/actions/checkout) [actions/setup-pythonSet up Python + built-in pip caching.github.com](https://github.com/actions/setup-python) [actions/setup-nodeSet up Node + built-in npm caching.github.com](https://github.com/actions/setup-node)

Covered in this guide

### Scaling features

Artifacts, caching, job outputs, reusable & composite workflows, concurrency, environments.

[Storing data as artifactsUpload/download files between jobs and keep build outputs.docs.github.com](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts) [Caching dependencies`actions/cache` keys, restore-keys, and `hashFiles()`.docs.github.com](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows) [Reusing workflows`workflow_call`, inputs, secrets: inherit — the full rules.docs.github.com](https://docs.github.com/en/actions/using-workflows/reusing-workflows) [Creating a composite actionBundle repeated steps into one local action.docs.github.com](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) [ConcurrencyCancel superseded runs and serialize deployments.docs.github.com](https://docs.github.com/en/actions/using-jobs/using-concurrency) [Environments & approvalsRequired reviewers, wait timers, scoped secrets.docs.github.com](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) [Service containersRun Postgres/Redis sidecars for integration tests.docs.github.com](https://docs.github.com/en/actions/using-containerized-services/about-service-containers)

Deeper — beyond this guide

### Topics this guide only touches

Worth a read once you're comfortable — the areas a single learning page can't fully cover.

[Security hardeningPinning to SHAs, `permissions`, protecting against injection.docs.github.com](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions) [Writing your own actionsCustom JavaScript, Docker and composite actions.docs.github.com](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions) [Self-hosted runnersRun jobs on your own hardware (GPUs, internal networks).docs.github.com](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners) [Deployment patternsReal deployment strategies and OIDC cloud auth.docs.github.com](https://docs.github.com/en/actions/deployment/deploying-with-github-actions) [Awesome ActionsA huge curated list of actions, tutorials and tooling.github.com](https://github.com/sdras/awesome-actions) [Docker + ActionsOfficial guide to building and pushing images in CI.docs.docker.com](https://docs.docker.com/build/ci/github-actions/)

Free courses — learn by doing

### GitHub Actions courses & hands-on learning

The fastest way to actually learn is an interactive course plus your own test repo. These are the best free ones.

[GitHub SkillsOfficial hands-on courses that run inside real repos — the single best starting point.skills.github.com](https://skills.github.com/) [Understanding GitHub ActionsThe official conceptual walkthrough — read this alongside tab 02.docs.github.com](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions) [GitHub Actions in 1 hour (TechWorld with Nana)The most-watched free crash course — concepts to real pipelines.youtube.com](https://www.youtube.com/watch?v=R8_veQiYBjI) [freeCodeCamp: step-by-step guideA written, beginner-paced walkthrough you can follow at your own speed.freecodecamp.org](https://www.freecodecamp.org/news/learn-to-use-github-actions-step-by-step-guide/) [Build your first workflow15-minute official quickstart — do this today in a throwaway repo.docs.github.com](https://docs.github.com/en/actions/writing-workflows/quickstart) [DevOps roadmapSee where CI/CD sits in the bigger DevOps picture.roadmap.sh](https://roadmap.sh/devops)

Enterprise & at scale

### Templates & organization-wide CI/CD

The patterns from tab 07 — starter templates, centralized reusable workflows, rulesets, GitOps.

[Starter workflow templatesCreate org-wide templates in the `.github` repo.docs.github.com](https://docs.github.com/en/actions/sharing-automations/creating-workflow-templates-for-your-organization) [Reusing workflows (central pipeline)The `workflow_call` pattern every big org relies on.docs.github.com](https://docs.github.com/en/actions/using-workflows/reusing-workflows) [Repository rulesetsEnforce required workflows and checks across the org.docs.github.com](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) [Sharing actions & workflowsPublish internal actions for the whole organization.docs.github.com](https://docs.github.com/en/actions/sharing-automations/sharing-actions-and-workflows-with-your-organization) [Argo CD (GitOps)The leading GitOps tool for Kubernetes deployments.argo-cd.readthedocs.io](https://argo-cd.readthedocs.io/en/stable/) [Flux (GitOps)The other major GitOps controller.fluxcd.io](https://fluxcd.io/flux/)

Other CI/CD tools

### The same ideas, different vocabulary

Once you know GitHub Actions, these are mostly translation — the concepts carry over.

[GitLab CI/CD (`.gitlab-ci.yml`)`stages`, `script`, `image` — the GitLab equivalent.docs.gitlab.com](https://docs.gitlab.com/ee/ci/yaml/) [Jenkins Pipeline (Jenkinsfile)Declarative pipeline syntax for the self-hosted classic.jenkins.io](https://www.jenkins.io/doc/book/pipeline/syntax/) [CircleCI configAnother widely-used YAML pipeline format.circleci.com](https://circleci.com/docs/configuration-reference/) [Azure Pipelines YAMLMicrosoft's CI/CD schema reference.learn.microsoft.com](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/)

If any GitHub link ever 404s, the docs occasionally reshuffle their paths — start from **docs.github.com/en/actions** and search the keyword. The content stays; only the URL moves.