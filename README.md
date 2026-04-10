# Hardware Price Tracker + Reddit Sentiment Analysis
### Web Mining — Group Project

---

## Project Overview

We are building a system that tracks computer hardware prices (RAM, GPUs) over time and correlates those price movements with sentiment expressed in Reddit hardware communities. The goal is to answer:

> **Do Reddit discussions about hardware reflect or predict price movements — and what topics dominate during price spikes?**

This is timely given recent hardware market volatility driven by AI chip demand, supply chain shifts, and tariff uncertainty.

---

## Research Question

Does sentiment in hardware subreddits (r/buildapc, r/hardware, r/pcmasterrace) **lead, lag, or correlate** with weekly price fluctuations in consumer RAM and GPU products?

---

## Folder Structure

```
MiningProject/
├── README.md               ← this file
├── data/
│   ├── prices/             ← raw scraped price data (CSV)
│   ├── reddit/             ← raw Reddit posts/comments (CSV)
│   └── processed/          ← cleaned, merged, analysis-ready data
└── notebooks/
    ├── 01_price_scraping.ipynb
    ├── 02_reddit_scraping.ipynb
    ├── 03_preprocessing_sentiment.ipynb
    └── 04_topic_modeling_correlation.ipynb
```

---

## Team Roles

### Person 1 — Price Scraping
**Notebook:** `notebooks/01_price_scraping.ipynb`

**Responsibilities:**
- Scrape historical and current prices for target hardware (DDR5 RAM, mid-range GPUs) from PCPartPicker and/or Newegg using BeautifulSoup and Selenium
- Collect: product name, price, date, retailer, product URL
- Clean and normalize price data into a weekly time series
- Output: `data/prices/prices_clean.csv`

**Key columns in output:**
`date | product | category | price_usd | retailer | url`

---

### Person 2 — Reddit Scraping
**Notebook:** `notebooks/02_reddit_scraping.ipynb`

**Responsibilities:**
- Use PRAW (Python Reddit API Wrapper) to pull posts and top-level comments from:
  - r/buildapc
  - r/hardware
  - r/pcmasterrace
- Filter by relevant keywords: "RAM price", "DDR5", "GPU price", "memory prices", "graphics card price"
- Collect: post title, body, top comments, upvotes, timestamp, subreddit
- Match date range to price data coverage
- Output: `data/reddit/reddit_raw.csv`

**Key columns in output:**
`post_id | date | subreddit | title | body | top_comments | upvotes | url`

**Note:** You will need to register a Reddit app to get API credentials (free). See: https://www.reddit.com/prefs/apps

---

### Person 3 — Preprocessing + Sentiment Analysis
**Notebook:** `notebooks/03_preprocessing_sentiment.ipynb`

**Responsibilities:**
- Load `data/reddit/reddit_raw.csv`
- Preprocess text: tokenize, lowercase, remove stopwords, lemmatize (spaCy/NLTK)
- Score sentiment per post using VADER (handles informal/internet text well)
- Aggregate sentiment scores to a weekly average aligned with price dates
- Output: `data/processed/sentiment_weekly.csv`

**Key columns in output:**
`week | subreddit | avg_sentiment | post_count | avg_upvotes`

---

### Person 4 — Topic Modeling + Correlation + Visualizations
**Notebook:** `notebooks/04_topic_modeling_correlation.ipynb`

**Responsibilities:**
- Load `data/processed/sentiment_weekly.csv` + `data/prices/prices_clean.csv`
- Run LDA topic modeling on Reddit posts — find dominant topics during price spike weeks vs. stable weeks
- Compute Pearson/Spearman correlation between weekly sentiment and price
- Test lagged correlations (does sentiment this week predict price next week, or vice versa?)
- Build visualizations: dual-axis price + sentiment timeline, topic distribution heatmap, correlation plot
- Summarize findings

---

## Techniques Used (Course Alignment)

| Technique | Course Unit | Applied In |
|---|---|---|
| Web scraping (BeautifulSoup, Selenium) | Web Scraping I & II | Notebook 01, 02 |
| Text preprocessing (tokenize, lemmatize) | Preprocessing | Notebook 03 |
| Sentiment analysis (VADER) | Preprocessing / Classification | Notebook 03 |
| TF-IDF + Classification | Classification | Optional extension |
| LDA Topic Modeling | Clustering / Topic Modeling | Notebook 04 |
| Correlation analysis | Applied stats | Notebook 04 |

