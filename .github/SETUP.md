## Installation Options


Copy only the `.github/` folder into your repo. RepoSherlock runs from a
pre-built image — no source code needed.

```text
your-repo/
├── .github/
│   ├── actions/reposherlock/action.yml
│   ├── workflows/reposherlock.yml
│   ├── workflows/reposherlock-backlog.yml
│   ├── maintainers.yml
│   └── SETUP.md
└── docs/                    (optional — domain documentation)
```

1. Copy the `.github/` folder into your repo
2. Add a `GEMINI_API_KEY` secret (Settings → Secrets → Actions)
3. Create `.github/maintainers.yml` from the template
4. Put domain docs in `docs/` (optional)
5. Push — automation starts on the next issue or PR

> The pre-built image has to exist first. Until `📦 Publish Image` has run
> on the source repository, the action falls back to building from source,
> which needs `Dockerfile`, `requirements.txt`, `app/` and `frontend/`
> present — the full checkout, not just `.github/`.

---

# RepoSherlock — Setup Guide

## Quick Start (3 steps)

### 1. Add one secret

Your repo → **Settings → Secrets and variables → Actions → New repository secret**

| Secret | Required | How to get it |
|--------|----------|---------------|
| `GEMINI_API_KEY` | Yes | [aistudio.google.com](https://aistudio.google.com) → Get API Key |
| `COPILOT_PAT` | No — only for Copilot | GitHub PAT with `repo` scope |
| `MAIL_USERNAME` | No — only for email | Your SMTP username |
| `MAIL_PASSWORD` | No — only for email | Your SMTP password |

`GITHUB_TOKEN` is **not** in this list: Actions provides it automatically.

Issues are triaged and labelled even without the API key. PR reviews require
the key — the Action will fail with a clear error if it is missing.

### 2. Create the maintainers config

Copy `.github/maintainers.yml.template` to `.github/maintainers.yml` and set
your GitHub username. Without the file the automation still runs, with Copilot
and email off.

### 3. Push and test

Open an issue. The automation triggers on its own.

> Workflows must be on your **default branch** for `issues:` and `schedule:`
> events to fire. A workflow sitting on a feature branch is ignored.

---

## What happens

**When someone opens an issue**

1. Classified as question / bug / enhancement
2. Labelled automatically
3. Questions → answered from your documentation, if any is indexed
4. Bugs and enhancements → assigned to Copilot, if enabled
5. Maintainers are notified

**When someone opens a PR**

1. PR is labelled
2. Copilot is asked to review, if enabled
3. RepoSherlock runs a structured checklist review
4. The review is posted as a PR comment
5. Maintainers are notified for external contributors

---

## The workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `reposherlock.yml` | issue opened, PR opened/updated | Triage, label, answer, review, notify |
| `reposherlock-backlog.yml` | weekdays 08:00 UTC, or manual | Marks inactive issues and PRs stale |
| `reposherlock-setup.yml` | called by other workflows | Builds and starts RepoSherlock on the runner |

`.github/actions/reposherlock/` is a composite action that runs the service
inside the calling job, so later steps can reach it on `localhost:8000`.

---

## Optional: Backlog Sweep

`reposherlock-backlog.yml` runs on weekdays and labels anything untouched for
`stale_days`. Delete the file to turn it off.

## Optional: Domain Documentation

Put documents in `docs/` to enrich reviews and answers with your
own context. Without them, question issues come back as "not covered" and are
escalated to a maintainer. Locally, enable it under **Settings → Knowledge
Base**.

---

## Troubleshooting

**Nothing triggers.** Workflows must be on the default branch, and Actions must
be enabled under Settings → Actions → General.

**RepoSherlock fails to start.** Check the run log. The usual cause is a
missing `GEMINI_API_KEY`, and the two paths differ: an **issue** still gets
triaged, labelled and escalated to maintainers, with a warning in the log and
the AI steps skipped. A **pull request** fails at the first step with an error,
because a review without a model is nothing at all.

**Copilot assignment fails.** `COPILOT_PAT` needs `repo` scope, or set
`copilot_enabled: false`. Either way the run stays green — the step warns and
moves on.

**PRs from forks get no review.** GitHub withholds secrets from fork PRs, so
`GEMINI_API_KEY` is empty and the review step is skipped. Labelling and the
maintainer notification still run. Use `pull_request_target` only if you
understand the risk of running untrusted code with write permissions.

**Every run builds the Docker image.** The composite action builds from source
on each event, which installs torch and downloads the embedding model. On a
private repo those minutes are metered. Publishing the image to GHCR and
pulling it instead is the fix.
