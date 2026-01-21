# LLD — User Story #6: Batch Import from Folder
## Sequence Diagram & API / Worker Specification

---

## ER Diagram (Relevant Entities)
```mermaid
erDiagram
    BOOKS ||--o{ ETL_JOBS : has
```

---

## Sequence Diagram — Folder-based Batch Import

```mermaid
sequenceDiagram
    participant S3 as S3 Import Prefix
    participant W as Batch Import Worker
    participant DB as Database

    Note over W: Scheduled job (e.g. every N minutes)

    W->>S3: List objects under s3://bucket/import/BRANCH-XXX/
    S3-->>W: Return list of PDF objects

    loop For each object
        W->>W: Check file extension (.pdf only)
        alt Not PDF
            W->>DB: (optional) Insert ETL_JOB with status=failed (reason=unsupported_format)
            Note over W: Skip non-PDF object
        else PDF file
            W->>DB: Find BOOK by file_key (S3 object key)
            alt BOOK already exists
                Note over W: Idempotent — skip existing book
                W->>DB: (optional) Update existing ETL_JOB or log duplicate_file_key
            else No existing BOOK
                W->>DB: Create BOOK (title from file name,\nstatus=uploaded,\nfile_key = S3 key)
                W->>DB: Create ETL_JOB (job_type="full_etl", status=pending,\nbook_id = new book)
                Note over W,DB: Actual ETL (OCR → chunk → embedding)\nจะถูกรันโดย ETL Worker ตาม US#5
            end
        end
    end
```

---

## Worker Responsibilities
- Scan import folder on schedule
- Accept only `.pdf` files
- Detect duplicate file names
- Create BOOK + ETL_JOB per file
- Execute ETL independently per file
- Ensure idempotency (file processed once)

---

## Notes & Constraints
- File name is the source of initial book title
- Duplicate file names are rejected
- Processing order is not guaranteed
- Worker must be resilient to partial failures
- Metadata refinement is handled separately via US#4
