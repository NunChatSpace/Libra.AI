# LLD — User Story #1: Upload Book
## Sequence Diagram & API Specification

---

## ER diagram
```mermaid
erDiagram
    USERS ||--o{ BOOKS : uploads
    BOOKS ||--o{ ETL_JOBS : has
```

## Sequence Diagram — Upload Book

```mermaid
sequenceDiagram
    participant L as Librarian
    participant FE as Frontend
    participant BE as Backend API
    participant S3 as Object Storage
    participant DB as Database

    L->>FE: Fill book info + select PDF
    FE->>BE: POST /books
    BE->>DB: Create BOOK (status=uploading)
    BE->>S3: Generate presigned URL
    BE-->>FE: book_id + presigned URL

    FE->>S3: Upload PDF (PUT via presigned URL)
    S3-->>FE: Upload success

    FE->>BE: POST /books/{book_id}/upload-complete
    BE->>S3: Verify object exists
    BE->>DB: Update BOOK (status=uploaded)
    BE->>DB: Create ETL_JOB (status=pending)
```

---

## API Specification

### 1. Initialize Upload

**POST** `/books`

**Request**
```json
{
  "title": "Book Title",
  "file_name": "book.pdf",
  "file_size": 12345678,
  "content_type": "application/pdf"
}
```

**Behavior**
- Validate librarian permission
- Create BOOK record with status `uploading`
- Generate presigned upload URL

**Response**
```json
{
  "book_id": "uuid",
  "status": "uploading",
  "file_key": "books/uuid/original.pdf",
  "upload": {
    "presigned_url": "...",
    "method": "PUT",
    "expires_at": "2026-01-21T10:00:00Z"
  }
}
```

---

### 2. Upload Completion Callback

**POST** `/books/{book_id}/upload-complete`

**Request**
```json
{
  "file_key": "books/uuid/original.pdf"
}
```

**Behavior**
- Verify file exists in storage
- Update BOOK status to `uploaded`
- Create ETL_JOB with status `pending`

**Response**
```json
{
  "book_id": "uuid",
  "status": "uploaded"
}
```

---

### 3. Get Book Status

**GET** `/books/{book_id}`

**Response**
```json
{
  ... book's fields ...
}
```

---

## Notes & Constraints

- One PDF per book
- No file replacement after upload
- Upload completion callback must be idempotent
- ETL job starts only after upload verification
