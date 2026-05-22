[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on003195-blue)](https://doi.org/10.82901/nemar.on003195)

The quest for neuroprotection in Parkinson’s disease (PD) has been for new compounds to slow disease progression and stable and non-invasive biomarkers to document their benefits. Neuroepo, a new formulation of EPO with low content of sialic acid reported good results in animal model and tolerance in healthy participants and PD patients. 
In a double-blind randomized placebo (https://clinicaltrials.gov/ number NCT04110678) twenty-five PD patients were assigned randomly to Neuroepo (n=15) or placebo (n=10) groups we reported the tolerance of the drug. We recorded resting-state EEG before and six months after the administration of the drug. The qualitative analysis of the abnormalities of the EEG was evaluated by two experts using a Likert-type scale and a multivariate item response theory (MIRT) approach was employed to stablish the differences between groups in the two times. The quantitative EEG (qEEG) analysis was performed at the sources looking for generators of the neural activity using software VARETA and co-registering the results using the Montreal Neurological Institute Atlas. The statistical analysis between the sources was conducted using a permutation test and later a contrast method using the surfstat software between groups and before vs after condition, with Bonferroni correction for multiple comparisons.
Here in this repository we placed the raw EEG in BIDS format  (Pernet, C. R. et al. EEG-BIDS, an extension to the brain imaging data structure for electroencephalography. Sci. data 6, 103 (2019). For the use of VARETA the qEEG program you can use (Bosch-Bayard, J. et al. A Quantitative EEG Toolbox for the MNI Neuroinformatics Ecosystem: Normative SPM of EEG Source Spectra. Front. Neuroinform. 14, (2020).)
The EEG dataset from the different stages of processing can be requested to the authors. 

## NEMAR curation changes (2026-05-21)

BIDS validator: 1 error + 423 warnings → 0 errors + 341 warnings. Raw `.edf` binary payloads unchanged.

### `participants.tsv`
- Stripped a trailing tab on every line (header was `participant_id\tsex\t` with an empty third column) and converted CRLF line endings to LF. Why: the trailing tab introduced an unnamed third column that the validator flagged as `TSV_ADDITIONAL_COLUMNS_UNDEFINED` with an empty `subCode`; cleaning the header removes the phantom column without changing any participant data.

### `participants.json` (new)
- Declared the `participant_id` column and the `sex` column with explicit `Levels` for the three observed codes (`0`, `1`, `6`). Why: the source dataset codes `sex` numerically (values `0`/`1`/`6` appear in the TSV body), which fired `TSV_VALUE_INCORRECT_TYPE:sex` against the BIDS-standard `F/M/O/n/a` enum. The numeric-to-category mapping is not documented in the source files, so the sidecar declares the codes verbatim and points the reader to the source publication; `Levels` is intentionally limited to the values that actually appear in the TSV. Value `6` is noted as a singleton (sub-PLAC17) with no source explanation. No mapping was invented.

### `scans.json` (new, root inheriting)
- Declared the non-standard `age_acq_time` column with a Description that records what the values look like rather than inventing a unit. Why: every per-session `_scans.tsv` carries this column (firing 20× `TSV_ADDITIONAL_COLUMNS_UNDEFINED:age_acq_time`); 19 of 20 entries are clock times in `HH:MM:SS` format and one (sub-PLAC17 ses-placebopre) is the decimal `69.39` whose semantics are undocumented. The column name combines two concepts ("age" and "acquisition time") and the source dataset does not say which is intended, so no `Units` is asserted and the Description captures the ambiguity.

### `task-taskplacebopre_events.json` and `task-taskplacebo6m_events.json` (new, root inheriting per task)
- Declared the `value` and `sample` columns (both standard MNE-BIDS export columns) and the `trial_type` column (free-form, no `Levels`). Why: every per-recording `_events.tsv` carries `value` and `sample` (firing 20× each of `TSV_ADDITIONAL_COLUMNS_UNDEFINED:value` and `:sample`); declaring them once at the root closes all 40 warnings via BIDS inheritance. The `trial_type` column carries 12 distinct free-form Spanish labels (`photic stimulation`, `ojos cerrados`, `ojos abiertos`, `hiperventilacion 1`-`4`, `recuperacion`, `discontinuity`, `fotoestimulacion`, `HPVOJOS ABIERTOS`, `hipervantilacion 4`) — too varied (and typo-laden) to enumerate as a `Levels` map without rejecting unmapped rows, so the observed vocabulary is described in the column Description and the column stays free-form.

### `dataset_description.json`
- Added `"DatasetType": "raw"`. Why: the validator otherwise infers derivative-rules when `GeneratedBy` is present without `DatasetType`, producing spurious `Description` cascades; pairing the two avoids that mode.
- Added a `GeneratedBy` entry recording `nemar-cli`. Why: documents curation provenance and clears `JSON_KEY_RECOMMENDED:GeneratedBy`.
- Bumped `BIDSVersion` `1.2.0` → `1.8.0`. Why: BIDSVersion < 1.8.0 fires `UNKNOWN_BIDS_VERSION` on newer validators; 1.8.0 fully supports the BIDS-EEG fields used here.

### All 20 per-recording `_eeg.json` files (`sub-*/ses-*/eeg/*_eeg.json`)
- Renamed the key `MiscChannelCount` (camelCase) to `MISCChannelCount` (BIDS-canonical, uppercase `MISC`). Why: the validator does not recognise the camelCase variant, so it kept firing 20× `SIDECAR_KEY_RECOMMENDED:MISCChannelCount` even though every file already declared the correct value of `0`. The rename closes all 20 warnings; values were preserved unchanged.

### Out of mechanical scope (remaining 341 warnings)

All remaining warnings are equipment/lab/study context not present in this dataset:

- 17 × 20 `SIDECAR_KEY_RECOMMENDED` for `Manufacturer`, `ManufacturersModelName`, `SoftwareVersions`, `DeviceSerialNumber`, `Instructions`, `CogAtlasID`, `CogPOID`, `InstitutionName`, `InstitutionAddress`, `InstitutionalDepartmentName`, `CapManufacturer`, `CapManufacturersModelName`, `EEGGround`, `HeadCircumference`, `HardwareFilters`, `SubjectArtefactDescription`, `StimulusPresentation`. These require information not documented in the dataset (amplifier hardware, lab/institution, study protocol, per-subject anthropometry). NEMAR curation policy is to leave these as warnings rather than invent values.
- 1 × `JSON_KEY_RECOMMENDED:HEDVersion` on `dataset_description.json`. This dataset does not use HED tags, so the field is not applicable.

### Notes for re-hosting

- An upstream-pipeline finding from the eeg-verify sweep (not addressed by this curation pass): all 20 `.edf` files have a header `n_records` count that overstates the actual payload by a fractional record (1-7 seconds of nominal duration). MNE's lenient EDF reader truncates and proceeds; `pyedflib`'s strict reader rejects every file. Touching the binary `.edf` headers is outside the mechanical-sidecar curation scope and was deliberately not attempted here; rehost would benefit from re-encoding the EDF files with a correct `n_records` field.
