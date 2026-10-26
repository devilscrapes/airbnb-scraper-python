<p align="center"><a href="https://apify.com/DevilScrapes/airbnb-scraper"><img src="assets/banner.svg" alt="Airbnb Listings &amp; Pricing Scraper — Python example" width="100%"></a></p>

# Airbnb Scraper in Python: short-term rental comps for pandas

<p align="center">
<a href="https://apify.com/DevilScrapes/airbnb-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/airbnb-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/airbnb-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repository is a small, tested Python runner for the **[Airbnb Listings & Pricing Scraper](https://apify.com/DevilScrapes/airbnb-scraper)** on Apify. Name a destination the way you would type it into Airbnb's search box, or paste a search URL that already carries your guest count, price slider and room-type filters, and it returns the homes that search shows as a flat table: listing id, title, room type, city, latitude and longitude, nightly rate, currency, stay total when you give dates, star rating, review count, superhost flag, bedroom, bed and bathroom counts, and photo links. The collection runs on Apify's servers through residential exits; your notebook only opens `results.json` or `results.csv`. That is the shape a short-term rental analyst wants before arguing about a purchase price.

> **Actor:** [Airbnb Listings & Pricing Scraper](https://apify.com/DevilScrapes/airbnb-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [airbnb-scraper-javascript](https://github.com/devilscrapes/airbnb-scraper-javascript)

## 🔥 What you get

- **Two ways to aim a search**: type a place such as `Lisbon, Portugal` and the Actor builds the `/s/<place>/homes` address for you, or paste a URL you filtered on airbnb.com and it is used exactly as given.
- **Stay totals that match real dates**: supply both `checkIn` and `checkOut` and `total_price` carries the amount Airbnb shows for that stay; leave the dates out and the column stays empty instead of holding a guess.
- **Room shape in numbers**: bedrooms, beds and bathrooms are parsed out of each card's summary line into integers and decimals, so half baths survive as `1.5`.
- **Every listing id once**: Airbnb's result pages overlap as you scroll; repeats are dropped before they reach the dataset or the bill. Up to 1,000 homes per run, 18 per results page.
- **Exit country you choose**: requests always leave through Apify's residential pool in the country given by `proxyCountry`, which matters because Airbnb localises prices and currency to the visitor.
- **Partial runs say so**: if Airbnb stops answering midway, the rows already collected are kept and the status message reads how many arrived out of how many were asked for. Throttled requests get a few spaced-out retries before that happens.

## 🎯 Use cases

### Comp a property before you buy it

Search the neighbourhood you are underwriting, filter rows to the same bedroom count as the target and take the median `price_per_night`. Ten minutes in a notebook gives a defensible rate assumption instead of a number someone heard at a meetup.

### Track seasonal pricing for one market

Schedule the script with cron for the same city and a rolling set of `checkIn` dates. Each dated CSV adds a snapshot, and a pivot on date against bedrooms shows how far rates climb into peak season and how early they start.

### Study short-term rental density for housing research

Coordinates per listing let researchers join homes to census tracts or wards in GeoPandas. Counting entire homes against private rooms per area is a common first table in policy papers on rental supply.

### Benchmark a managed portfolio

Property managers can run the same search their guests use, then see where their own homes sit on rating, review volume and nightly rate against nearby superhost listings with the same bedroom count.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/airbnb-scraper-python.git
cd airbnb-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Airbnb Listings & Pricing Scraper](https://apify.com/DevilScrapes/airbnb-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Without the command-line extras, the whole integration fits in the lines below. `apify-client` launches the Actor, waits for a final status and then yields rows from the run's dataset.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/airbnb-scraper").call(run_input={'location': 'Austin, TX', 'maxResults': 10, 'proxyConfiguration': {'useApifyProxy': True, 'apifyProxyGroups': ['RESIDENTIAL'], 'apifyProxyCountry': 'US'}})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` forwards `input.json` unchanged. The bundled file searches Austin, Texas and stops after ten homes, which keeps a trial run to cents. To price a specific trip, add `checkIn` and `checkOut` as `YYYY-MM-DD` strings and set `guests`. Note that `proxyCountry`, not the country inside `proxyConfiguration`, decides where requests exit.

```json
{
  "location": "Austin, TX",
  "maxResults": 10,
  "proxyConfiguration": {
    "useApifyProxy": true,
    "apifyProxyGroups": [
      "RESIDENTIAL"
    ],
    "apifyProxyCountry": "US"
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `location` | string | no | — | — |
| `searchUrl` | string | no | — | — |
| `checkIn` | string | no | — | — |
| `checkOut` | string | no | — | — |
| `guests` | integer | no | `1` | — |
| `maxResults` | integer | no | `50` | — |
| `enableDetailEnrichment` | boolean | no | `false` | — |
| `proxyCountry` | string | no | `"US"` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true, "apifyProxyGroups": ["RESIDENTIAL"], "apifyProxyCountry": "US"}` | — |

## 📤 Output

One record per home. Nullable columns arrive as `None` in pandas when Airbnb's search card does not carry the value, so the Actor never fills a gap with a guess. `price_per_night` is read from Airbnb's own price breakdown, and `currency` is mapped from the symbol shown ($, € or £), staying empty for other symbols. Host identity, guest capacity and country come back empty from the search card, and `amenities` stays an empty list unless enrichment is on and the listing page includes them. `image_urls` holds up to twenty photo links.

| Field | Type | Always present |
|---|---|---|
| `listing_id` | string | yes |
| `url` | string | yes |
| `title` | string or null | no |
| `listing_type` | string or null | no |
| `room_type` | string or null | no |
| `city` | string or null | no |
| `country` | string or null | no |
| `lat` | number or null | no |
| `lng` | number or null | no |
| `price_per_night` | number or null | no |
| `currency` | string or null | no |
| `total_price` | number or null | no |
| `rating` | number or null | no |
| `review_count` | integer or null | no |
| `host_id` | string or null | no |
| `host_name` | string or null | no |
| `is_superhost` | boolean or null | no |
| `bedrooms` | integer or null | no |
| `beds` | integer or null | no |
| `bathrooms` | number or null | no |
| `max_guests` | integer or null | no |
| `amenities` | array | no |
| `image_urls` | array | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result scraped | $0.0029 |

Prices as configured on 2026-09-09. The [Actor page](https://apify.com/DevilScrapes/airbnb-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

The pandas example below (`pip install pandas`) turns raw rows into a quick comp table: median nightly rate by bedroom count, split by superhost status, plus a `comp_set.csv` of well-reviewed homes sorted by rating.

```python
import json

import pandas as pd

comps = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
comps = comps.dropna(subset=["price_per_night"])
comps["bedrooms"] = comps["bedrooms"].fillna(0).astype(int).clip(upper=4)
comps["size"] = comps["bedrooms"].map(lambda n: "studio" if n == 0 else f"{n}{'+' if n == 4 else ''} bd")

by_size = comps.pivot_table(
    index="size", columns="is_superhost", values="price_per_night",
    aggfunc="median", observed=True,
).rename(columns={False: "regular_host", True: "superhost"})
print(by_size.round(0))

reviewed = comps[comps["review_count"].fillna(0) >= 10]
reviewed.sort_values("rating", ascending=False)[
    ["listing_id", "room_type", "bedrooms", "price_per_night", "currency", "rating", "review_count", "url"]
].to_csv("comp_set.csv", index=False)
```

## ❓ FAQ

### Does Airbnb offer a public API I could use instead?

Not for search data. Airbnb's API program is limited to approved software partners working for hosts. This Actor reads the same results page a traveller sees, and all you need is an Apify token.

### Why is `price_per_night` empty on some rows?

The nightly figure comes from Airbnb's own price explanation on each card. When a card does not include that breakdown, the field stays empty rather than being estimated from the stay total.

### Can I reproduce a search I built on the website?

Yes. Set every filter on airbnb.com, copy the address bar into `searchUrl` and run. That URL takes priority over `location`, so the dataset matches the page you were looking at.

### Why does it insist on residential proxies?

Airbnb answers datacenter traffic poorly, so the Actor always switches to Apify's residential group, even if your input names another. On a plan without residential access the run stops at the start instead of returning an empty table.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/airbnb-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Airbnb Listings & Pricing Scraper on Apify Store](https://apify.com/DevilScrapes/airbnb-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/airbnb-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Hostelworld Hostel Listings Scraper](https://apify.com/DevilScrapes/hostelworld-hostel-listings-scraper) — budget beds and hostel ratings for the same destinations.
- [Furnished Finder Corporate Housing Scraper](https://apify.com/DevilScrapes/furnished-finder-corporate-housing-scraper) — monthly furnished rentals, the mid-term side of the market.
- [Hipcamp Campsite Listings Scraper](https://apify.com/DevilScrapes/hipcamp-campsite-listings-scraper) — outdoor stays and glamping sites priced per night.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/airbnb-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Airbnb, Inc..
