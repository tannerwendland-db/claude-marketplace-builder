# Claude Code Skills Marketplace

This is {{ORG_NAME}}'s private Claude Code skills marketplace. It contains multiple skill plugins organized by domain.

## Project Structure

```
.claude-plugin/
  marketplace.json                 Root marketplace catalog
plugins/
  databricks-skills/               Databricks workflow skills
    .claude-plugin/plugin.json
    skills/
      databricks-workspace-files/  Workspace file explorer (with scripts/)
      databricks-lineage/          Unity Catalog lineage tracer (with scripts/)
  internal-skills/                 Internal workflow & productivity skills
    .claude-plugin/plugin.json
    skills/
      onboarding/                  New hire setup guide (template)
      incident-response/           Incident triage & response (template)
  marketplace-management/          Marketplace self-management
    .claude-plugin/plugin.json
    skills/
      update-skills/               Pull latest and re-register plugins
  specialized-tools/               Specialized utility tools
    .claude-plugin/plugin.json
    skills/
      lucid-diagram/               Diagram generation (with scripts/ and references/)
evals/
  src/skill_evals/                 Python eval runner package (Agent SDK)
  scripts/
    generate-routing-tests.py      Generates routing YAMLs from per-skill evals.json
  test-cases/                      Generated routing test YAMLs (do not edit manually)
    all.yaml                       Full catalog (default for `make evals`)
    databricks-skills.yaml         Per-plugin test cases
    internal-skills.yaml
    marketplace-management.yaml
    specialized-tools.yaml
  pyproject.toml                   uv + hatchling config
.claude/
  skills/
    build-skill/SKILL.md           Repo-scoped authoring tool (NOT distributed)
    skill-creator/                 Anthropic's skill authoring + eval tooling
    staging/                       Staging area for in-progress skills
templates/
  basic-skill/                     Simple skill template (no scripts)
  advanced-skill/                  Full skill template (scripts + references)
scripts/
  init.sh                          One-time setup — replaces {{placeholders}}
  install.sh                       End-user install and update
  update.sh                        Safe update from within Claude Code
  validate-skill.sh                Validates skill structure and frontmatter
docs/
  INSTALL.md                       Installation guide
  SKILL-AUTHORING.md               Skill authoring guide
  CONTRIBUTING.md                  Contributing guidelines
Makefile                           make targets for evals, validation, install
```

## Adding a Skill

Use `/build-skill` to create a new skill — it runs the full **Stage → Eval Loop → Promote** pipeline end-to-end:

1. **Phase 1**: Requirements gathering + scaffold into `.claude/skills/staging/<skill-name>/`
2. **Phase 2**: Create `evals/evals.json` + run `skill-creator` routing check (single-pass, then optional optimization)
3. **Phase 3**: Move to target plugin + regenerate routing YAMLs + run plugin-scoped evals + version bump + commit

Or manually:

1. Pick the target plugin under `plugins/`
2. Copy a template: `cp -r templates/basic-skill/ plugins/<plugin>/skills/<name>/`
3. Rename: `mv plugins/<plugin>/skills/<name>/SKILL.md.template plugins/<plugin>/skills/<name>/SKILL.md`
4. Edit the SKILL.md — fill in frontmatter and content
5. Validate: `bash scripts/validate-skill.sh plugins/<plugin>/skills/<name>`
6. **Create `evals/evals.json`** (see "Eval Requirements" below)
7. Run `make evals-generate` to update routing YAMLs
8. Bump the version in the plugin's `plugin.json` and root `marketplace.json`

## Adding a Plugin

To add a new plugin group (e.g., `plugins/security-skills/`):

1. Create the directory: `mkdir -p plugins/security-skills/.claude-plugin plugins/security-skills/skills plugins/security-skills/commands`
2. Create `plugins/security-skills/.claude-plugin/plugin.json` (copy from an existing plugin)
3. Add an entry to `.claude-plugin/marketplace.json` in the `plugins` array
4. Add the plugin's `plugin.json` path to the `FILES_TO_REPLACE` array in `scripts/init.sh`
5. Add the plugin to the `PLUGINS` list in `Makefile`
6. Add skills under `plugins/security-skills/skills/`

## Skill Frontmatter

Every `SKILL.md` must start with YAML frontmatter:

```yaml
---
name: my-skill-name          # kebab-case, required
description: >                # required — Claude uses this to decide when to load the skill
  What this skill does and when to use it.
user-invocable: true          # set true for /slash-command access
allowed-tools: Read, Bash     # optional — tools allowed without confirmation
---
```

## Testing Locally

```bash
claude plugin marketplace add .
claude plugin install {{ORG_SLUG}}-databricks-skills@{{ORG_SLUG}}-marketplace
claude plugin install {{ORG_SLUG}}-internal-skills@{{ORG_SLUG}}-marketplace
claude plugin install {{ORG_SLUG}}-marketplace-management@{{ORG_SLUG}}-marketplace
claude plugin install {{ORG_SLUG}}-specialized-tools@{{ORG_SLUG}}-marketplace
```

Or use `make install-local` after running `scripts/init.sh`.

## Eval Requirements

Every skill **must** have a per-skill eval file at `plugins/<plugin>/skills/<skill-name>/evals/evals.json`.

**Format** (simple JSON array):

```json
[
  {"query": "A realistic prompt that should trigger this skill", "should_trigger": true},
  {"query": "Another phrasing a user would say", "should_trigger": true},
  {"query": "A near-miss prompt that should NOT trigger this skill", "should_trigger": false},
  {"query": "An unrelated prompt that should NOT trigger this skill", "should_trigger": false}
]
```

Rules:
- Minimum **2** `should_trigger: true` + **2** `should_trigger: false` entries
- At least one negative must be a **near-miss** (same domain, wrong intent)
- Positive queries must be **substantive** and realistic — not just the skill name

**Generate routing YAMLs** from per-skill files:

```bash
make evals-generate
```

This produces `evals/test-cases/<plugin>.yaml` and `evals/test-cases/all.yaml`. These files are generated — do not edit them manually.

**Run evals**:

```bash
make evals                          # all skills (uses all.yaml)
make evals PLUGIN=databricks-skills # scoped to one plugin
make evals FILTER=lineage           # filter by test name substring
```

PRs that add or modify skills without a corresponding `evals/evals.json` should not be merged. Always run `make evals-generate` after adding or editing `evals/evals.json`.

## Make Targets

| Target | Description |
|--------|-------------|
| `make evals` | Run routing evals (default: all.yaml) |
| `make evals PLUGIN=<name>` | Run evals for one plugin |
| `make evals-generate` | Regenerate routing YAMLs from per-skill evals.json |
| `make evals-check-generated` | CI check: YAMLs are up to date |
| `make evals-install` | Install eval dependencies (`uv sync`) |
| `make validate` | Validate all skill structure and frontmatter |
| `make install-local` | Register marketplace + install all plugins |
| `make init` | One-time repo initialization (replaces placeholders) |

## Version Bumping

When skills change, bump the `version` field in:
- The plugin's `.claude-plugin/plugin.json`
- The root `.claude-plugin/marketplace.json` (matching plugin entry)
