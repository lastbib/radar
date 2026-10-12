# Daily data update (runs 07:00 Bangkok time)

Instructions for the scheduled task that keeps `data/races.json` and `data/news.json` current.
The site (`index.html`) reads both files; GitHub Pages republishes on every push to `main`.

## Scope
- Road races only: marathon (42K), half marathon (21K), 10K. Races that also offer 5K or other distances are fine.
- Countries: Southeast Asia and India first (Thailand incl. islands like Koh Samui, Phuket; Vietnam, Cambodia, Laos, Malaysia, Singapore, Indonesia, Philippines, Myanmar; India). Also cover East Asia every run: Japan, South Korea, China, Taiwan and Hong Kong (big city races plus regional ones such as Okinawa, Gyeongju, Xiamen). Then Sri Lanka and Nepal as time allows. Use country names exactly as in the ISO map in index.html so flags show.
- Race dates from today up to ~12 months ahead.

## Each run
1. Read both JSON files.
2. For every race with status `open`, `almost_full`, `soon` or `unknown`, verify on the organiser site or a registration platform (thai.run, runlah, regis.run, worldsmarathons, ahotu, raceroster, procam, checkpointspot, etc.):
   - registration status, opening/closing dates, current prices per distance, sold-out or "almost full" notices, date changes.
3. Look for new races in scope (aim for 3–5 new candidates per run, local races included). Add only races with a confirmed date.
4. Update fields, set `checked` to today, set top-level `updated` to now (ISO, +07:00).
5. Append to `data/news.json` `items` (newest any order; keep the last 60 days):
   - `opened` – registration just opened
   - `closing` – closes within 14 days, or an early-bird/price step ends within 7 days
   - `almost_full` – organiser says nearly sold out
   - `sold_out` – sold out / closed early
   - `new` – race newly added
   - `price` – price change
   - `date_change` – race date moved
   Each item: `{"date":"YYYY-MM-DD","type":"...","raceId":"...","text":"one plain sentence"}`. Don't repeat an alert already listed for the same race and type unless something changed.
6. Validate JSON (`python3 -m json.tool`), commit with a short message, push to `main`.

## Race record
```json
{
  "id": "kebab-case-name-year",
  "name": "Official race name",
  "city": "City", "country": "Country (use names from REGIONS in index.html)",
  "lat": 0.0, "lon": 0.0,
  "date": "YYYY-MM-DD", "dateTentative": false,
  "distances": [{"d": "42K|21K|10K|5K", "price": "₫950,000"}],
  "priceNote": "optional",
  "status": "open | almost_full | soon | full | closed | unknown",
  "regOpens": "YYYY-MM-DD or empty", "regOpensNote": "optional, for soon",
  "regCloses": "YYYY-MM-DD or empty",
  "relevance": 0-100,
  "facts": "Exactly two sentences with the most interesting facts, in your own words.",
  "url": "official or registration page",
  "source": "page used to verify",
  "checked": "YYYY-MM-DD"
}
```
- `soon` = registration not open yet but the race is confirmed (shown as "Opening soon").
- `unknown` = could not verify; hidden on the site until verified.
- The site hides `full`, `closed`, `unknown`, past races, and anything whose `regCloses` has passed.
- Relevance: size and international field (40), certification such as World Athletics label / AIMS / Boston qualifier (20), destination and scenery (25), organisation reputation (15).
- Prices marked "≈" are from a previous edition. Replace them once current prices are published.
- Leave closed/full races in the file until race day so alerts stay accurate, then delete them.

## Covers and past winners
- Each race has `wiki`: a list of English Wikipedia article titles for its landmark or city (first one with a lead photo is used as the cover, credited to Wikimedia Commons). Add 1–3 titles for every new race.
- Each race has `regUrl`: the direct registration page (falls back to `url`).
- `data/results.json` → `results[<race id without the -YYYY suffix>]` holds the latest edition's winners:
  `{"edition":"2025","date":"YYYY-MM-DD","source":"url","fullResults":"url","distances":{"42K":{"men":[{"p":1,"n":"Name","c":"Country","t":"2:10:00"}],"women":[...]},"21K":{...},"10K":{...}}}`
- Each run, fill results for 2–3 races that have none, and extend existing ones toward the top 10 per gender per distance when official results pages are readable. Only enter places and times you actually read in a source. After a race takes place, replace its entry with the new edition.

## Prices, currency and German text
- Every distance with a known fee also gets numbers for the currency converter: `"amt": 950000, "cur": "VND"` (ISO code), plus `"qual": "from" | "upto" | "approx"` when the price is a range start, a maximum, or from a previous edition. Keep `price` as the human-readable original. Use the price currently on sale.
- Every race needs `facts_de`: the same two sentences in natural German.
- Every news item needs `text_de`: the same alert in German.

## Majors and certified races
- Always track the Abbott World Marathon Majors (Tokyo, Boston, London, Sydney, Berlin, Chicago, New York) and World Athletics Label races in Asia and the Middle East (e.g. Dubai, Abu Dhabi, RAK Half, Xiamen, Seoul, Daegu, Taipei, Mumbai, Delhi, Singapore, Hong Kong). Track their ballot/lottery windows closely; these are the most important alerts.
- `cert` lists certifications: `major`, `wa-platinum`, `wa-gold`, `wa-label`, `wa-elite`, `aims`, `bq` (Boston qualifier). Only add a tag you confirmed in a source.
- Country names for new regions: "United Arab Emirates", "Germany", "United Kingdom", "United States", "Australia" (add others to ISO/REGIONS in index.html if needed).
