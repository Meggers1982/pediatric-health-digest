# Children's & Pediatric Health Research Digest

A GitHub Actions workflow that searches curated pediatric medicine, child development, child behavioral science, and pediatric nutrition journals on PubMed, filters out widely covered stories, runs a single Claude pass for journalist-ready summaries and pitch angles, and publishes results to a GitHub Pages dashboard.

## How it works

1. **PubMed search** - Queries journals by ISSN for studies published in the past 7 days
2. **Title screening** - Prioritizes studies with novelty signals and excludes animal-only studies
3. **SERPAPI media filter** - Checks Google News and skips any study with 3+ news results
4. **Abstract fetch** - Retrieves full abstracts for shortlisted studies
5. **Claude pass** - Writes structured JSON: headline, summary, why it matters, caveats, relevance score, and pitch angles per publication type
6. **Artifact upload** - Saves JSON results as a GitHub Actions artifact
7. **Deploy job** - Downloads all job artifacts, merges and deduplicates by PMID, commits `data/results.json`, serves via GitHub Pages
8. **Email notification** - Sends a short email with study count and a dashboard link

## Dashboard

Features:
- Card view per study with headline, summary, caveats, fact-check notes
- Expandable pitch angles section for publications such as Parents Magazine, Today's Parent, NPR Health, The New York Times (Parenting), TIME Health, Good Housekeeping, Health.com, Women's Health Magazine, Verywell Health, Everyday Health, and general health outlets
- Filter by category, groundbreaking type, status, date range, and score
- Search across all study text and pitches
- Status tracking (New / Saved / Pitched / Passed) saved to localStorage
- Deduplication across runs by PMID

## Schedule

Runs automatically every morning at 7:00 AM ET. All jobs run in parallel; the deploy job merges results and publishes the dashboard once complete.

Can also be triggered manually via **Actions -> Children's & Pediatric Health Research Digest -> Run workflow**.

## Categories

| Category | Journals | Jobs |
|---|---:|---|
| Pediatrics | 86 | 2 (chunks 1-2) |
| Behavioral Sciences | 89 | 2 (chunks 1-2) |
| Nutritional Sciences | 62 | 2 (chunks 1-2) |

Large categories are split into chunks to keep run times under 20 minutes.

The journal CSVs in `data/` are now hand-maintained. `scripts/extract_journals.py` originally generated them from a source workbook that no longer exists, so re-running it would wipe hand-added rows. Edit the CSVs directly.

## Journal list audit (2026-09-14)

Method: pulled OpenAlex's top sources for this digest's subject areas over the prior year, diffed them against the CSVs, and kept only titles PubMed actually indexes (listed in NCBI's journal catalog with PubMed articles in the last 12 months). Because every row's full weekly output enters the digest with no topic filter, only journals whose whole output is pediatric were considered.

Added to `Pediatrics.csv` (neonatology and child development were the thinnest areas):

| Journal | ISSN (Online) | PubMed articles/yr |
|---|---|---:|
| Journal of Perinatology | 1476-5543 | 591 |
| Early Human Development | 1872-6232 | 202 |
| Developmental Science | 1467-7687 | 196 |
| Advances in Neonatal Care | 1536-0911 | 120 |
| Journal of Child Language | 1469-7602 | 107 |

Notable exclusions:
- **Not usable in PubMed** - Journal of Neonatal Nursing, Cognitive Development, First Language, Reading and Writing, Reading Psychology, Journal of Motor Learning and Development, and Behavioral Interventions have zero or near-zero PubMed articles in the past year, so the pipeline could never find them.
- **Education research, not child health** - Language, Speech, and Hearing Services in Schools; Annals of Dyslexia; Journal of Deaf Studies and Deaf Education; Behavior Analysis in Practice.
- **Off-beat or mostly adult** - Journal of Adolescent and Young Adult Oncology (ages 15-39, mostly adult oncology), Clinical Linguistics & Phonetics, Psychology of Sport and Exercise, Contraception and Reproductive Medicine, cereal and food-chemistry titles.
- **Non-English or not indexed** - a batch of Indonesian and Russian education/psychology journals that OpenAlex lumps into these subject areas.

No mega-journals surfaced in this pass. Pediatrics grew from 81 to 86 journals (about 6%), so the workflow chunking is unchanged.

## Manual Trigger

Go to **Actions -> Children's & Pediatric Health Research Digest -> Run workflow**.

- Leave **category** blank to run all jobs
- Enter an exact category name, such as `Pediatrics`, to run just that category

## GitHub Pages Setup

1. Go to **Settings -> Pages**
2. Set source to **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. Save; GitHub will serve `index.html` at the dashboard URL

## Required Secrets

Add these in **Settings -> Secrets and variables -> Actions**:

| Secret | Description |
|---|---|
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `SERPAPI_KEY` | SerpAPI key for Google News filtering |
| `SUPABASE_URL` | Supabase project URL (dashboard save/delete personalization) |
| `SUPABASE_KEY` | Supabase API key (read-only) |
| `DASHBOARD_REPO_TOKEN` | Token with push access to the shared `research-digest-dashboard` repo |

## Repo Structure

```text
.github/
  workflows/
    pediatric-health-digest.yml
scripts/
  pediatric_health_digest.py
  merge_results.py
  extract_journals.py
data/
  Pediatrics.csv
  Behavioral Sciences.csv
  Nutritional Sciences.csv
  results.json
index.html
requirements.txt
```

## Dashboard Study Card Fields

Each study card shows:

- **Headline** - plain-language present-tense summary
- **Relevance score** - 1-10, weighted for child health, parenting, child development, and family medicine journalism fit
- **Category & journal** - source metadata
- **Groundbreaking type** - counterintuitive, overturns prior research, first-in-class, or domain-relevant finding
- **Media coverage** - SERPAPI verification status
- **The study** - what was done, who participated, and the key finding
- **Why it matters** - real-world significance for the target audience
- **Caveats** - limitations flagged automatically
- **Fact-check note** - corrections made during the Claude pass
- **Pitch angles** - expandable publication-specific pitch blocks
- **Status** - New / Saved / Pitched / Passed, tracked in your browser
