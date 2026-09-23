# repo-template

Canonical agent rules and release automation, reused by every other repository.

This repository is the source of truth for two things that must not drift between
repositories: the operating rules agents follow (`AGENTS.md`) and the release
pipeline (semantic-release, driven by Conventional Commit prefixes).

## Contents

| Path | Role | How it is reused |
|---|---|---|
| `AGENTS.md` | operating rules for agents | copied into each repository root |
| `.releaserc.json` | semantic-release configuration | copied once, then extended per repository |
| `.github/workflows/release.yml` | the release pipeline | **called** as a reusable workflow, never copied |
| `.github/workflows/release-self.yml` | releases this repository itself | stays here |
| `examples/caller-release.yml` | the caller stub | copied once into each repository |
| `scripts/sync.sh` | syncs the copied files and opens a PR | run from here |

## Adopting in a new repository

```bash
gh repo create <owner>/<name> --public --template yang-dong-yd/repo-template
```

Or click **Use this template** on the repository page.

## Adopting in an existing repository

```bash
scripts/sync.sh ../some-repo
```

This copies `AGENTS.md`, adds `.releaserc.json` and the release workflow when they
are missing, pushes a branch, and opens a PR. Review the diff and merge it with
squash. If the repository publishes to npm, add `@semantic-release/npm` to its
`.releaserc.json` and define the `NPM_TOKEN` secret.

## Keeping repositories up to date

- **Release pipeline** — nothing to do. Callers pin `@v1`, so a fix here reaches
  them as soon as `v1` moves. Repositories that pin a commit SHA instead get a
  bump PR from Dependabot.
- **`AGENTS.md`** — re-run `scripts/sync.sh` and merge the resulting PR.

## The `v1` tag

semantic-release publishes immutable `vX.Y.Z` tags. Reusable workflow callers need
a stable ref, so keep a movable `v1` tag pointing at the latest release:

```bash
git tag -f v1 v1.0.0 && git push -f origin v1
```

A movable tag means whoever can push tags here can change what every downstream
repository runs. If that is unacceptable, pin commit SHAs instead of `v1`.

## Ground rules

- Edit the rules and the pipeline **here**. Downstream copies are generated; a
  change made there is lost on the next sync.
- Keep `AGENTS.md` language- and project-agnostic. Repository-specific facts
  belong in that repository.
