# Scripts Directory

This directory contains the v2 scripts that power the automated README updates via Steam’s API.

---

## Script Overview

### 1. `update-vrchours-workflow-v2.sh`

- Purpose:
  - Bash script to automate updating hours for a Steam game in the repository.

- How it works:
  - Accepts the repository path and Steam Game ID as arguments.
  - Optionally introduces a random delay when run in "cron" mode.
  - Calls `update-myhours-workflow-v2.py` to fetch playtime data from the Steam API and prepare a temporary README.
  - If the playtime changed, creates a PR with the updated README and enables auto-merge.

- Relation:
  - Used by the GitHub Actions workflow to keep your playtime stats up to date.

### 2. `update-myhours-workflow-v2.py`

- Purpose:
  - Python script that fetches playtime and updates a README template using a small markdown insertion helper.

- How it works:
  - Reads Steam API credentials from `steam_vars.txt` (created by the workflow or you, locally).
  - Fetches playtime for the specified Steam game and formats the output.
  - Inserts the formatted hours into the block between `<!-- start myhoursHERE -->` and `<!-- end myhoursHERE -->` in `templates/README-template.md`, producing `TMP-README.md`.

- Relation:
  - Called by the shell script.

### 3. `python-requirements/`

- Purpose:
  - Directory containing the tiny `minsert` helper used by the Python script.

- How it works:
  - The Python script sets `sys.path` to include this directory.

- Relation:
  - No changes typically required here.

---

## Required Repository Secrets for v2 Workflows

Set these in your repository under Settings → Secrets and variables → Actions:

- `MY_GH_PAT`
  - GitHub Personal Access Token with repo permissions. Used for pushing branches and opening PRs.
- `STEAM_API_KEY`
  - Your Steam Web API key. Used for fetching playtime.
- `STEAM_USER_ID`
  - Your 64-bit SteamID. Used to identify which account’s playtime to query.
- `STEAM_GAME_ID`
  - The Steam App ID to track (e.g., 438100 for VRChat).

---

## How the v2 scripts work together

- The GitHub Actions workflow calls `update-vrchours-workflow-v2.sh`.
- The shell script runs `update-myhours-workflow-v2.py` which builds `TMP-README.md` from your template.
- If hours changed, the shell script commits `README.md` updates on a new branch and opens a PR.

---

## Usage (manual)

From the repository root:

```bash
# Create a steam_vars.txt with your credentials if running locally
cat > steam_vars.txt << 'EOF'
STEAM_API_KEY=your_steam_api_key_here
STEAM_ID=your_steamid64_here
EOF

# Run the update (replace with your Steam App ID)
./scripts/update-vrchours-workflow-v2.sh "$PWD" 438100

# Verbose mode
./scripts/update-vrchours-workflow-v2-verbose.sh "$PWD" 438100
```

> Note:
>
> - The template file `templates/README-template.md` must exist and include the snippet block.
> - In CI, the workflow creates `steam_vars.txt` for you from repository secrets.

---

