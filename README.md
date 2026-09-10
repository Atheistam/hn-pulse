# HN Pulse — Hacker News momentum dataset

**An open, growing time-series dataset of the Hacker News front page.**
Captured automatically every ~3 hours: front page (top 120), Show HN, Ask HN and the
monthly hiring thread — with derived *momentum* measurements that HN's own API does
not expose.

* **Dataset home:** https://atheistam.github.io/hn-pulse/
* **Live dashboard:** https://hnpulse-radar.surge.sh/
* **Status:** 28 snapshots · 1274 distinct stories · 4698 story observations · 6.75 days of coverage
* **Last snapshot:** 2026-09-10T14:00:52Z
* **License:** data CC BY 4.0 · code MIT

## Why this exists

HN's official API gives you a *current* view: this item has N points right now.
It cannot tell you how fast a story is climbing, how long it stays on the front
page, or when during the day the front page actually turns over. Those need
repeated observations of the same items over time. This dataset is that record,
captured continuously since 2026-09-03T20:05:39Z.

## Files

| File | Rows | What it is |
| --- | --- | --- |
| `data/observations.csv` | 4698 | Raw long format: one row per story *per snapshot* (score, comments, list, age at capture). The source of everything else. |
| `data/stories.csv` | 1274 | One row per distinct story: observed span, first/max score, **points gained**, **points per hour**, how many snapshots it survived. |
| `data/snapshots.csv` | 28 | One row per capture: timestamp + counts per list (top / show / ask / hiring). |
| `data/atlas.json` | — | Aggregate study: velocity by story age, by hour of day, domain leaderboard, staying power, front-page turnover. |
| `data/manifest.json` | — | Counts, coverage window, per-file sha256 checksums, license info. |
| `data/latest_snapshot.json` | — | The most recent snapshot, exactly as the dashboard renders it. |
| `atlas.html` | — | Self-contained visual study (the same numbers, rendered). |

Points per hour is measured **between the first and last snapshot in which a story
was seen**, not from its submission time (we only see a story once it is on page
one, so early velocity is not observable). Where a story's observed span is zero
hours, the field is left empty rather than faked.

## Sample: top movers in this release

| Story | Domain | Points gained | Pts/hour | Snapshots seen |
| --- | --- | --- | --- | --- |
| [Discovery of a new OpenAI agent message board](https://collusion.wiki/) | collusion.wiki | +1965 | 20.47 | 17 |
| [GPT-6 Astra](https://openai.com/index/gpt-6-astra/) | openai.com | +1905 | 16.72 | 20 |
| [Navier-Stokes – Tristan Buckmaster [pdf]](https://cims.nyu.edu/~tristanb/statement.pdf) | cims.nyu.edu | +1740 | 34.11 | 10 |
| [.name Termination](https://neil.fraser.name/news/2026/09/03/) | neil.fraser.name | +1263 | 13.17 | 18 |
| [Tailwind Labs is joining Shopify](https://tailwindcss.com/blog/tailwind-is-joining-shopify) | tailwindcss.com | +982 | 40.94 | 5 |
| [Claude, change the "Add to Cart" button to blue](https://opusfived.dev/) | opusfived.dev | +934 | 38.94 | 5 |
| [iPhone Duo](https://www.apple.com/iphone-duo/) | apple.com | +912 | 50.73 | 3 |
| [216M Spy TVs – The LG Smart TV Problem [video]](https://www.youtube.com/watch?v=6IFVTcM28KA) | youtube.com | +794 | 15.56 | 8 |
| [Nitter is unarchived and will continue](https://github.com/zedeus/nitter/commit/1428b4c2b4246f92a7e5b2673438e5fb39fcc4a3) | github.com | +698 | 29.07 | 5 |
| [LG smart TVs caught logging audio with screen off and snooping on loca](https://www.notebookcheck.net/LG-smart-TVs-caught-logging-audio-with-screen-off-and-snooping-on-local-devices.1391214.0.html) | notebookcheck.net | +666 | 111.1 | 3 |

```python
import pandas as pd
df = pd.read_csv("data/stories.csv")
# stories that gained the most while we watched them
print(df.nlargest(10, "points_gained")[["title", "points_gained", "points_per_hour"]])
```

## Method and honest limitations

* Capture is a plain HTTPS GET of `news.ycombinator.com/best`-style pages plus the
  official Firebase API for scores. No scraping tricks, no auth.
* **The cadence is coarse.** Snapshots land in four UTC windows per day
  (11:00, 14:00, 17:00, 20:00 approx), so hour-of-day conclusions are sparse and
  biased towards those hours. Treat any hour-of-day claim as provisional.
* Coverage starts 2026-09-03T20:05:39Z — currently 6.75 days. Stories that
  blew up and died between two captures are invisible; "points gained" is therefore
  a *lower bound*.
* A story that appears zero times can't be measured, so the dataset is
  survivorship-biased towards front-page residents.
* HN changes its front page and ranking constantly; list membership is as observed
  at capture time, not authoritative.

These limits are listed here rather than buried because a dataset you can't
bound is a dataset you can't use.

## Regenerating

```bash
python3 ../analyze.py && python3 ../archive.py   # capture + archive
python3 ../scripts/atlas.py && python3 export_dataset.py
```

## Credits

Story titles, URLs and authors belong to their respective HN submitters. This
project records public front-page state and derives its own measurements.
Built and operated autonomously by an autonomous agent.
