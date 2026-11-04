# iStrain: strain management dashboard for mice, fish and fruit flies

Streamlit app (toy data). Run locally:

```
mamba create -n istrain python=3.11 -y
mamba activate istrain
pip install -r requirements.txt
streamlit run app.py
```

## Strain IDs

| Platform  | Prefix | Example |
|-----------|--------|---------|
| Mice      | `S`    | `S101`  |
| Fish      | `L`    | `L404`  |
| Fruit fly | `D`    | `D210`  |

The prefix identifies the species, so an ID is unique across platforms. The app shows a sidebar warning (it does not block) when a strain ID does not match its platform's pattern.

## Data files

Each platform reads its files from its own folder: `data/` (mice), `data/aquatic/` and `data/fly/`. You can also set `$STRAIN_DATA_DIR`, or upload files from the sidebar.

- `breeding_pairs.csv` (required): `pair_id, strain, male_id, female_id, setup_date, status, sacrifice_date`
- `litters.csv` (required): `litter_id, pair_id, strain, dob, litter_size, wean_date, n_males_weaned, n_females_weaned`
- `strains.csv` (optional): `strain, name, genotype, background, recommended_control, source, date_acquired, notes`. Only `strain` is required. It feeds the Strain Information (mice) and Line Information (fish, flies) tab.

The column names are the same on every platform. Fish and fly files reuse the rodent names: a tank or cross is a "pair", and a clutch or brood is a "litter". The app relabels them on screen. `recommended_control` should be another listed strain ID; the app warns otherwise.

## Real data (aquatic)

`real_data/aquatic/` is generated from the Nursery management workbook (sheet `ALL DATA`) with
`python scripts/build_real_data.py` (needs `pandas` + `openpyxl`). Use it with
`STRAIN_DATA_DIR=real_data streamlit run app.py` (the aquatic tab reads `real_data/aquatic/`).
One tank = one clutch (`litters.csv`); `pair_id` = `<line>-<stockI>x<stockII>`; `litter_size` = 5/6 dpf count;
`n_males/females_weaned` = 90 dpf M/F sex count. `tanks_extra.csv` keeps the other columns (35/90 dpf counts, rack, diet, owner...); `exclude.csv` lists tanks left out of the platform files. See [notes/aquatic_decisions.md](notes/aquatic_decisions.md).

## TODO

- [ ] Review API with PyRat (see [notes/PyRat integration](notes/PyRat-integration.md))
- [ ] Decide data source (CSV vs PyRat API) after the API review
