# Central agentic workflow catalog

A library maintained by a Platform or AI Engineering team, distributing one
reusable TypeScript / Node.js PR reviewer. Each consuming repository executes its
own installed copy with its own repository context and Actions secret.

```text
Central repo → gh aw add → application repository → compiled GitHub Actions workflow
```

## Package

`aw.yml` uses `includes` with `kind: agentic-workflow` to install
`payload/workflows/pr-reviewer.md` as `.github/workflows/pr-reviewer.md`.
The source remains inert here, outside `.github/workflows`. This repository does
not run an agent remotely against application repositories.

Tested with `gh-aw v0.89.21`; the manifest requires that version or newer. The
deprecated `files` manifest field is not used.

## pr-reviewer

Codex reads the PR diff, changed files, and relevant code/tests at the head SHA.
It reports actionable behavior, security, API-contract, and coverage problems,
ignoring cosmetic style. Each finding includes Problem, Impact, and Recommended
Fix, followed by an AI Review Summary. This is static review; application CI
executes tests and builds.

Runs on same-repository PRs opened, reopened, or synchronized. Manual dispatch
accepts a `pr_number`. Fork PRs are excluded. Approved-content filtering applies;
use a PR authored by a repository writer for the demo.

## Installation

Publish this repository's `main` branch first. From the consuming repository:

```sh
gh aw add AakashPandya/gh-aw-demo-workflows@main
gh aw compile
gh aw validate
gh aw validate --strict
git add .github .gitattributes
git commit -m "Install central agentic PR reviewer"
```

Keep all generated source/package tracking, action pins, and `.lock.yml` files.
GitHub Actions executes the lockfile: commit it together with its Markdown.
Do not hand-copy the source or hand-edit generated YAML. For production, select
a catalog release/tag explicitly and review updates before adoption.

## Updates

From the consuming repository:

```sh
gh aw update pr-reviewer
gh aw validate --strict
git diff
```

The CLI fetches the recorded source, merges local modifications, and recompiles.
Review and commit the changes. Catalog updates do not instantly rewrite consumers.

## Security

- Agent permissions: `contents: read` and `pull-requests: read` only.
- GitHub tools: `pull_request_read` and `get_file_contents`; shell disabled.
- Reads restricted to the current repository, with an explicit network allowlist.
- One `add-comment` per run, targeted to the specified PR; at most four findings.
- No approval, merge, code-push, branch, release, or settings output.
- The compiler creates a separate writeback job. In this version its comment
  handler requires `issues: write` and `pull-requests: write`; the agent never
  receives those permissions. Conclusion jobs also use compiler-managed reporting
  permissions. Failure issue reporting is disabled.
- Manually configure only `OPENAI_API_KEY` in the consuming repository's Actions
  secrets. `CODEX_API_KEY` is a supported alternative; only one is needed. No
  model override is configured, and no secret value belongs in Git.

Initial compilation may warn about the new Codex secret references. They are
expected runtime authentication. Review the official SHA-pinned actions and
generated references before committing. Safe outputs constrain mutations; an
engineer still verifies findings and makes the merge decision.

## Author validation

These commands validate the root manifest and inert workflow source:

```sh
gh aw compile --dir payload/workflows --no-emit
gh aw validate --dir payload/workflows
gh aw validate --strict --dir payload/workflows
```

Omit `--no-emit` to inspect a generated lockfile; the inert payload lock is ignored
by Git. In v0.89.21 `validate` skips external linters in its no-emit path. To
request those checks, use `gh aw compile --validate --actionlint --zizmor --poutine
--dir payload/workflows`. Its container-based linters require Docker, which is
outside this demo's setup. CLI validation is not an end-to-end Actions run.

## Official references

- [Package schema](https://github.github.com/gh-aw/reference/aw-yml-package-manifest/)
- [Package specification](https://github.github.com/gh-aw/specs/repository-package-manifest-specification/)
- [Codex configuration and authentication](https://github.github.com/gh-aw/engines/codex/)
- [Safe outputs](https://github.github.com/gh-aw/reference/safe-outputs/)
- [GitHub read tools](https://github.github.com/gh-aw/reference/github-tools/)

Inspect `gh aw <command> --help` and revalidate when upgrading the CLI.
