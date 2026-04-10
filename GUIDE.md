# Setup Guide

Everything you need to get the project running. All commands are copy-paste ready.

---

## Step 1 — Accept the GitHub Invite

Check your email for a collaboration invite from GitHub and accept it. Then clone the repo:

```bash
git clone https://github.com/nshreeyut/hardware-price-sentiment.git
cd hardware-price-sentiment
```

---

## Step 2 — Install Prerequisites

You need **Python 3.12+**, **uv**, and **Google Chrome**.

**Check Python:**
```bash
python3 --version
```
If missing or below 3.12, download from https://www.python.org/downloads/

**Install uv** (the package manager for this project):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then restart your terminal, or run:
```bash
source ~/.zshrc
```

**Verify:**
```bash
uv --version
```

---

## Step 3 — Install All Dependencies

Make sure you are inside the cloned repo folder before running anything here. After Step 1 you should already be there, but double-check:

```bash
cd hardware-price-sentiment
```

Then install everything:
```bash
uv sync
```

This reads `pyproject.toml` and installs everything — pandas, sklearn, spaCy, selenium, PRAW, VADER, and more. It creates an isolated virtual environment so nothing touches your system Python.

Then download the spaCy English model:
```bash
uv run python -m spacy download en_core_web_sm
```

---

## Step 4 — Verify Everything Works

```bash
uv run python -c "import pandas; import sklearn; import praw; import spacy; from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer; print('All good')"
```

You should see: `All good`

---

## Step 5 — Open JupyterLab

Make sure you are in the repo root (`hardware-price-sentiment/`), then run:

```bash
uv run jupyter lab
```

This opens JupyterLab in your browser with the full project already visible in the left-hand file browser. Open the `notebooks/` folder and click on the notebook assigned to you (see README.md for role assignments). Everything — data folders, other notebooks — is right there in the sidebar.

When you are done working, save your notebook (`Ctrl+S` / `Cmd+S`) and follow Step 7 to push your changes.

---

## Step 6 — Reddit API Credentials (Person 2 only)

You need free API credentials to pull Reddit data.

1. Go to https://www.reddit.com/prefs/apps while logged into Reddit
2. Click **"create another app"** at the bottom
3. Fill in:
   - **name:** `hardware_price_tracker`
   - **type:** select **script**
   - **redirect uri:** `http://localhost:8080`
4. Click **Create app**
5. Copy your credentials:
   - The short string directly under your app name → **client_id**
   - The string next to **secret** → **client_secret**

Paste them into the credentials cell in `notebooks/02_reddit_scraping.ipynb`.

**Do not commit credentials to GitHub.** Enter them manually each session, or save them to a `.env` file in the repo root (already in `.gitignore` so it won't be pushed).

---

## Step 7 — Working with Git

Pull the latest changes before starting a session:
```bash
git pull origin main
```

After you've done work, push it:
```bash
git add notebooks/YOUR_NOTEBOOK.ipynb
git commit -m "brief description of what you did"
git push origin main
```

If there's a merge conflict, reach out to the group before force-pushing anything.

**Data files** (`data/` folder) are in `.gitignore` and won't be pushed. Share CSVs with the group via Google Drive or a shared folder if needed.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `uv: command not found` | Restart terminal, or run `source ~/.zshrc` |
| `ModuleNotFoundError` | Run `uv sync` again from inside the repo folder |
| Jupyter kernel not found | Run `uv run python -m ipykernel install --user --name web-mining` |
| Chrome not found by Selenium | Download from https://www.google.com/chrome |
| spaCy model missing | Run `uv run python -m spacy download en_core_web_sm` |
| `git push` rejected | Run `git pull origin main` first, resolve any conflicts, then push |
