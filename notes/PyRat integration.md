# PyRat integration

Review of how iStrain could read data from PyRat (Scionics) instead of hand-prepared CSVs.

## 0. Evidence level (read this first)

- **I could not find or open any PyRat API documentation.** Web searches returned no endpoint list, no Swagger/OpenAPI spec, no auth description. Direct fetches of the vendor site (`scionics.com`) and of the RSpace docs failed (host not reachable from the sandbox this was written in).
- **What I do have is search-engine summaries of three pages, not the pages themselves:**
  - Scionics product pages say PyRAT has an API for integrating with other systems ([PyRAT](https://www.scionics.com/pyrat.html), [PyRAT Aquatic](https://www.scionics.com/pyrat_aquatic)).
  - RSpace's integration doc says users need a personal access token from their PyRAT admin, and that Scionics must grant permission and issue a client ID ([RSpace: PyRAT integration](https://documentation.researchspace.com/article/9kkeooveia-py-rat-integration)).
  - A search summary suggested the API may sit under a paid add-on for PyRAT Aquatic. **That is an inference; unconfirmed.**
- **Everything below about endpoints, fields, limits and licensing is therefore a hypothesis to check**, not a finding. Sections 1 and 2 are firm only where they describe *this repo*.

## 1. What iStrain needs (the data contract)

The app reads three tables per platform. Any PyRat integration has to end up producing exactly these dataframes.

| Table | Columns | Notes |
|---|---|---|
| `breeding_pairs` | `pair_id, strain, male_id, female_id, setup_date, status, sacrifice_date` | Active/sacrificed state is derived from the two dates and the as-of date. |
| `litters` | `litter_id, pair_id, strain, dob, litter_size, wean_date, n_males_weaned, n_females_weaned` | Blank weaning counts mean "not weaned yet"; this drives the "to wean" KPI and the survival and sex-ratio models. |
| `strains` (optional) | `strain, name, genotype, background, recommended_control, source, date_acquired, notes` | Descriptive only. |

Platform differences: mice use `S[number]`, fish `L[number]`, flies `D[number]`. Fish and fly data reuse the rodent column names and are relabelled on screen.

## 2. How to integrate

**Option A (recommended first step): sync script that writes the same CSVs.**
- A separate `pyrat_sync.py` calls the API, normalises, and writes `breeding_pairs.csv`, `litters.csv` (and what it can of `strains.csv`) into `data/`, `data/aquatic/`.
- `app.py` stays untouched. CSVs remain the interface, so the sidebar upload and the toy data keep working as a fallback.
- Run it manually or on a schedule (cron). The snapshot is also an audit trail and makes numbers reproducible.
- Cost: data is as stale as the last sync.

**Option B: live adapter inside the app.**
- A `load_from_pyrat()` returning the same dataframes, wrapped in `st.cache_data(ttl=...)`.
- Fresher, but every page load depends on PyRat being up, reachable and fast. Worse failure mode, and harder to debug.

**Option C: hybrid.** Start with A. Move to B only if a snapshot proves too stale and the API proves fast and stable.

**Why not a database:** already decided to keep CSVs for now. A sync script that writes CSVs avoids adding a store.

## 3. Downstream changes required

- **New module and config:** `pyrat_sync.py` (or `pyrat/` package) plus a mapping config (PyRat field → iStrain column).
- **Secrets:** token and instance URL via environment variables or Streamlit secrets. Never in the repo. Add to `.gitignore` guidance in the README.
- **Strain ID mapping:** PyRat strain names must be mapped to `S`/`L`/`D` IDs. Decide the source of truth for IDs (PyRat field, or a mapping table in this repo). The sidebar prefix check will flag mismatches.
- **Terminology mapping:** a PyRat "breeding" or "cage" must map to a `pair_id`. A litter must map to a `litter_id`. Trios and harems do not fit `male_id` / `female_id` as single columns.
- **Weaning data:** `n_males_weaned` and `n_females_weaned` may not exist as such. They might have to be derived from animals created from a litter and sexed at weaning. Needs a defined rule.
- **Historical state:** the as-of date feature needs `setup_date` and `sacrifice_date` per pair. If the API only gives current state, past as-of dates may be wrong.
- **`strains.csv` stays partly manual:** `recommended_control`, `genotype`, `background` and the benchmarks are probably not in PyRat in a usable form. Expect a merge of API data and a hand-kept file.
- **README and TODO:** document the sync, the token setup and the field mapping; update the TODO item.
- **Tests:** keep the toy CSVs as fixtures. Add a check that the sync output passes the same column validation as uploaded files.

## 4. Bottlenecks and caveats

- **Access and licensing (unconfirmed):** per the RSpace doc, Scionics may need to approve a client ID and a PyRAT admin must issue tokens. The API may be a licensed add-on. This is the likely first blocker, and it is organisational, not technical.
- **Personal tokens:** a token tied to one person means the sync breaks when they leave. Ask for a service account.
- **Fruit flies:** PyRat is a rodent system, with a separate Aquatic module for fish. **I found nothing indicating PyRat supports Drosophila.** The fly pane may have no PyRat source at all and stay CSV-only. Check this before promising a three-platform integration.
- **API granularity unknown:** if the API exposes animals and cages, not litters or breedings, litter reconstruction is real analytical work with room for silent errors.
- **Missing-data semantics:** the app treats blank weaning data as "not weaned yet". PyRat may use different conventions, such as empty, zero or a status flag. A mismatch would silently skew the survival models and the "to wean" KPI.
- **Rate limits, pagination, API versioning:** all unknown. Pulling the full history (about 500 litters per platform here, probably more in real life) may need paging. Version changes can break the mapping.
- **Network reach:** the instance is likely on an institutional network or behind a VPN. A hosted Streamlit app may not be able to reach it, which is another point in favour of snapshots (Option A) generated inside the network.
- **Data governance:** this is animal-facility data. Check what may be copied out, where snapshots may be stored (a git repo is probably the wrong place), and who may see them.
- **Validation:** until the app's counts are reconciled against PyRat's own reports for a sample period, treat the integrated numbers as unverified.

## 5. Suggested next steps (in order)

1. Ask the PyRat admin / Scionics for: API documentation (Swagger URL), whether the API is included in the licence, a service-account token, and whether fish and fly (Drosophila) data are in scope.
2. With a token, pull one sample each of strains, pairs/breedings and litters. Check which iStrain columns can be filled directly, which need a rule, and which are absent.
3. Write the field mapping as a table (PyRat field → iStrain column, rule, confidence).
4. Only then build Option A for one platform (mice), and reconcile its output against PyRat's own report.
5. Decide, based on step 2, whether the data source question in the README TODO is settled.

**Scope warning:** steps 1 and 2 cost days, not weeks, and decide everything else. Do not start on the sync script, the mapping layer or any refactoring before they are done.
