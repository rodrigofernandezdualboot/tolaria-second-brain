---
type: Note
related_to: "[[run-with-hal]]"
status: Active
url: https://apps.apple.com/us/app/run-with-hal/id1437166081
---

# Run with Hal — App Store Review Theme & Sentiment Analysis

Analysis of **400 US App Store reviews** (July 2021 → July 2026, the full history available via Apple's public RSS review feed) for [[run-with-hal]]. Sentiment scored per review with VADER (-1 most negative … +1 most positive); themes assigned by keyword matching on title + review text (a review can match multiple themes).

**Data & code** (in `~/Dualboot/Estimates/Run-with-Hal/`):
- `reviews.csv` — raw reviews (date, userName, title, review, rating)
- `reviews_analyzed.csv` — adds `sentiment`, `sentiment_label`, `themes` per review
- `app_store_reviews.py` — fetcher (direct Apple API/RSS; the `app_store_scraper` library is broken/unmaintained)
- `analyze_reviews.py` — theme keywords + sentiment scoring

## Overall picture

- **Avg rating 3.98★, avg sentiment +0.62** — 339 positive, 50 negative, 11 neutral reviews
- Sentiment correlates well with star rating (r = 0.66), so the score is a trustworthy signal
- Rating distribution: 240×5★, 43×4★, 32×3★, 39×2★, 46×1★

## Themes, ranked worst to best sentiment

| Theme | Reviews | Avg sentiment | Avg rating | % negative |
|---|---|---|---|---|
| Customer support | 25 (6%) | **+0.17** | **2.4★** | 40% |
| Bugs / crashes / lost runs | 18 (5%) | +0.31 | 2.0★ | 33% |
| Watch/Garmin/Strava integration | 47 (12%) | +0.50 | 3.2★ | 13% |
| Pricing / subscription | 41 (10%) | +0.52 | 3.2★ | 20% |
| GPS / run tracking | 91 (23%) | +0.61 | 3.9★ | 12% |
| Training plans | 313 (78%) | +0.65 | 4.0★ | 10% |
| UI / ease of use | 50 (13%) | +0.68 | 4.2★ | 12% |
| Coaching / motivation ("Hal") | 166 (42%) | **+0.70** | **4.3★** | 8% |

**Key takeaway: the product's core is loved; the software around it is not.** Training plans dominate the conversation (78% of reviews) and score well, and the coaching/motivation angle is the single most positive theme. The pain points are operational:

- **Customer support** (40% negative) — unanswered emails, refund disputes, a recent review calling the free trial "a scam"
- **Bugs** — runs failing to save at the end of a workout, forcing manual entry (also the top complaint in the newest reviews)
- **Device sync** — Apple Watch / Garmin / Strava integration friction

## Trend: sentiment is deteriorating

| Year | Reviews | Avg rating | Avg sentiment |
|---|---|---|---|
| 2021 | 45 | 4.27★ | +0.68 |
| 2022 | 107 | 3.74★ | +0.57 |
| 2023 | 85 | 4.28★ | +0.72 |
| 2024 | 99 | 4.30★ | +0.67 |
| 2025 | 56 | 3.38★ | +0.52 |
| 2026 (8 reviews) | 8 | 2.62★ | **-0.04** |

After holding at ~4.3★ / +0.7 through 2023–2024, reviews dropped sharply in 2025–2026. Recent negatives (13 negative reviews since Jan 2025) cluster on:

1. **Plan inflexibility** — can't set a target race pace; automatic adjustments "seem really off"
2. **A disliked app redesign** — "updates ruined the app"
3. **Subscription / free-trial billing complaints**

Small sample for 2026, but the direction is consistent.
