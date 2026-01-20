# LLD — User Story #3: Ask AI About Books

## Sequence Diagram & API Specification

---

## ER Diagram

```mermaid
erDiagram
    BOOKS ||--o{ PAGES : has
    PAGES ||--o{ CHUNKS : has
    CHUNKS ||--|| CHUNK_EMBEDDINGS : embedded_as
    BOOKS ||--o{ QNA_INCIDENT_LOGS : has_incidents
```

---

## Sequence Diagram — WebSocket Q&A Flow

```mermaid
sequenceDiagram
    participant G as Guest
    participant FE as Frontend
    participant BE as Backend API
    participant DB as Database
    participant LLM as LLM Service

    Note over FE,BE: WebSocket already connected (or will connect on first question)

    G->>FE: Type question & press Enter
    FE->>BE: WS Message { type: "question", content: "..." }

    BE->>DB: Query books with status = ready
    DB-->>BE: Ready books (or none)

    alt No ready books
        BE-->>FE: WS Message { type: "error", code: "NO_BOOKS_AVAILABLE", ... }
    else Ready books exist
        Note over BE,DB: Retrieval over only ready books\n(using vector search on CHUNK_EMBEDDINGS)

        BE->>DB: Vector search relevant chunks (question + optional history, filtered by ready books)
        DB-->>BE: Relevant chunks with book/page refs

        Note over BE,LLM: Single LLM call:\n- ให้ LLM เลือก Q&A / Summary / Search / Location\n- ใช้ question + conversation history + retrieved chunks\n- Prompt บังคับให้ตอบจากเนื้อหาหนังสือเท่านั้น

        BE->>LLM: Stream generate (question + history + chunks)
        
        loop Streaming Response
            LLM-->>BE: Token chunk
            BE-->>FE: WS Message { type: "stream", content: "token..." }
            FE-->>G: Append token to UI
        end

        LLM-->>BE: Stream complete (with final answer + sources)
        BE->>DB: (optional) Insert QNA log (success/latency, etc.)
        BE-->>FE: WS Message { type: "complete", sources: [...], qna_log_id: "uuid", latency_ms: ... }
        FE-->>G: Render final answer + sources
    end

    Note over FE,BE: If no activity for N seconds → server may close WS connection (idle timeout)
```

---

## WebSocket Specification

### 1. Connect to Chat

**WebSocket** `ws://host/ws/chat`

**Connection Behavior**

- Open connection when user sends first message
- Maintain connection for streaming responses
- Search across all books with status `ready`

**Connection Success Response**

```json
{
  "type": "connected",
  "session_id": "uuid"
}
```

---

### 2. Client Messages

#### Ask Question

```json
{
  "type": "question",
  "content": "What is machine learning?"
}
```

---

### 3. Server Messages

#### Streaming Token

```json
{
  "type": "stream",
  "content": "Machine learning is"
}
```

#### Stream Complete

```json
{
  "type": "complete",
  "qna_log_id": "uuid",
  "sources": [
    {
      "book_id": "uuid",
      "book_title": "Introduction to ML",
      "page_start": 45,
      "page_end": 45,
      "chunk_id": "uuid",
      "snippet": "...relevant text excerpt...",
      "relevance_score": 0.92
    },
    {
      "book_id": "uuid",
      "book_title": "Deep Learning Basics",
      "page_start": 12,
      "page_end": 13,
      "chunk_id": "uuid",
      "snippet": "...another relevant excerpt...",
      "relevance_score": 0.87
    }
  ],
  "latency_ms": 1250
}
```

#### Error

```json
{
  "type": "error",
  "code": "NO_RELEVANT_CONTENT",
  "message": "No relevant content found for your question"
}
```

---

### 4. Error Codes

| Code | Description |
| ---- | ----------- |
| NO_RELEVANT_CONTENT | No relevant chunks found |
| NO_BOOKS_AVAILABLE | No books with ready status |
| LLM_ERROR | LLM service error |
| INVALID_MESSAGE | Invalid message format |
| CONNECTION_TIMEOUT | WebSocket connection timeout |

---

## QNA_INCIDENT_LOGS Storage

**Purpose**: Log only negative feedback for improvement tracking

**Logged Data**

| Field | Description |
| ----- | ----------- |
| book_id | Reference to the book (from sources) |
| question_text | User's question |
| answer_text | Generated answer |
| feedback_text | User's negative feedback |
| latency_ms | Response generation time |
| created_at | Timestamp |

**Note**: Only logged when user submits negative feedback (thumbs down)

---

## Notes & Constraints

- Search across all books with status `ready`
- No book selection required - user just types and sends
- WebSocket opens on first message (Enter key)
- Streaming enables real-time token-by-token display
- Sources include book info (title, page) for reference
- If no relevant content found, return appropriate error message
- Questions outside book content scope return graceful fallback response
- โหมดการตอบ (Q&A, Summary, Search, Location) ถูกตัดสินใจโดย LLM จากคำถาม + context:
  - Backend ไม่รับ parameter `mode` จาก client
  - Backend ไม่ทำ LLM call แยกเพื่อ classify โหมด
  - การเลือกโหมดถูกควบคุมผ่าน system prompt ฝั่ง LLM (1 call ต่อคำถาม)
- หากไม่พบเนื้อหาที่เกี่ยวข้อง:
  - Backend ไม่ fallback ไปตอบความรู้ทั่วไปของ LLM
  - ต้องส่ง error `NO_RELEVANT_CONTENT` หรือข้อความ fallback ที่ระบุชัดว่าไม่มีเนื้อหาจากหนังสือ
