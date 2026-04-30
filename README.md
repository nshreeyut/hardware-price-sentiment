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
- [x] Person 2: Reddit scraping pipeline (see Data Coverage Limitations below)
- [x] Person 3: Preprocessing + sentiment
- [ ] Person 4: Topic modeling + correlation

---

## Data Coverage Limitations (Reddit)

**Final coverage of `data/reddit/reddit_raw.csv`:** 2024-10-01 → 2025-05-19 (3,452 posts across r/buildapc, r/hardware, r/pcmasterrace).

The original target window ran to April 2026, but sourcing recent Reddit data without a Reddit API app turned out to be the bottleneck. Methods attempted:

### 1. Reddit API via PRAW (the standard path)
Requires a Reddit developer-app registration. The application was submitted but did not get approved before the project deadline, so this route was unavailable. PRAW also caps `subreddit.search()` at ~1000 results, which would have been limiting for an 18-month backfill anyway.

### 2. PullPush.io (what produced our actual data)
A public archive that mirrors Reddit and needs no API key. This is what `notebooks/02_reddit_scraping.ipynb` uses, and it produced the 3,452 posts we have. **Limitation discovered during scraping:** PullPush's archive itself stops indexing around **mid-May 2025**. Setting `DATE_END = 2026-04-30` had no effect — the archive simply has no records past that date, so the scrape returned nothing for the ~11-month gap (2025-05-20 → present).

### 3. Reddit's public `.json` endpoints (no auth) — attempted as a backfill
Append `.json` to any Reddit URL — works without an API key. Attempted to fill the gap with this. **Why it did not work in our timeframe:**
- ~100 results per page, ~1000 per search query, no precise timestamp filtering (only coarse `t=year/month/week` buckets)
- Aggressive IP-based rate limiting — hit `HTTP 429` repeatedly even at 1 request per 1.5 seconds
- Reddit blocks the IP for ~10 minutes once the limit trips, requiring 60–300s backoffs between retries
- After tuning sleeps to (3.0, 5.0)s and capping comment fetches to top-10 per query, realistic runtime was 60–90 minutes for the full 78-query grid, still with a high probability of further 429s mid-run

### 4. Options not pursued (would require more time)
- **Arctic Shift** (https://github.com/ArthurHeitmann/arctic_shift) — academic Reddit dump similar to old Pushshift. Coverage of mid-2025 onward is improving but distribution is bulk-monthly archive files, not a queryable API.
- **Reddit OAuth via a script-type app** — does not require app *approval*, only signup. Would give ~100 req/min and make the public-JSON approach viable, but still caps at ~1000 results per search query.

### Impact on the rest of the project
The 7.5-month window we do have (Oct 2024 – May 2025) is enough for a meaningful sentiment-vs-price study, but Person 4's correlation analysis must be constrained to that window — the price data from Person 1 should be filtered to the same date range before merging. Lagged-correlation lookbacks longer than ~4 weeks will lose statistical power near the window edges.

---

*Web Mining Group Project — Spring 2026*
