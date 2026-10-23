[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003195-blue)](https://doi.org/10.82901/nemar.on003195)

The quest for neuroprotection in Parkinson’s disease (PD) has been for new compounds to slow disease progression and stable and non-invasive biomarkers to document their benefits. Neuroepo, a new formulation of EPO with low content of sialic acid reported good results in animal model and tolerance in healthy participants and PD patients. 
In a double-blind randomized placebo (https://clinicaltrials.gov/ number NCT04110678) twenty-five PD patients were assigned randomly to Neuroepo (n=15) or placebo (n=10) groups we reported the tolerance of the drug. We recorded resting-state EEG before and six months after the administration of the drug. The qualitative analysis of the abnormalities of the EEG was evaluated by two experts using a Likert-type scale and a multivariate item response theory (MIRT) approach was employed to stablish the differences between groups in the two times. The quantitative EEG (qEEG) analysis was performed at the sources looking for generators of the neural activity using software VARETA and co-registering the results using the Montreal Neurological Institute Atlas. The statistical analysis between the sources was conducted using a permutation test and later a contrast method using the surfstat software between groups and before vs after condition, with Bonferroni correction for multiple comparisons.
Here in this repository we placed the raw EEG in BIDS format  (Pernet, C. R. et al. EEG-BIDS, an extension to the brain imaging data structure for electroencephalography. Sci. data 6, 103 (2019). For the use of VARETA the qEEG program you can use (Bosch-Bayard, J. et al. A Quantitative EEG Toolbox for the MNI Neuroinformatics Ecosystem: Normative SPM of EEG Source Spectra. Front. Neuroinform. 14, (2020).)
The EEG dataset from the different stages of processing can be requested to the authors. 

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 1 error + 423 warnings to 0 errors + 342 warnings. None of the raw `.edf` files were modified — every change is to a text sidecar.

**Participant table (`participants.tsv`) and its new sidecar (`participants.json`)**
- Each row ended in an extra tab, which created an empty, unnamed third column that the validator flagged; the stray tab was removed and the file's line endings were normalized. No participant data was changed.
- The `sex` column records each participant as a number (`0`, `1`, or `6`), but there was no sidecar to explain those codes, so the validator read them against its built-in `F`/`M`/`O` list and rejected them. A new `participants.json` declares the column with exactly the codes that appear and notes that the dataset does not document what each number means — that key has to be looked up in the source publication. The lone `6` (one participant, sub-PLAC17) is flagged as unexplained in the source. No mapping was invented.

**Scan tables — new shared sidecar (`scans.json`) at the dataset root**
- The scan tables use a column named `age_acq_time` that BIDS does not define, so a root-level `scans.json` describes it. Most entries are a time of day (`HH:MM:SS`); one (sub-PLAC17, pre-placebo session) is a decimal (`69.39`) whose meaning the source does not give. The column name mixes two ideas ("age" and "acquisition time") and the source does not say which is intended, so no unit was asserted and the description records that ambiguity rather than guessing.

**Event tables — two new shared sidecars at the dataset root (`task-taskplacebopre_events.json`, `task-taskplacebo6m_events.json`)**
- The event tables had no sidecar describing their columns. One shared sidecar per task now documents the `value` and `sample` columns (standard MNE-BIDS export columns) and the `trial_type` column, and applies to every matching recording at once. The `trial_type` labels are kept as free text rather than a fixed list, because the recordings use a dozen varied (and sometimes misspelled) Spanish labels and pinning them to a fixed list would reject the unmapped ones; the observed labels are described in the column's description.

**Dataset description (`dataset_description.json`)**
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative.
- Updated `BIDSVersion` from `1.2.0` to `1.11.1` (the version the current validator checks against).
- `GeneratedBy` was left absent, exactly as the source published it — nothing was added there.

**Recording sidecars (`_eeg.json`, all 20 recordings)**
- The misc-channel-count field was spelled `MiscChannelCount`; BIDS uses the all-uppercase `MISCChannelCount`. It was renamed (the value, `0`, was already correct) so the validator recognizes it.

**Remaining warnings (342) — left on purpose**
- These are all "recommended but missing" study, lab, and equipment fields not present in the dataset (for example: manufacturer, cap model, device serial number, instructions, institution, hardware filters, ground electrode, head circumference, cognitive-atlas IDs). One of them is `GeneratedBy`: the source does not document what tool generated the data, so it was left absent rather than guessed. All were left blank rather than filled with invented values.

**The recording files have a header defect (not fixed here)**
- All 20 `.edf` files declare more data records in their header than the file actually contains (off by roughly 1–7 seconds). MNE's reader tolerates this and loads the data; stricter readers (such as pyedflib) reject the files. Correcting it means rewriting the `.edf` binary headers, which is beyond this text-only cleanup; a rehost would benefit from re-encoding the files with the correct record count.
