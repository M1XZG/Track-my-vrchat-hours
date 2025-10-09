# Track playtime hours from Steam in your README

This repository contains a small automation that fetches your playtime for a Steam game (for example, VRChat) and updates a section in your repository README with the latest hours. It works locally or, more commonly, via GitHub Actions on a schedule.

It’s generic: you can use it for any Steam app by providing the Steam App ID. The repo includes ready-to-use workflows, scripts, and a markdown “insertion” helper to place your hours wherever you want in a README template.

---

## What you’ll get

- Automatic fetch of your total playtime from the Steam Web API
- A formatted line like: “As of 2025-10-09 @ 13:37 UTC — 1,234.5 lifetime hrs”
- A pull request that updates your README whenever the hours change
- Optional verbose workflow and a local runner script

---

## Prerequisites

- A GitHub repository with a README.md (this repo can be your profile README or any project README)
- A Steam Web API key
- Your SteamID64 (64-bit Steam ID)
- The Steam App ID of the game you want to track (VRChat is 438100)
- GitHub Personal Access Token (classic) with repo permissions (for the workflow to push branches and PRs)

Optional for local runs:

- Python 3.8+
- pip to install Python packages
- GitHub CLI (gh) authenticated to your account

---

## Find your Steam credentials

- Steam Web API key:
  - Visit the Steam Web API key page while logged into your Steam account: [steamcommunity.com/dev/apikey](https://steamcommunity.com/dev/apikey)
  - Register a domain (can be localhost) and copy your key

- SteamID64:
  - If your profile has a custom URL, use a lookup service like [steamid.io](https://steamid.io) to convert to the 64-bit ID
  - Or open your Steam profile in a browser and copy the numeric ID from the URL when not using a custom URL

- Steam App ID:
  - Look up the game on the Steam store and check the number in the URL, or search on [steamdb.info](https://steamdb.info)
  - Example: VRChat is 438100

---

## Quick start (GitHub Actions)

Follow these steps to get the hourly update workflow running in your own repository.

1. Fork or copy this repository into your GitHub account.

2. Prepare your README template and snippet placement:

   - Copy your current README content into a template file the scripts expect.
   - Append the snippet block that marks where to insert your playtime.

   On macOS (zsh), from the repo root:

```zsh
# Create the template folder if needed
mkdir -p templates

# Copy your existing README into the template path the script expects
cp README.md templates/README-template.md

# Append the insertion snippet to the end of the template
# You can later move this block within the file to where you want it displayed
# Use the clean snippet file to insert the playtime block
cat templates/README-template-snippet.clean.md >> templates/README-template.md
```

   The snippet looks like this and defines the insertion block:

   <!-- start myhoursHERE -->
   <!-- end myhoursHERE -->

   Place that comment block exactly where you want the hours to appear in `templates/README-template.md`.

3. Configure repository secrets (GitHub → Settings → Secrets and variables → Actions → New repository secret):

   - `MY_GH_PAT` — Your GitHub Personal Access Token with repo scope
   - `STEAM_API_KEY` — Your Steam Web API key
   - `STEAM_USER_ID` — Your SteamID64 (example: 76561198000000000)
   - `STEAM_GAME_ID` — Steam App ID to track (e.g., 438100 for VRChat)

4. (Optional but recommended) Update the git identity used by the workflow:

   - In `.github/workflows/update-vrchat-hours-v2.yaml` and/or `update-vrchat-hours-v2-verbose.yaml`, change the “Set up git user” step to your details, for example:

     ```yaml
     - name: Set up git user
       run: |
         git config --global user.email "123456+yourusername@users.noreply.github.com"
         git config --global user.name "Your Name"
     ```

5. Enable the workflow and run it:

- The non-verbose workflow is scheduled hourly by default. You can also trigger it manually via the “Run workflow” button in the Actions tab.
- Adjust the schedule by editing the `cron` expression in `.github/workflows/update-vrchat-hours-v2.yaml`.

That’s it. When your hours change, the action will open a PR that updates `README.md` with the new value. If there’s no change, it exits without committing.

---

## How it works

- `scripts/update-myhours-workflow-v2.py` fetches your playtime via the Steam API, prepares a formatted line, and inserts it into a temporary README built from your template.
  - It reads credentials from a small `steam_vars.txt` file (the workflow creates this file on the fly from repository secrets).
  - It uses a very small helper (`minsert`) to replace the content between `<!-- start myhoursHERE -->` and `<!-- end myhoursHERE -->`.
  - By default, it applies an offset for “missing hours” you may want to reclaim from prior tracking (`OFFSET_MISSING_HOURS = 13.0`). You can change this constant in the script.

- `scripts/update-vrchours-workflow-v2.sh` coordinates the update:
  - Calls the Python script
  - Compares the current hours in `README.md` with the hours in `TMP-README.md`
  - If different, creates a branch, commits, pushes, opens a PR, and enables auto-merge

- Workflows in `.github/workflows/` wire everything together for GitHub Actions.

---

## Manual/local usage (optional)

If you prefer to run the update locally for testing:

1. Create the template as described above (ensure `templates/README-template.md` exists and contains the snippet block).

1. Install Python requirements:

```zsh
python3 -m venv .venv
source .venv/bin/activate
pip install -r scripts/requirements.txt
```

1. Create `steam_vars.txt` in the repository root:

```zsh
cat > steam_vars.txt << 'EOF'
STEAM_API_KEY=your_steam_api_key_here
STEAM_ID=your_steamid64_here
EOF
```

1. Run the shell script with your repo path and the game ID:

```zsh
./scripts/update-vrchours-workflow-v2.sh "$PWD" 438100
```

This will create `TMP-README.md` and, if hours changed, commit and open a PR (requires `gh` CLI authenticated to your GitHub account).

For more logging, use:

```zsh
./scripts/update-vrchours-workflow-v2-verbose.sh "$PWD" 438100
```

---

## Customization and tips

- Track any Steam game by changing `STEAM_GAME_ID` (examples: VRChat `438100`, your game’s App ID from store or SteamDB)
- Want a different schedule? Update the `cron` in the YAML workflow
- Don’t want auto-merge? Remove `gh pr merge --auto -m` in the shell script
- Use your GitHub no-reply email in the workflow to avoid exposing a personal email
- If you have ever-tracked hours outside Steam’s reporting, adjust `OFFSET_MISSING_HOURS` in `scripts/update-myhours-workflow-v2.py`

---

## File map

- `.github/workflows/update-vrchat-hours-v2.yaml` — Scheduled workflow (hourly by default)
- `.github/workflows/update-vrchat-hours-v2-verbose.yaml` — Manual, verbose workflow
- `scripts/update-vrchours-workflow-v2.sh` — Main orchestration script
- `scripts/update-vrchours-workflow-v2-verbose.sh` — Verbose variant
- `scripts/update-myhours-workflow-v2.py` — Fetches/updates README contents
- `scripts/requirements.txt` — Python dependencies (requests)
- `scripts/python-requirements/minsert.py` — Minimal markdown insertion helper
- `templates/README-template-snippet.clean.md` — Snippet block to paste where hours should appear
- `templates/README-template.md` — Your template (you create this by copying your README and inserting the snippet)

---

## Troubleshooting

- Steam API error or “Game not found”: confirm `STEAM_API_KEY`, `STEAM_ID`, and `STEAM_GAME_ID` are correct; ensure the game is visible in your owned games and tracked by Steam
- No PR opened: if the hours didn’t change since last run, the script exits quietly; try the verbose workflow to inspect logs
- YAML permission errors: ensure `MY_GH_PAT` has repo permissions and the workflow can push to your repository
- Insertion didn’t show up: confirm your `templates/README-template.md` contains the `<!-- start myhoursHERE -->` and `<!-- end myhoursHERE -->` markers

---

## Security notes

- Store secrets only in GitHub Actions → Secrets. Never commit them to the repo.
- Use your GitHub no-reply email in the workflow to avoid exposing a personal email.

---

## License

This project is provided as-is under a permissive license. See the repository for details.
