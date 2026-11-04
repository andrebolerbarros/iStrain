# Aquatic data: decisions

How `real_data/aquatic/` is built from the Nursery management workbook (`scripts/build_real_data.py`, sheet `ALL DATA` of the 20240723 file; it is a superset of the 20240110 file). The workbook is a real-world example only; a different dataset will replace it later. The toy data is not used at this stage.

**Source and mapping**
* One tank row is one clutch (`litters.csv`).
* `strain` is the line number parsed from the tank label (`Line nr.` is almost empty). IDs such as `LM12` and `L358A` are valid.
* `pair_id` is `<line>-<stockI>x<stockII>`, the two parental stock numbers. `male_id` and `female_id` are left blank: parental sex is not used for fish.
* `litter_size` is the 5/6 dpf count. The 90 dpf male/female counts are the sex proportion at the "weaning" stage (`n_males_weaned`, `n_females_weaned`, with the counting date as `wean_date`).
* `tanks_extra.csv` keeps the other columns (35 and 90 dpf counts, rack, position, intervention, owner, notes). The app does not use it.

**Background and controls**
* Wild-type lines (L170, L171, L173, L387, L450–L452) point to themselves as `recommended_control`.
* A background is treated as one exact value, so AB-TL-TU and TU-TL-AB differ. A line gets the wild type with the same background string.
* Lines with no matching wild type point to themselves.
* For TL, L387 (TL-Utrecht) is the control: it has the most data of the TL wild types.

**Missing & problematic info**
* The 15 tanks with no line number are out of the platform CSVs and in `exclude.csv`.
* The 18 tanks where the 90 dpf male plus female count exceeds the 5/6 dpf count are in `exclude.csv` too.
* `exclude.csv` has the reason, the original tank label and the counts.

**Status**
* Sheet `Status` of `finished` or `Euthanized` is non-active (`inactive`).
* Blank, `not finished`, `to transfer` and `to be crossed` are active.
* A pair is `inactive` only if all its tanks are.

**App settings**
* Aquatic strain ID pattern: `L[A-Z]?\d+[A-Z]?`.
* Benchmark clutch size: default 35 embryos (maximum 100), against a median of 35 in the data. The toy fish benchmarks were removed.

**Repository**
* `notebooks/` and `results/` are gitignored. `real_data/` and `scripts/` stay untracked.
