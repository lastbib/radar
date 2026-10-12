# Daily data update (runs 07:00 Bangkok time)

Instructions for the scheduled task that keeps `data/races.json` and `data/news.json` current.
The site (`index.html`) reads both files; GitHub Pages republishes on every push to `main`.

## Scope
- Road races only: marathon (42K), half marathon (21K), 10K. Races that also offer 5K or other distances are fine.
- Countries: Southeast Asia and India first (Thailand incl. islands like Koh Samui, Phuket; Vietnam, Cambodia, Laos, Malaysia, Singapore, Indonesia, Philippines, Myanmar; India). Then wider Asia (Sri Lanka, Nepal, Hong Kong, Taiwan, Japan, Korea) as time allows.
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
