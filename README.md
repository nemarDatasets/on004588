[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004588-blue)](https://doi.org/10.82901/nemar.on004588)

 A novel multimodal Neuromarketing dataset that encompasses the data from 42 individuals who participated in an advertising brochure-browsing scenario is introduced here. In more detail, participants were exposed to a series of supermarket brochures (containing various products) and instructed to select the products they intended to buy. The data collected for each individual executing this protocol included: (i) encephalographic (EEG) recordings, (ii) eye tracking (ET) recordings, (iii) questionnaire responses (demographic, profiling and product related questions), and (iv) computer mouse data. 

The preprocessed version of this dataset can be found here: https://figshare.com/articles/dataset/NeuMa_PreProcessed_A_multimodal_Neuromarketing_dataset/22117124

## NEMAR curation changes (2026-05-21)

BIDS validator: 43 errors + 1850 warnings → 0 errors + 1764 warnings. Raw `.set`/`.fdt` binary payloads unchanged.

### `dataset_description.json`
- Added `DatasetType: "raw"`. Why: BIDS-validator otherwise infers a derivative-rules cascade when `DatasetType` is missing alongside `GeneratedBy`, producing spurious warnings.
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "https://github.com/nemar-org/nemar-cli"}]`. Why: records the NEMAR rehost step in the dataset's provenance chain.
- `ReferencesAndLinks`: `[""]` → `[]`. Why: an empty-string element is not a valid reference URL.

### `task-unnamed_events.json`
- File body rewritten from `[]` (an empty JSON array) to a proper JSON object documenting the two non-standard columns that every per-recording `_events.tsv` carries: `sample` and `value`. Why: BIDS requires sidecar JSON files to be objects, not arrays; the empty-array form triggered the `JSON_NOT_AN_OBJECT` error. The `value` column is declared as free-form text because the observed labels span ~80 distinct strings (`MouseButtonLeft pressed/released`, `MouseWheelDown120 pressed`, key-press events, etc.) — an enum would reject the long tail. Closes the 1× `JSON_NOT_AN_OBJECT` error.

### `participants.tsv`
- `participant_id` column: padded the single-digit IDs `sub-S1`, `sub-S2`, `sub-S3`, `sub-S5`, `sub-S6` to `sub-S01`, `sub-S02`, `sub-S03`, `sub-S05`, `sub-S06`. Why: every on-disk subject directory uses the zero-padded `sub-SNN` form; the unpadded TSV entries did not match the directories and fired `PARTICIPANT_ID_MISMATCH`. No other rows changed.

### `sub-S37/eeg/sub-S37_task-unnamed_events.tsv`
- Dropped the final data row. The trailing event at sample 200123 carried the corrupted value `'MouseButt\xbf>\xcf'` — the original write was truncated mid-string and the residual bytes were not valid UTF-8 (visible as `MouseButt�>�` when read with replacement). That fired `INVALID_FILE_ENCODING` and cascaded into `TSV_COLUMN_MISSING:onset` / `:duration` on the same file (3 errors). Trying to guess the intended label (likely `MouseButtonLeft pressed` based on the prior row's pattern, but not certain) would be invention; removing the unrecoverable row is the defensible mechanical option. The other 244 data rows are preserved unchanged.

### `.bidsignore`
- Two non-BIDS top-level resources are kept inside the dataset but excluded from validation: each subject's `eye_tracker/` modality directory (an EEGLAB-format `.set` file plus paired sidecars, used for eye-tracking data — BIDS-MEG has a `gaze` modality but the file shape here doesn't match the BIDS-canonical eye-tracking spec) and the dataset-root `QuestionnaireResponses/` folder.
- Patterns updated from the original `**/eye_tracker/**` (anchored, only matches files *under* the directory) to the unanchored triple `*/eye_tracker`, `*/eye_tracker/`, `*/eye_tracker/**` (covers both the directory entries and their contents). Same shape was added for `QuestionnaireResponses`. Why: anchored `**/.../**` patterns let the validator still flag the directory entry itself with `NOT_INCLUDED`; the unanchored form covers both. Closes the 42× `NOT_INCLUDED:/sub-NN/eye_tracker/` errors.