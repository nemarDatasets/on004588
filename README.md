[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004588-blue)](https://doi.org/10.82901/nemar.on004588)

 A novel multimodal Neuromarketing dataset that encompasses the data from 42 individuals who participated in an advertising brochure-browsing scenario is introduced here. In more detail, participants were exposed to a series of supermarket brochures (containing various products) and instructed to select the products they intended to buy. The data collected for each individual executing this protocol included: (i) encephalographic (EEG) recordings, (ii) eye tracking (ET) recordings, (iii) questionnaire responses (demographic, profiling and product related questions), and (iv) computer mouse data. 

The preprocessed version of this dataset can be found here: https://figshare.com/articles/dataset/NeuMa_PreProcessed_A_multimodal_Neuromarketing_dataset/22117124

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 43 errors + 1850 warnings to 0 errors + 1765 warnings. None of the raw `.set`/`.fdt` binary payloads were modified; every change is to a text sidecar or to the validation-ignore list.

**Dataset description (`dataset_description.json`)**
- Added `DatasetType: "raw"` so the dataset is validated as raw data rather than a derivative; without it the validator falls through to derivative-rules checks and emits cascading warnings.
- Updated `BIDSVersion` from `1.2` to `1.11.1` (the version the current validator checks against).
- `GeneratedBy` was left absent, exactly as the source published it. Nothing was added there.
- `ReferencesAndLinks` was `[""]` (a one-element list containing an empty string, which is not a usable URL); it is now `[]`.

**Root events sidecar (`task-unnamed_events.json`)**
- The file was an empty JSON array (`[]`). BIDS requires sidecar JSON files to be objects, not arrays, so this fired a top-level error. It was rewritten as a proper JSON object documenting the two non-standard columns that every per-recording events table carries: `sample` (the sample index of the event) and `value` (the event label). The `value` column is declared as free-form text because the observed labels span roughly 80 distinct strings (mouse-button presses and releases, mouse-wheel ticks, key-press events), which an enum would not cover.

**Participants table (`participants.tsv`)**
- The `participant_id` column listed `sub-S1`, `sub-S2`, `sub-S3`, `sub-S5`, and `sub-S6` (single-digit IDs), but every on-disk subject directory uses the zero-padded `sub-SNN` form. The five unpadded entries were padded to `sub-S01`, `sub-S02`, `sub-S03`, `sub-S05`, and `sub-S06` so the TSV matches the directories. No other rows changed.

**Subject S37 events table (`sub-S37/eeg/sub-S37_task-unnamed_events.tsv`)**
- The final data row at sample 200123 carried a corrupted value (`MouseButt` followed by stray bytes `\xbf>\xcf` that are not valid UTF-8). The original write appears to have been truncated mid-string. That single row failed encoding validation and cascaded into missing-column errors on the whole file. The corrupted row was dropped; the likely intended label (`MouseButtonLeft pressed`, by analogy with the prior row) was not invented in. The other 244 data rows are preserved unchanged.

**Validation-ignore list (`.bidsignore`)**
- Each subject keeps a non-BIDS `eye_tracker/` modality directory alongside `eeg/` (an EEGLAB-format `.set` file plus paired sidecars holding the eye-tracker recording), and the dataset root keeps a `QuestionnaireResponses/` folder. Neither shape matches a BIDS modality, so both belong on the ignore list.
- The original patterns were anchored (`**/eye_tracker/**`), which only matches files inside the directory and lets the validator still flag the directory entry itself as not-included. The patterns were broadened to the unanchored triple `*/eye_tracker`, `*/eye_tracker/`, `*/eye_tracker/**`, and the same shape was added for `QuestionnaireResponses`, so both the directory entries and their contents are excluded.

**Left untouched (out of mechanical scope)**
- Recording-level recommended metadata (manufacturer, model, software version, serial number, task description, instructions, cognitive-atlas / CogPO URIs, institution name and address, and similar fields across all subject `_eeg.json` sidecars) was left blank rather than filled with guesses. These are study-specific facts that need the original lab to confirm.

**Harness note**
- The eeg-verify harness bundle for this dataset is partial: a UnicodeDecodeError killed the git-audit stage mid-pass (a separate harness bug, tracked upstream). The validator output reproduced above was generated against the on-disk curated tree and is the trustworthy record of what changed.

**Remaining warnings (1765) — left on purpose**
- These are all "recommended but missing" fields: 1722 sidecar entries for the per-recording fields listed above, 42 event-onset-ordering notices on per-subject events tables (a soft warning, not a structural defect), and 1 dataset-level entry flagging that `GeneratedBy` is recommended but absent. `GeneratedBy` was deliberately not added so the dataset reflects the source publication; the warning is expected.