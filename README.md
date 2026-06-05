[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on004588-blue)](https://doi.org/10.82901/nemar.on004588)

 A novel multimodal Neuromarketing dataset that encompasses the data from 42 individuals who participated in an advertising brochure-browsing scenario is introduced here. In more detail, participants were exposed to a series of supermarket brochures (containing various products) and instructed to select the products they intended to buy. The data collected for each individual executing this protocol included: (i) encephalographic (EEG) recordings, (ii) eye tracking (ET) recordings, (iii) questionnaire responses (demographic, profiling and product related questions), and (iv) computer mouse data. 

The preprocessed version of this dataset can be found here: https://figshare.com/articles/dataset/NeuMa_PreProcessed_A_multimodal_Neuromarketing_dataset/22117124

## NEMAR curation changes

Changes applied to the OpenNeuro source during the NEMAR rehost. The BIDS validator goes from 43 errors + 1850 warnings to 0 errors + 1765 warnings; the residual warnings are recommended-but-missing study-specific fields that cannot be filled without information from the original lab. None of the raw `.set` / `.fdt` binary payloads are modified; every change is to a text sidecar or to the validation-ignore list.

**Dataset description (`dataset_description.json`)**
- `DatasetType: "raw"` is added so the dataset is validated as raw data rather than a derivative; without it the validator falls through to derivative-rules checks and emits cascading warnings.
- `BIDSVersion` is set to `1.11.1`, the version the current validator checks against.
- `GeneratedBy` is absent, matching the source. Nothing is added there.
- `ReferencesAndLinks` was `[""]` (a one-element list containing an empty string, not a usable URL); it is now `[]`.

**Root events sidecar (`task-unnamed_events.json`)**
- The source file was an empty JSON array (`[]`), which BIDS rejects because sidecar JSON files must be objects. It is rewritten as a proper JSON object documenting every column actually present in the per-recording `events.tsv` tables (see below).
- Documents `sample` (sample index of the event) and `value` (verbatim event label as recorded in the source acquisition; carries three vocabularies — stimulus presentation strings of form `Category:IMG=ID:<file>=Type:<category>`, mouse / mouse-wheel events of form `<device> <action>`, and atomic trial markers `fixation_cross` / `EOE`).
- Also documents the three derived columns added to every `events.tsv` (see next bullet group).

**Events tables (`*_events.tsv`, 84 files: 42 subjects × `eeg/` + `eye_tracker/`)**
- Three columns are appended to every row to make the structure of `value` machine-readable without losing the original string:
  - `event_class`: one of `stimulus`, `response`, `marker`. Pure regex classification of `value`.
  - `stim_id`: for `event_class=stimulus` rows, the filename from the `ID:` field of `value` (e.g. `FYLLADIO_1.tif`). `n/a` for response and marker rows.
  - `stim_category`: for `event_class=stimulus` rows, the category label from the `Type:` field of `value` (e.g. `Leaflet_Images_1`). `n/a` for response and marker rows.
- Every new cell is either a literal substring of the original `value` or a class label that follows unambiguously from the regex match. No semantic interpretation, no invented metadata.
- Row distribution across the dataset: 902 stimulus, 3741 response, 168 marker (4811 rows total touched). The original `value` column is preserved verbatim alongside the new columns.

**Subject S37 events table (`sub-S37/eeg/sub-S37_task-unnamed_events.tsv`)**
- The final source row at sample 200123 carried a corrupted value (`MouseButt` followed by stray bytes `\xbf>\xcf`, not valid UTF-8) — the original write was truncated mid-string. That single row failed encoding validation and cascaded into missing-column errors on the whole file. The corrupted row is dropped; the likely intended label (`MouseButtonLeft pressed`, by analogy with the prior row) is not invented. The other 244 data rows are preserved unchanged.

**Participants table (`participants.tsv`)**
- The `participant_id` column listed `sub-S1`, `sub-S2`, `sub-S3`, `sub-S5`, and `sub-S6` (single-digit IDs), but every on-disk subject directory uses the zero-padded `sub-SNN` form. The five unpadded entries are padded to `sub-S01`, `sub-S02`, `sub-S03`, `sub-S05`, and `sub-S06` so the TSV matches the directories. No other rows changed.

**Validation-ignore list (`.bidsignore`)**
- Each subject keeps a non-BIDS `eye_tracker/` modality directory alongside `eeg/` (an EEGLAB-format `.set` file plus paired sidecars holding the eye-tracker recording), and the dataset root keeps a `QuestionnaireResponses/` folder. Neither shape matches a BIDS modality, so both belong on the ignore list.
- The original anchored patterns (`**/eye_tracker/**`) only matched files inside the directory and let the validator still flag the directory entry itself as not-included. The patterns are broadened to the unanchored triple `*/eye_tracker`, `*/eye_tracker/`, `*/eye_tracker/**`, and the same shape is added for `QuestionnaireResponses`, so both the directory entries and their contents are excluded.

**Left untouched**
- Recording-level recommended metadata (manufacturer, model, software version, serial number, task description, instructions, cognitive-atlas / CogPO URIs, institution name and address, and similar fields across all subject `_eeg.json` sidecars) is left blank rather than filled with guesses. These are study-specific facts that need the original lab to confirm.

**Remaining warnings (1765)**
- All "recommended but missing" fields: 1722 sidecar entries for the per-recording fields listed above, 42 event-onset-ordering notices on per-subject events tables (a soft warning, not a structural defect), and 1 dataset-level entry flagging that `GeneratedBy` is recommended but absent. `GeneratedBy` is deliberately not added so the dataset reflects the source publication; the warning is expected.
