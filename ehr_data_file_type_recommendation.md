
# Recommendations for EHR File Format for Submission to Google Cloud Storage

- Both `CSV` and `JSONL` file formats are compatible with the Google Cloud Storage API and can be ingested into BigQuery in appropriate table format. 
- My preference is to receive the data in `JSONL` form. If legitimate concerns/objections are raised, `CSV` is acceptable.
    - Large CSV files can cause headaches if commas appear within a cell or if the stream of text data is broken somewhere.
    - JSONL is more robust for streaming large uploads.
- All sites should submit their data via the same format.
- All-of-Us has probably experienced all possible issues with this, so consulting with them could make sense.

## Preferred Format: JSONL

In `JSONL` format, each object must be delimited by a newline.

**Example `JSONL` Format for OMOP CDM's `condition_occurrence` Table:**

```jsonl
{ "condition_occurrence_id": 1, "person_id": 1, "condition_concept_id": 319835, ... }
{ "condition_occurrence_id": 2, "person_id": 2, "condition_concept_id": 319835, ... }
{ "condition_occurrence_id": 3, "person_id": 3, "condition_concept_id": 433736, ... }
```

> **Note:**  You can omit the field entirely if the value is missing. Alternatively, use null to explicitly indicate missing values.

## Alternative Format: CSV

**Example `CSV` Format for OMOP CDM's `condition_occurrence` Table:**

```csv
condition_occurrence_id,person_id,condition_concept_id,condition_start_date,condition_end_date,condition_type_concept_id,stop_reason
1,1,319835,2023-05-23,2023-05-30,38000245,NULL
2,2,319835,2023-06-01,2023-06-10,38000245,NULL
3,3,433736,2023-07-05,2023-07-12,38000245,NULL
```
> **Note:** Leave the field empty (e.g. `, ,` for a missing field between two commas).



## References: 
- https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-json