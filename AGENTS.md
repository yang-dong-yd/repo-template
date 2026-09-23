# AGENTS.md

Operating rules for agents working in this repository. They are mandatory, not advisory.

## 1. Git Workflow

**Never modify code directly on the default branch.** Always create a new branch first.

- The default branch may be `main` or `master`; `<default-branch>` below means whichever it is.
- Sync before branching: `git switch <default-branch> && git pull --ff-only`.
- Create a branch: `git switch -c <type>/<short-description>`.
  - Examples: `feat/oauth-login`, `fix/null-pointer-on-logout`, `chore/bump-deps`.
  - `<type>` is one of the commit types listed below.
- Keep branches small and single-purpose. One branch solves one problem.
- Keep the branch current by rebasing on `<default-branch>`; do not merge `<default-branch>` into it.
- Never rewrite history that has already been pushed to a shared branch.

### Commits

Every commit message follows Conventional Commits:

```
<type>: <subject>

<body>

<footer>
```

- `<type>` is required and must be one of: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- `<subject>`: imperative mood, lowercase, no trailing period, 72 characters or fewer. Example: `feat: add OAuth login`.
- `<body>`: optional. Explain what changed and why, not how. Wrap at 72 characters.
- `<footer>`: optional. Link issues (`Closes #123`) and mark breaking changes with `BREAKING CHANGE: <description>`.
- One logical change per commit. Do not mix refactoring with behavior changes.
- Never commit generated artifacts, secrets, credentials, or local config.

### Issues

- Search before creating: `gh issue list --search "<keywords>"`. Never open a duplicate.
- Read the full issue and all its comments before starting; check for a linked PR to avoid repeating work.
- Create issues with `gh issue create`. Title: short, imperative, no trailing period.
  - Bug body: **Context**, **Expected**, **Actual**, **Reproduce**.
  - Feature body: **Problem**, **Proposal**, **Alternatives**.
- Use only labels that already exist in the repository. Never invent or create labels.
- One issue per problem. Do not bundle unrelated work; link related issues instead.
- Reference the issue in commits and in the PR body. Use a closing keyword (`Closes #123`, `Fixes #123`, `Resolves #123`) only when the PR fully resolves it; otherwise use `Refs #123`.
- Let the merge close the issue. Do not close it by hand first; verify it is closed after merge.
- A PR links to an issue and closes it automatically only when the PR targets `<default-branch>`; a PR targeting any other branch links nothing and closes nothing.
- Never edit, close, reopen, reassign, or label an issue without explicit user approval. Never rewrite someone else's issue text; comment instead.

### Pull Requests

- Push the branch and open a PR with `gh pr create`.
- **PR title must follow the same commit convention**, because squash merge uses it as the final commit on `<default-branch>`. Example: `fix: prevent null pointer on logout`.
- PR body must contain:
  - **Summary** — what this changes and why.
  - **Changes** — bullet list of the concrete edits.
  - **Testing** — the exact commands run and their result.
  - **Issues** — `Closes #123` when applicable.
- Do not open a draft PR unless asked. Do not open a PR before the test suite passes.
- Address review feedback with new commits; do not force-push a PR under review unless asked.

### Merging

- Repository owners merge with **squash and merge** only (`gh pr merge --squash`), producing exactly one commit on `<default-branch>`.
- Never use a merge commit or rebase merge. Never push directly to `<default-branch>`.
- Before merging, the PR title must be a valid Conventional Commit message and the CI must be green.
- Delete the source branch after merging.

## 2. Testing

- The repository has a test suite. Run it before **every** commit and **every** push.
- All tests must pass. A failing suite blocks the commit, the push, and the PR.
- Never delete, skip, weaken, or comment out tests to make a suite pass. Fix the cause.
- Add tests for every new behavior and every bug fix.
- If a test failure is pre-existing and unrelated, report it explicitly instead of ignoring it.
- Do not commit code you know to be broken.

## 3. Versioning and Releases

- Releases are automated with semantic-release driven by the commit prefixes on `<default-branch>`.
- **Never edit version numbers, `CHANGELOG.md`, git tags, or GitHub releases by hand.**
- The release level is determined by the merged commit prefix: `feat` → minor, `fix`/`perf` → patch, `BREAKING CHANGE:` → major.
- Therefore commit and PR prefixes must be accurate. An incorrect prefix ships an incorrect version.
- Never create or push tags manually.

## 4. Required Tooling

- Use `git` for all version control operations.
- Use `gh` for all GitHub operations: repositories, issues, PRs, reviews, releases, CI status.
- Do not call the GitHub API with `curl` or `wget` when `gh` can do the job.
- **If `git` or `gh` is not installed, stop and tell the user to install it**, including the suggested install command. Do not install tooling unless the user approves.
- Confirm authentication with `gh auth status` before GitHub operations; if unauthenticated, tell the user to run `gh auth login`.

## 5. Reporting

- State the branch, the commit hashes, and the PR URL when reporting completed work.
- Report test commands and their results verbatim.
- Never claim a task is complete when tests fail, CI is red, or a step was skipped.
