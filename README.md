# ATS Public Job APIs — a cheatsheet

Pull job listings **straight from a company's ATS** (Applicant Tracking System) instead of scraping LinkedIn or Indeed. Most ATS platforms expose the company's job board as a **public JSON/XML endpoint** — no API key, no login, no browser, no anti-bot. It's the company's own source of truth, so it's cleaner and fresher than any aggregator.

`{company}` below is the company's slug (e.g. `stripe`), taken from its careers-board URL.

## The endpoints

| ATS | Endpoint | Method | Notes |
|-----|----------|--------|-------|
| **Greenhouse** | `https://boards-api.greenhouse.io/v1/boards/{company}/jobs?content=true` | GET | JSON, full HTML descriptions |
| **Lever** | `https://api.lever.co/v0/postings/{company}?mode=json` | GET | JSON, full descriptions |
| **Ashby** | `https://api.ashbyhq.com/posting-api/job-board/{company}?includeCompensation=true` | GET | JSON, sometimes salary |
| **SmartRecruiters** | `https://api.smartrecruiters.com/v1/companies/{company}/postings` | GET | JSON, paginated (`limit`/`offset`) |
| **Recruitee** | `https://{company}.recruitee.com/api/offers/` | GET | JSON, descriptions inline |
| **Breezy HR** | `https://{company}.breezy.hr/json` | GET | JSON, list has no descriptions |
| **BambooHR** | `https://{company}.bamboohr.com/careers/list` | GET | JSON |
| **Personio** | `https://{company}.jobs.personio.com/xml` | GET | XML feed |
| **Workday** | `https://{tenant}.{dc}.myworkdayjobs.com/wday/cxs/{tenant}/{site}/jobs` | POST | JSON, needs the full board URL (tenant + datacenter + site) |

A `404` on the JSON endpoints just means "this company isn't on that ATS" — try the next one.

## Quick start (Python)

Stripe uses Greenhouse:

```python
import requests

company = "stripe"
url = f"https://boards-api.greenhouse.io/v1/boards/{company}/jobs?content=true"
jobs = requests.get(url).json()["jobs"]

for j in jobs[:5]:
    print(j["title"], "—", j["location"]["name"])
```

Or with `curl`:

```bash
curl "https://api.lever.co/v0/postings/netflix?mode=json"
```

No Selenium, no proxy, no CAPTCHA solver. Runs in ~200ms and won't break next week because Cloudflare changed something.

## Auto-detecting the ATS

If you don't know which ATS a company uses, try the GET endpoints in order and take the first that returns jobs:

**Greenhouse → Lever → Ashby → SmartRecruiters → Recruitee → Breezy** covers a huge chunk of tech companies. Workday is worth special-casing (POST + full board URL).

## Gotchas

- **Rate limits** are lenient but real — be polite, set a `User-Agent`.
- **Descriptions**: Greenhouse/Lever/Recruitee include full HTML in the list; Breezy/BambooHR need a per-posting fetch.
- **Personio** returns XML, not JSON.
- **Workday** can't be guessed from a bare company name — you need the tenant, datacenter, and site from the full `myworkdayjobs.com` URL.

## A ready-made tool

If you'd rather not build and maintain a fetcher per ATS, there's a hosted actor that does all 9 (Greenhouse, Lever, Ashby, SmartRecruiters, Workday, BambooHR, Personio, Recruitee, Breezy), auto-detects the ATS from a bare company name or board URL, and returns normalized jobs:

**https://apify.com/ponderable_hydrometer/multi-ats-jobs**

Longer write-up: [Skip LinkedIn/Indeed — most companies' job boards have a public JSON API](https://dev.to/ronin13/skip-linkedinindeed-most-companies-job-boards-have-a-public-json-api-48b)

## Contributing

Spotted an ATS that's missing (Workable, Teamtailor, JazzHR, Comeet...) or an endpoint that changed? PRs welcome.
