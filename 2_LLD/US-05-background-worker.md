# LLD — User Story #5: Background Worker (ETL Processing)
## Sequence Diagram & API Specification

---

## ER diagram
```mermaid
erDiagram
    BOOKS ||--o{ ETL_JOBS : has
    BOOKS ||--o{ PAGES : generates
    PAGES ||--o{ CHUNKS : splits_into
    CHUNKS ||--|| CHUNK_EMBEDDINGS : embedded_as
```

## Sequence Diagram — Background Processing (ETL)

```mermaid
sequenceDiagram
    participant W as Background Worker
    participant DB as Database
    participant S3 as Object Storage
    participant V as Vector Store

    W->>DB: Poll ETL_JOBS (status=pending)
    DB-->>W: ETL_JOB

    W->>DB: Update ETL_JOB (status=running)
    W->>DB: Update BOOK (status=processing)

    W->>S3: Download PDF by file_key
    S3-->>W: PDF file

    Note over W: OCR entire PDF Store text per page

    W->>DB: Insert PAGES (ocr_text, page_number)

    Note over W: Split pages into chunks

    W->>DB: Insert CHUNKS (page_start, page_end, content)

    Note over W: Generate embeddings for chunks

    W->>V: Write embeddings
    V-->>W: vector_ids

    W->>DB: Insert CHUNK_EMBEDDINGS

    alt Success
        W->>DB: Update ETL_JOB (status=done, progress=100)
        W->>DB: Update BOOK (status=ready)
    else Failure
        W->>DB: Update ETL_JOB (status=failed)
        W->>DB: Update BOOK (status=failed)
    end
```

---

## Worker Behavior Specification

### Job Pickup Rules
- Worker must pick only ETL_JOBS with status `pending`
- Job selection must be atomic to avoid duplicate processing
- One ETL_JOB is processed by only one worker at a time

### Processing Steps
1. Download PDF from object storage
2. Perform OCR on entire document
3. Store OCR results per page
4. Split pages into chunks
5. Generate embeddings for each chunk
6. Persist embeddings and metadata
7. Update job and book status

### Progress Reporting
- Worker must update `ETL_JOBS.progress` periodically
- Progress represents overall ETL completion percentage
- Progress values must be monotonic (never decrease)

### Error Handling
- Any unrecoverable error marks:
  - `ETL_JOBS.status = failed`
  - `BOOKS.status = failed`
- Error details must be stored in `ETL_JOBS.error_message`
- Partial results may remain for debugging

---

## Constraints & Assumptions

- Worker operates asynchronously and independently from API requests
- OCR, chunking, and embedding are executed offline
- Worker does not accept direct external requests
- Retry strategy (manual or automatic) is out of scope for this user story
