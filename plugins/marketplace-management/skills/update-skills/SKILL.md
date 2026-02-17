---
name: update-skills
description: >
  Update {{ORG_SLUG}} skills to the latest version. Re-runs the full install
  process: pulls latest changes from the repository, re-registers the
  marketplace, and re-installs all plugins. Use when you want to get the
  latest skills and updates.
user-invocable: true
allowed-tools: Bash, Read
---

# Update Skills

Pull the latest skills from the repository, re-register the marketplace, and re-install all plugins so you're fully up to date.

## Execution

1. Locate the install directory at `~/.claude-skills/{{ORG_SLUG}}`.

2. Verify the install directory exists:
   ```bash
   ls ~/.claude-skills/{{ORG_SLUG}}/scripts/install.sh
   ```
   If it does not exist, tell the user:
   > Skills are not installed yet. Run the install command first:
   > ```
   > curl -sSL {{REPO_URL}}/raw/main/scripts/install.sh | bash
   > ```

3. Capture the current git revision before updating:
   ```bash
   cd ~/.claude-skills/{{ORG_SLUG}} && git rev-parse HEAD
   ```

4. Run the full install script, which handles git pull, marketplace re-registration, and plugin re-installation:
   ```bash
   bash ~/.claude-skills/{{ORG_SLUG}}/scripts/install.sh
   ```

5. Show what changed since the previous revision:
   ```bash
   cd ~/.claude-skills/{{ORG_SLUG}} && git log --oneline <old-rev>..HEAD
   ```

6. Report the results to the user:
   - If the revision changed, list the new commits and summarize what was updated (new skills, modified skills, etc.).
   - If already up to date, confirm that no new changes were found.
