# Week 7 Assignment — Incremental Data Processing using Delta Lake

## Objective
Perform incremental data processing using Delta Lake on the Superstore dataset — load raw data, clean it, simulate incremental updates, and apply a MERGE operation to keep the target table in sync.

---

## 1. Unity Catalog Structure

**Catalog:** `dlt_assignement`
**Schema:** `default`

<img width="482" height="252" alt="Screenshot 2026-07-02 160423" src="https://github.com/user-attachments/assets/c77af67a-3c20-47b2-a466-50af4ca2a808" />


### Tables (2)

| Table Name        | Layer  | Description                                                              |
|--------------------|--------|---------------------------------------------------------------------------|
| `superstore_bronze` | Bronze | Raw, unclean data loaded directly from `Superstore.csv`. No transformations applied. |
| `superstore_silver` | Silver | Cleaned data (nulls handled, duplicates removed, correct data types applied) and the target table for the incremental MERGE. |

### Volumes (1)

| Volume Name     | Path                                                        |
|------------------|--------------------------------------------------------------|
| `assignment_vol` | `/Volumes/dlt_assignement/default/assignment_vol`             |

---

## 2. Raw Files in Volume (`RawVol`)

**Path:** `/Volumes/dlt_assignement/default/assignment_vol/RawVol`
<img width="1157" height="317" alt="Screenshot 2026-07-02 160402" src="https://github.com/user-attachments/assets/02e45e60-ac8f-4273-9f1a-67a77c65ddc7" />

---

## 3. Pipeline Flow

```
RawVol/Superstore.csv
        │
        ▼
  superstore_bronze   (raw, as-is)
        │
        ▼  (clean: handle nulls, remove duplicates, fix types)
  superstore_silver   (cleaned baseline)
        │
        ▼  (MERGE with Incremental_load.csv on Row ID)
  superstore_silver   (updated: existing rows updated, new rows inserted)
```

---

## 4. Layer Summary

- **Bronze (`superstore_bronze`)** — Untouched raw load from `Superstore.csv`. Serves as the single source of truth for reprocessing if needed.
- **Silver (`superstore_silver`)** — Cleaned dataset (data type corrections, null/duplicate handling) and the live target for incremental MERGE operations.
- **Incremental data (`Incremental_load.csv`)** — Not stored as a separate persistent table; read directly from the Volume, schema-aligned to Silver, and merged in using `Row ID` as the match key (`whenMatchedUpdateAll`, `whenNotMatchedInsertAll`).
