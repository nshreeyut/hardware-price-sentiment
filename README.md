# Hardware Price Tracker + Reddit Sentiment Analysis
### Web Mining — Group Project

> For setup instructions, see [GUIDE.md](GUIDE.md)

---

## Research Question

Does sentiment in hardware subreddits (r/buildapc, r/hardware, r/pcmasterrace) **lead, lag, or correlate** with weekly price fluctuations in consumer RAM and GPU products?

---

## Project Overview

We scrape hardware prices and Reddit discussions in parallel, run NLP analysis on the forum text, then measure whether community sentiment tracks or predicts price movements.

This is timely given recent hardware market volatility driven by AI chip demand, supply chain shifts, and tariff uncertainty.

---

## Folder Structure

```
MiningProject/
├── README.md                              ← this file
├── GUIDE.md                               ← setup instructions
├── data/
│   ├── prices/                            ← raw scraped price data (CSV)
│   ├── reddit/                            ← raw Reddit posts/comments (CSV)
│   └── processed/                         ← cleaned, merged, analysis-ready data
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

Scrape the top 10 Amazon Best Sellers per category (CPU, GPU, RAM) to identify products, match each to Pangoly.com, and pull 6 months of price history across all available retailers (Newegg, BestBuy, B&H, etc.) via Pangoly's internal price-chart API. Aggregate to weekly averages.

**Output:** `data/prices/prices_clean.csv`
**Columns:** `date | product | category | price_usd | retailer | url`

---

### Person 2 — Reddit Scraping
**Notebook:** `notebooks/02_reddit_scraping.ipynb`

Use PRAW (Python Reddit API Wrapper) to pull posts and top-level comments from r/buildapc, r/hardware, and r/pcmasterrace filtered by hardware price keywords. Match the date range to price data coverage.

**Output:** `data/reddit/reddit_raw.csv`
**Columns:** `post_id | date | subreddit | title | body | top_comments | upvotes | url`

**Note:** Requires free Reddit API credentials — see GUIDE.md Step 6.

---

### Person 3 — Preprocessing + Sentiment Analysis
**Notebook:** `notebooks/03_preprocessing_sentiment.ipynb`

Clean Reddit text (tokenize, lemmatize, remove stopwords via spaCy/NLTK), score sentiment per post using VADER, and aggregate to a weekly sentiment index aligned with price dates.

**Input:** `data/reddit/reddit_raw.csv`
**Output:** `data/processed/sentiment_weekly.csv`
**Columns:** `week | subreddit | avg_sentiment | post_count | avg_upvotes`

---

### Person 4 — Topic Modeling + Correlation + Visualizations
**Notebook:** `notebooks/04_topic_modeling_correlation.ipynb`

Run LDA topic modeling on Reddit posts to identify dominant topics during price spike weeks vs. stable weeks. Compute Pearson/Spearman correlation and lagged correlations between weekly sentiment and price. Produce final visualizations.

**Inputs:** `data/processed/sentiment_weekly.csv` + `data/prices/prices_clean.csv`

---

## Techniques Used (Course Alignment)

| Technique | Course Unit | Notebook |
|---|---|---|
| Web scraping (BeautifulSoup, Selenium) | Web Scraping I & II | 01, 02 |
| Text preprocessing (tokenize, lemmatize) | Preprocessing | 03 |
| Sentiment analysis (VADER) | Preprocessing / Classification | 03 |
| LDA Topic Modeling | Clustering / Topic Modeling | 04 |
| Correlation analysis | Applied stats | 04 |

---

## Target Hardware

- **RAM:** DDR5-6000 32GB kits (e.g. Corsair Vengeance, G.Skill Trident Z5)
- **GPU:** RTX 4060, RX 7600 (mid-range — most consumer discussion)

---

## Status

- [x] Person 1: Price scraping pipeline
- [ ] Person 2: Reddit scraping pipeline
- [ ] Person 3: Preprocessing + sentiment
- [ ] Person 4: Topic modeling + correlation

---

*Web Mining Group Project — Spring 2026*
