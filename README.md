# FastMCP Project Template

[![CI](https://github.com/ahcodedev/fastmcp-template/actions/workflows/python.yml/badge.svg?branch=main)](https://github.com/ahcodedev/fastmcp-template/actions/workflows/python.yml)
[![Security](https://github.com/ahcodedev/fastmcp-template/actions/workflows/security.yml/badge.svg?branch=main)](https://github.com/ahcodedev/fastmcp-template/actions/workflows/security.yml)
[![betterleaks](https://img.shields.io/badge/protected%20by-betterleaks-blue)](https://github.com/betterleaks/betterleaks)
[![zizmor](https://img.shields.io/badge/%F0%9F%8C%88-zizmor-white?labelColor=white)](https://zizmor.sh/)

FastMCP project template

## Setup

Project supports local development and VSCode Dev container development

### Local Development Installation

Note: Project uses `uv` and `just` to
manage project configuration and commands

* Install tools and sync the project - `just setup` (runs `brew bundle install`, then `just sync`)

### VS Code Dev Container

* Install Docker
* Install VSCode
* Open project folder in VS Code
* Install recommended extensions
* Reopen in container using Remote Development Container menu

## Project Commands

* `just setup` - install Homebrew tools from the `Brewfile` and sync the project
* `just sync` - sync project dependencies (all groups)
* `just lint` - type check, lint, format, spell check (typos), and lint workflows (actionlint), YAML (yamllint),
  and JSON (jq) (applies fixes)
* `just ci-lint` - run the same checks without modifying files, except spell check (used in CI)
* `just spellcheck` - check spelling with typos
* `just test` - run unit tests
* `just coverage` - run unit tests with coverage (fails under 98%)
* `just audit` - check dependencies for known vulnerabilities with pip-audit
* `just markdownlint` - lint Markdown files with markdownlint-cli2
* `just docs-build` - build the docs site with Zensical (strict mode, output in `site/`)
* `just run` - run the MCP server
* `just dev` - run the server with the MCP Inspector (requires Node / `npx`)
* `just docs-serve` - serve the docs locally with live reload
* `just prek` - run pre-commit hooks via prek
* `just betterleaks` - scan staged changes for secrets with betterleaks
* `just zizmor` - audit GitHub Actions workflows for security issues
* `just set-github-labels` - remove GitHub's non-release default labels, then create or update the release labels
  (for fresh repos, requires `gh auth login`)
* `just update-template` - pull in the latest changes from the Copier template
* `uv add {package}` - install a dependency

## CI and Security

| Workflow              | Runs on                 | Purpose                                                 |
| --------------------- | ----------------------- | ------------------------------------------------------- |
| `python.yml`          | push / PR to `main`     | lint, spell check, unit tests, docs build, markdownlint |
| `security.yml`        | push to `main`, all PRs | zizmor, betterleaks, pip-audit dependency audit         |
| `release-drafter.yml` | push to `main`, PRs     | update the draft release, label PRs                     |
| `release.yml`         | manual                  | publish a release                                       |

* GitHub Actions are pinned to full commit SHAs, with the version in a trailing comment
* Dependabot (`.github/dependabot.yml`) checks weekly for updates to GitHub Actions, Python
  dependencies, the dev container image, and dev container features. New releases wait 7 days
  before an update is proposed. Dependabot PRs get the `dependencies` label

## Releases

Releases are managed with [Release Drafter](https://github.com/release-drafter/release-drafter),
which reads the **release labels** on merged pull requests.

A release label is a GitHub label added to the pull request itself, from the **Labels** section
of the PR sidebar. Release Drafter only reads these labels. Commit messages and PR titles are not
parsed, so prefixes like `feat:` or `fix:` have no effect. Only the labels below count as release
labels; any other repository labels you add are ignored.

| Release label                                                     | Version bump | Release notes section |
| ----------------------------------------------------------------- | ------------ | --------------------- |
| `breaking`                                                        | major        | Breaking Changes      |
| `feature`, `enhancement`                                          | minor        | Features              |
| `bug`, `fix`                                                      | patch        | Bug Fixes             |
| `chore`, `ci`, `dependencies`, `documentation`, `refactor`        | patch        | Maintenance           |
| `skip-changelog`                                                  | none         | Excluded              |

* Every PR must have at least one release label, or the `Release Drafter / require-label` check fails. If a release
  has PRs with different labels, the largest bump wins
* Release labels are added automatically from the PR's branch name prefix (`feat/`, `fix/`, `chore/`,
  `ci/`, `docs/`, `refactor/`) and from the files it changes. You can add or change them by hand
* Run `just set-github-labels` once to create the release labels in the GitHub repository
* Merging to `main` updates a draft GitHub Release with the next version and notes
* To release, run the **Release** workflow from the Actions tab. It bumps the version in
  `pyproject.toml`/`uv.lock`, tags the release, and uploads build artifacts. Release notes live on the GitHub Release

## Usage

The MCP server is exposed as the `example` script (`[project.scripts]` in `pyproject.toml`), which
runs the server over stdio.

* `uv run example` - run the server from the project
* `uv run python -m example` - run the server as a module
* `just dev` - run the server with the MCP Inspector (requires Node / `npx`)

### Client Configuration

After publishing the package (`uv build`, then `uv publish`), MCP clients can run it with `uvx`
without a separate install step:

```json
{
  "mcpServers": {
    "calculator": {
      "command": "uvx",
      "args": ["example"]
    }
  }
}
```

For Claude Code: `claude mcp add calculator -- uvx example`

Rename the package (`name` in `pyproject.toml`) and the script to something unique before publishing
to PyPI.
