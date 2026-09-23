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
| `.github/workflows/release.yml` | the release pipeline: version, tag, GitHub release — no build | **called** as a reusable workflow, never copied |
| `.github/workflows/release-self.yml` | releases this repository itself | stays here |
| `examples/caller-release.yml` | the caller stub, source-only | copied once into each repository |
| `examples/publish.yml` | job fragment for publishing to a registry | pasted into a repository's `release.yml` when it publishes |
| `examples/ci.yml` | minimal pull request gate: build and test | copied once into each repository |
| `examples/package.yml` | job fragment that builds artifacts and attaches them to the release | pasted into a repository's `release.yml` when it ships artifacts |
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

This copies `AGENTS.md`, adds `.releaserc.json` and a source-only release workflow
when they are missing, pushes a branch, and opens a PR. Review the diff and merge
it with squash. Then add the jobs the repository actually needs:

| The repository ships | Jobs in its `.github/workflows/release.yml` |
|---|---|
| nothing but source | `release` — GitHub attaches the source tarball and zip itself |
| a registry package (npm, PyPI, crates.io) | `release` + `publish` from `examples/publish.yml` |
| build artifacts on the release (deb, rpm, tarball, image) | `release` + `package` from `examples/package.yml` |
| both | `release` + `publish` + `package` |

The jobs are independent siblings, each gated on
`needs.release.outputs.new_tag != ''`, so adding or deleting one never rewires the
others. Repositories that ship nothing keep the stub exactly as synced.

## Releasing and publishing are separate

The shared workflow is deliberately language-agnostic. It computes the version,
creates the tag, and creates the GitHub release, then exposes the tag as
`new_tag`. Building and publishing the artifact is language-specific, so it lives
in the `publish` job of each repository's own caller stub.

Two consequences worth knowing:

- **Registry secrets stay out of the shared workflow.** `NPM_TOKEN`,
  `PYPI_TOKEN`, `CARGO_REGISTRY_TOKEN` and friends are defined in the repository
  that publishes, never here.
- **A tag pushed with `GITHUB_TOKEN` does not trigger a new workflow run**, so a
  separate `on: push: tags` publish workflow would never fire. Chain the publish
  job with `needs:`, which is what the example does.

The Node runtime in the shared workflow is the runtime of the semantic-release
tool itself; it never touches the project's own `package.json` or `node_modules`.

## Tests run in their own workflow

`examples/ci.yml` is a minimal pull request gate, copied to
`.github/workflows/ci.yml`. It is deliberately separate from `release.yml`: tests
run on pull requests, releases run on pushes to the default branch. Replace the
toolchain and test command placeholders.

The testing rules in `AGENTS.md` only bind if the repository enforces them: in the
branch protection rule or ruleset for the default branch, require the `test` status
check before merging.

`scripts/sync.sh` does not install this file — the test command cannot be guessed,
and the placeholder fails on purpose rather than reporting a green run that tested
nothing.

## Packaging belongs to the repository

`examples/package.yml` is a job fragment, not a workflow: it reads
`needs.release.outputs.new_tag`, so it must sit in the same file as the `release`
job. Paste it into that repository's `.github/workflows/release.yml`.

It builds into `dist/` and attaches the files to the GitHub release. Replace the
build step with the project's own tool — `nfpm` for deb/rpm/apk, `cargo deb` for
Rust, `dpkg-buildpackage` for Debian-native packages, `tar` for plain archives,
`docker build`/`push` for images. GitHub Packages hosts npm, Docker, Maven, NuGet,
Gradle and RubyGems; deb and rpm have to go to the release assets or to a separate
apt or yum repository.

## Verified behaviour

A sandbox repository exercised the whole flow end to end. Confirmed:

- `chore:`, `docs:`, `ci:` merges release nothing and skip `publish`/`package`;
  `feat:` → minor, `fix:` → patch, `feat:` plus a `BREAKING CHANGE:` footer → major.
- The first release of an untagged repository is `1.0.0`.
- Release notes are generated from the merged PR, and the `package` job's artifact
  is attached to the release.
- The release job adds no commit to the default branch.

Two GitHub settings decide how a PR becomes the release commit. Both are defaults,
and both are easy to get wrong:

- **`squash_merge_commit_title` = `COMMIT_OR_PR_TITLE`** — a **single-commit** PR
  puts the *commit message* on the default branch and ignores the PR title; a
  multi-commit PR puts the *PR title*. Keep the PR title identical to the commit
  subject and neither case can be wrong.
- **`squash_merge_commit_message` = `COMMIT_MESSAGES`** — the PR's commit messages
  are carried into the squash commit, the PR **description is not**. A
  `BREAKING CHANGE:` footer written only in the PR description is dropped, and the
  change ships as a minor bump instead of a major one.

The repository's default workflow permissions may be read-only; that does not
block releases, because the caller sets `permissions:` explicitly and the job log
then reports `Contents: write`.

## Keeping repositories up to date

- **Release pipeline** — nothing to do. Callers pin `@v1`, so a fix here reaches
  them as soon as `v1` moves. Repositories that pin a commit SHA instead get a
  bump PR from Dependabot.
- **`AGENTS.md`** — re-run `scripts/sync.sh` and merge the resulting PR.

## The `v1` tag

semantic-release publishes immutable `vX.Y.Z` tags. Reusable workflow callers need
a stable ref, so `release-self.yml` keeps a movable `v1` tag pointing at the newest
release (the first one was created by hand when this repository was bootstrapped).

A movable tag means whoever can push tags here can change what every downstream
repository runs. If that is unacceptable, pin commit SHAs in the caller stubs
instead of `v1`, and let Dependabot open the bump PRs.

## Ground rules

- Edit the rules and the pipeline **here**. Downstream copies are generated; a
  change made there is lost on the next sync.
- Keep `AGENTS.md` language- and project-agnostic. Repository-specific facts
  belong in that repository.
