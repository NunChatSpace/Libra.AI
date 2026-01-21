# AI Librarian Project — High-Level Specification (Draft)

## Project Overview
- Project: AI Librarian for Library
- Goal: Enable users to search, locate, and summarize books using AI
- Primary Users:
  - Guest
  - Librarian

---

## Input / Data
- Physical books scanned into PDF format
- PDFs are mostly scanned images (OCR required)
- Primary language: Thai (may include others)
- Large volume of books (must support scaling)

---

## Processing (Backend / AI)
- OCR to convert PDF pages into text
- Store text at multiple levels:
  - Book
  - Page
  - Chunk
- Generate embeddings from text chunks
- Store embeddings and metadata in database
- ETL / background workers separated from API services

---

## Knowledge Constraints
- AI answers must be grounded strictly in book content
- Only books with status `ready` can be used for answering
- Answers must be traceable to:
  - Book
  - Page number

---

## User Capabilities

### Guest
- Ask natural-language questions
- Does not need to know book titles in advance
- Common questions:
  - Where is this book located?
  - Which book contains content like this?
  - Summarize this book or part of it

### Librarian
- Upload book PDFs
- Edit book metadata (title, category, tags)
- View processing status of books

---

## Interaction
- Chat-based UI (WebSocket with streaming responses)
- AI automatically determines response mode:
  - Q&A
  - Summary
  - Search
  - Location
- Idle WebSocket connections may be closed automatically

---

## Output
- Human-readable answers suitable for children
- Source references (book and page)
- Physical book location (if metadata is available)

---

## Non-goals / Out of Scope
- Not a general-purpose AI chatbot
- No editing of PDF content
- No reprocessing triggered by metadata changes
- No real-time collaboration features
- Limited analytics (negative feedback only)

---

## Notes
- This document serves as a baseline specification
- Used as input for WBS, timeline estimation, and resource planning
