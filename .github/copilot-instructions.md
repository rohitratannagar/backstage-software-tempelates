**Overview**
- **Purpose:** This repository is a Backstage scaffolder template (see [template.yaml](template.yaml)). It provides a set of scaffolder steps that generate and publish a new GitHub repository and then register it with Backstage catalog.

**Key files**
- **template.yaml:** Primary scaffolder definition. It declares parameters for the form, the series of `steps`, and the templating expressions (e.g. `
  ${{ parameters.name }}`).
- **template/**: Expected source template folder referenced by `fetch:template` (currently empty in this repo). Populate it with the files you want copied into generated repos.
- **node-app/**: placeholder directory; nothing required here for scaffolder behavior.

**Big picture architecture & flow**
- The scaffolder runs steps in order. This repo uses these action types: `fetch:template`, `fetch:plain`, `publish:github`, `catalog:register` (see [template.yaml](template.yaml)).
- Typical flow in this template:
  1. `fetch:template` pulls files from `./template` into the generated repo.
  2. `fetch:plain` can pull external docs (here it targets a Backstage community URL).
  3. `publish:github` creates the GitHub repository and returns `repoContentsUrl`.
  4. `catalog:register` uses that `repoContentsUrl` and expects a `catalog-info.yaml` path to register the component with Backstage.

**Project-specific patterns & conventions**
- Template values use Backstage mustache-style expressions inside `${{ ... }}` (example: `${{ parameters.name }}`).
- Steps frequently reference prior outputs. Example pattern used here: `steps['publish'].output.repoContentsUrl` (used as `repoContentsUrl` when registering the component).
- The scaffolder expects a `catalog-info.yaml` at the repo root by default; the `catalog:register` step in this template points to `/catalog-info.yaml`.
- `RepoUrlPicker` UI: `repoUrl` uses `ui:field: RepoUrlPicker` and allows only `github.com` by default (`ui:options.allowedHosts`).
- Default branch: `publish:github` uses `defaultBranch: 'main'`—ensure generated repos align with this.

**Integration & infra notes**
- `publish:github` requires credentials with repo creation privileges in the Backstage environment where this template runs.
- `catalog:register` interacts with the Backstage catalog backend — registration will only succeed if the Backstage instance can access the generated repo contents URL.

**Developer workflows / quick checks (discoverable from repo)**
- Validate YAML syntax locally: `python -c "import yaml,sys; yaml.safe_load(sys.stdin)" < template.yaml` or use `yq`/`yamllint` as available.
- To test this template you will typically:
  - Populate `template/` with the scaffold source files (including a `catalog-info.yaml`).
  - Run it from a Backstage instance that has the scaffolder and GitHub publisher configured.
- There are no repository-local npm / build scripts or tests present in this repo — scaffolder behavior runs inside Backstage, not locally in this repo.

**Concrete examples from this repo**
- `publish` step (from template.yaml) sets `repoUrl: ${{ parameters.repoUrl }}` and `defaultBranch: 'main'` — ensure callers pass a GitHub URL and that the publisher is configured to use that host.
- `register` step uses `repoContentsUrl: ${{ steps['publish'].output.repoContentsUrl }}` and `catalogInfoPath: '/catalog-info.yaml'` — a generated repository MUST contain `catalog-info.yaml` at the path referenced.

**What an AI coding agent should do first**
- Inspect `template.yaml` to understand available parameters and steps.
- If asked to modify or extend scaffolding behavior, ensure changes are reflected both in `template.yaml` and by adding/updating files under `template/`.
- When adding new `publish` or `catalog` behavior, reference existing step-output patterns (e.g., `steps['<id>'].output.<key>`) for wiring later steps.

If anything here is unclear or you'd like the instructions adjusted to include run/test commands for a specific Backstage environment, tell me which environment and I will update this file.