---

## Target Hardware (Starting Point)

- **RAM:** DDR5-6000 16GB kits (e.g. Corsair Vengeance, G.Skill Trident Z5)
- **GPU:** RTX 4060, RX 7600 (mid-range — most discussed by consumers)

Can expand to more SKUs once pipeline is working.

---

## Setup Guide

Follow these steps exactly to get the project running on your machine. Every command here is copy-paste ready.

---

### Step 1 — Install Prerequisites

You need **Python 3.12+**, **uv** (our package manager), and **Google Chrome** installed.

**Check if Python is already installed:**
```bash
python3 --version
```
If you get `command not found` or a version below 3.12, download it from https://www.python.org/downloads/

**Install uv** (handles all Python packages for this project):
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
Then restart your terminal, or run:
```bash
source ~/.zshrc
```

**Verify uv installed:**
```bash
uv --version
```

---

### Step 2 — Clone the Repo

```bash
git clone https://github.com/YOUR_ORG/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME
```
> Replace `YOUR_ORG/YOUR_REPO_NAME` with the actual GitHub URL the repo owner shares with you.

---

### Step 3 — Install All Dependencies

From inside the repo folder, run:
```bash
uv sync
```

This reads `pyproject.toml` and installs everything — pandas, sklearn, spaCy, selenium, PRAW, VADER, etc. It creates an isolated virtual environment automatically so nothing touches your system Python.

Then download the spaCy English language model:
```bash
uv run python -m spacy download en_core_web_sm
```

---

### Step 4 — Install ChromeDriver (for Selenium)

Selenium needs ChromeDriver to control Chrome. Install it with:
```bash
uv run python -m pip install webdriver-manager
```

The notebooks already handle driver setup using this — no manual ChromeDriver download needed.

---

### Step 5 — Open Jupyter and Run Your Notebook

```bash
uv run jupyter notebook
```

This opens Jupyter in your browser. Navigate to `notebooks/` and open the notebook assigned to you.

> **If you prefer VS Code:** open the repo folder in VS Code, install the Jupyter extension, then open any `.ipynb` file directly. When prompted to select a kernel, choose the one that says `web-mining` or points to `.venv`.

---

### Step 6 — Reddit API Setup (Person 2 only)

You need free API credentials to pull Reddit data.

1. Go to https://www.reddit.com/prefs/apps while logged into Reddit
2. Click **"create another app"** at the bottom
3. Fill in:
   - **name:** `hardware_price_tracker`
   - **type:** select **script**
   - **redirect uri:** `http://localhost:8080`
4. Click **Create app**
5. You'll see a page with your credentials — copy:
   - The short string under your app name → that is your **client_id**
   - The string next to **secret** → that is your **client_secret**

Open `notebooks/02_reddit_scraping.ipynb` and paste them into the credentials cell. **Do not commit these to GitHub** — just enter them manually each session or store them in a local `.env` file (already in `.gitignore`).

---

### Step 7 — Verify Everything Works

Run this to confirm your environment is set up correctly:
```bash
uv run python -c "import pandas; import sklearn; import praw; import spacy; from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer; print('All good')"
```

You should see: `All good`

---

### Troubleshooting

| Problem | Fix |
|---|---|
| `uv: command not found` | Restart terminal after installing uv, or run `source ~/.zshrc` |
| `ModuleNotFoundError` | Run `uv sync` again from inside the repo folder |
| Jupyter kernel not found | Run `uv run python -m ipykernel install --user --name web-mining` |
| Chrome not found by Selenium | Install Google Chrome from https://www.google.com/chrome |
| spaCy model missing | Run `uv run python -m spacy download en_core_web_sm` |

---

## Dependencies

## Status

- [ ] Person 1: Price scraping pipeline
- [ ] Person 2: Reddit scraping pipeline
- [ ] Person 3: Preprocessing + sentiment
- [ ] Person 4: Topic modeling + correlation

---

## Next Steps

1. Each person reviews this README and confirms their role
2. Repo owner adds collaborators on GitHub
3. Each person works in their assigned notebook
4. Data handoffs happen through the `data/` folders — agree on column names before starting
5. Final integration in Notebook 04

---

*Web Mining Group Project — Spring 2026*
