# IIMJobs Scraper (Python): MBA, finance and management jobs to JSON and CSV

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00A67E?logo=apify&logoColor=white)](https://apify.com/data_daemon/iimjobs-jobs-scraper)
![Output](https://img.shields.io/badge/output-JSON%20%7C%20CSV%20%7C%20Excel-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

Scrape **IIMJobs.com jobs** into JSON or CSV from Python: title, company with its AmbitionBox profile, city, experience range, **salary in LPA** when disclosed, mandatory skills, applicant and view counts and the **full job description**. Search by keyword, city and experience, or export the whole board (about 170 new jobs a day).

[IIMJobs](https://www.iimjobs.com) is India's job board for MBA, finance, consulting and management roles. This repository is a small, tested Python client for the hosted
**[IIMJobs Jobs Scraper – MBA, Finance & Management Jobs India](https://apify.com/data_daemon/iimjobs-jobs-scraper)** on Apify: no browser, proxy, login or scraper maintenance on your side. Price:
**$2 per 1,000 jobs, full description included, plus $0.001 per run**; Apify's free plan ($5 of credit every month) covers about 2,500 jobs.

## Quick start

```bash
pip install apify-client
export APIFY_TOKEN=your_token      # Apify Console > Settings > Integrations
python scrape.py --keyword "product manager" --city "Bangalore" --max 50
```

`scrape.py` starts the Actor, waits for it and writes `iimjobs-jobs.json` (every field) and `iimjobs-jobs.csv`
(the main columns). For any other option, put the Actor input in a file and pass `--input`:

```bash
python scrape.py --input examples/input.json
```

```json
{
  "queries": [
    "product manager"
  ],
  "postedWithinDays": 7,
  "maxJobs": 5
}
```

## Sample output

A real run of [`examples/input.json`](examples/input.json) on 5 November 2026
([full JSON](examples/sample-output.json), [CSV](examples/sample-output.csv); descriptions shortened):

| postedAt | title | company | locations | experienceMinYears | experienceMaxYears |
|---|---|---|---|---|---|
| 2026-11-05 | Amgen - Manager - Global Product Supply Planning | Amgen | Hyderabad | 8 | 13 |
| 2026-11-02 | Keka - Manager - Product Marketing | Keka Technologies | Hyderabad, Bangalore | 8 | 11 |
| 2026-10-31 | Product Manager - Medical Education Technology | Karyarth Consultants | Hyderabad | 8 | 12 |
| 2026-10-31 | Piramal - Portfolio Manager - Personal Loan Product | Piramal Pharma Limited | Mumbai | 5 | 10 |
| 2026-11-04 | RBL Bank - Product Manager - Commercial Cards | RBL Bank | Gurgaon/Gurugram | 5 | 10 |

## Output fields

| Field | Description |
|---|---|
| `id`, `dedupKey`, `url` | IIMJobs job id, a stable `iimjobs:<id>` key for your own de-duplication, and the public job URL. |
| `title`, `designation`, `skills`, `mandatorySkills` | As tagged by the recruiter. |
| `experienceMinYears`, `experienceMaxYears` | Experience band in years. |
| `salaryHidden`, `salaryMinLpa`, `salaryMaxLpa` | Salary in LPA when disclosed. Most IIMJobs recruiters hide pay, so filter on `salaryHidden: false` for benchmarking. |
| `locations`, `remote`, `otherLocation` | Listed cities and a remote flag. |
| `applicants`, `views` | Applicant and view counts from the listing. |
| `postedAt` | ISO timestamp of the posting. |
| `description` | Full JD as HTML and plain text (`includeDescription: true`); `null` when the site has none. |
| `query`, `scrapedAt` | The keyword that found the job and the collection time. |

## Daily feed of only new jobs

```bash
python scrape.py --keyword "product manager" --only-new iimjobs-daily --max 0
```

With `--only-new <name>` the Actor remembers which jobs it has delivered under that name and returns only new ones;
skipped jobs are not charged. Run it from cron, a GitHub Actions schedule or an Apify schedule:

```yaml
# .github/workflows/iimjobs-daily.yml
on:
  schedule: [{cron: "30 2 * * *"}]
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install apify-client && python scrape.py --keyword "product manager" --only-new iimjobs-daily --max 0
        env: {APIFY_TOKEN: "${{ secrets.APIFY_TOKEN }}"}
      - uses: actions/upload-artifact@v4
        with: {name: iimjobs-jobs, path: "iimjobs-jobs.*"}
```

## Other ways to call it

**curl**, one request that runs the Actor and returns the rows:

```bash
curl -X POST "https://api.apify.com/v2/acts/data_daemon~iimjobs-jobs-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"queries": ["product manager"], "postedWithinDays": 7, "maxJobs": 5}'
```

**Apify CLI:** `apify call data_daemon/iimjobs-jobs-scraper --input-file examples/input.json`

**No code:** open the [Actor page](https://apify.com/data_daemon/iimjobs-jobs-scraper), fill the form and export to CSV/Excel, or connect it to Google Sheets,
Zapier, Make or n8n. AI agents can call it through the Apify MCP server (`https://mcp.apify.com`).

## FAQ

### Is there an official IIMJobs API?

No public one. IIMJobs (part of Info Edge since 2019) has no documented job-search API for third parties. This repo calls a hosted Apify Actor that reads IIMJobs' public job search, so you get JSON without maintaining a scraper.

### Can I get every IIMJobs job, not only one keyword?

Yes: `"allJobs": true` with `postedWithinDays` exports the whole board for that window (about 170 new jobs a day).

### Does it also cover Hirist.tech?

Hirist is IIMJobs' sister board for tech jobs and has its own Actor with the same output schema: [Hirist Jobs Scraper](https://apify.com/data_daemon/hirist-jobs-scraper).

### Is it legal to scrape IIMJobs?

The Actor reads only public job listings, without logging in, at a polite request rate, and returns job and company
data, not personal profiles. Check the rules that apply to your use of the data in your jurisdiction.

## Related

- [Hirist.tech Jobs Scraper – India IT & Tech Jobs, Full JD](https://apify.com/data_daemon/hirist-jobs-scraper)
- [Naukri Jobs Scraper – India Jobs API with Salary (LPA)](https://apify.com/data_daemon/naukri-jobs-scraper)
- [Foundit Jobs Scraper | foundit.in (Monster India) Jobs API](https://apify.com/data_daemon/foundit-jobs-scraper)
- [India Jobs Feed | Naukri + 9 Indian Job Boards in One API](https://apify.com/data_daemon/india-jobs-feed)
- [India Job Alerts](https://github.com/datadmon/india-job-alerts): new jobs from seven Indian boards every morning via GitHub Actions
- [All Daemon Data Actors](https://apify.com/data_daemon)

Unofficial project, not affiliated with IIMJobs. Issues and questions: open an issue here or on the
[Actor's Issues tab](https://apify.com/data_daemon/iimjobs-jobs-scraper/issues).
