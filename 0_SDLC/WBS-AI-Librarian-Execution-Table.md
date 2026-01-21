# WBS Execution Table — AI Librarian System

| WBS Title | WBS ID | Topic | Scope Summary | Est. Man-Day | Role(s) | Dependency | Risk |
|-----------|--------|-------|---------------|--------------|---------|------------|------|
| Data Preparation & Ingestion | 1.1 | Source Preparation | Storage setup, naming convention, duplicate check, permission, idempotency | 5 | Backend, Infra | - | Medium |
| Data Preparation & Ingestion | 1.2 | Batch Import Worker | Scan storage prefix, create BOOK + ETL_JOB, error handling | 7 | Backend | 1.1 | Medium |
| ETL & Document Processing | 2.1 | OCR Processing | OCR scanned PDF, text normalization | 12 | Backend, AI | 1.2 | High |
| ETL & Document Processing | 2.2 | Page Extraction | Split OCR result into pages | 4 | Backend | 2.1 | Medium |
| ETL & Document Processing | 2.3 | Layout Analysis | Analyze text blocks, font size, position | 8 | Backend, AI | 2.2 | High |
| ETL & Document Processing | 2.4 | Chapter Detection | Detect chapter candidates | 6 | AI | 2.3 | High |
| ETL & Document Processing | 2.5 | Chunking | Chunk text by page / heuristic | 5 | Backend | 2.4 | Medium |
| ETL & Document Processing | 2.6 | ETL Tuning | Improve heuristic & quality | 10 | Backend, AI | 2.5 | High |
| ETL & Document Processing | 2.7 | Embedding Generation | Generate embeddings from chunks | 4 | Backend | 2.6 | Medium |
| ETL & Document Processing | 2.8 | Vector Storage | Store embeddings in pgvector | 3 | Backend | 2.7 | Medium |
| ETL & Document Processing | 2.9 | ETL Control Flow | Status update, retry / resume | 5 | Backend | 2.1 | Medium |
| AI Question & Answer System | 3.1 | Prompt Design | System + instruction prompt | 4 | AI | 2.8 | Medium |
| AI Question & Answer System | 3.2 | Retrieval Logic | Vector search (ready books only) | 6 | Backend | 2.8 | Medium |
| AI Question & Answer System | 3.3 | LLM Integration | Context + history to LLM | 5 | Backend | 3.2 | Medium |
| AI Question & Answer System | 3.4 | Streaming Response | Token streaming via WebSocket | 4 | Backend | 3.3 | Medium |
| AI Question & Answer System | 3.5 | Mode Decision | LLM decides Q&A / Summary / Search | 5 | AI | 3.1 | High |
| Metadata Management | 4.1 | Metadata CRUD | Edit title, description, tag, category | 4 | Backend | - | Low |
| Metadata Management | 4.2 | Metadata UI | Metadata edit UI | 3 | Frontend | 4.1 | Low |
| Feedback & Incident Tracking | 5.1 | Incident Logging | Log negative feedback only | 3 | Backend | 3.4 | Low |
| Feedback & Incident Tracking | 5.2 | Feedback Analysis | Use incident data for future tuning | 2 | AI | 5.1 | Low |
| Frontend Development | 6.1 | Upload UI | Upload page & status | 4 | Frontend | 1.2 | Medium |
| Frontend Development | 6.2 | Book List UI | List + status view | 3 | Frontend | 1.2 | Low |
| Frontend Development | 6.3 | Metadata UI | Edit metadata page | 3 | Frontend | 4.1 | Low |
| Frontend Development | 6.4 | Chat UI | Chat interface + streaming | 5 | Frontend | 3.4 | Medium |
| Frontend Development | 6.5 | UX States | Error / processing / empty states | 3 | Frontend | 6.4 | Low |
| Infrastructure & Deployment | 7.1 | Storage Setup | Object storage provisioning | 2 | Infra | - | Low |
| Infrastructure & Deployment | 7.2 | OLTP DB Setup | Main database setup | 2 | Infra | - | Low |
| Infrastructure & Deployment | 7.3 | Vector DB Setup | pgvector setup | 3 | Infra | 7.2 | Medium |
| Infrastructure & Deployment | 7.4 | Vector Scaling Plan | Plan separation of vector workload | 4 | Infra | 7.3 | Medium |
| Infrastructure & Deployment | 7.5 | Worker Setup | Background worker infra | 3 | Infra | 1.2 | Medium |
| Infrastructure & Deployment | 7.6 | CI/CD | Pipeline setup | 4 | Infra | - | Medium |
| Infrastructure & Deployment | 7.7 | Env Separation | dev / staging / prod | 3 | Infra | 7.6 | Medium |
| Infrastructure & Deployment | 7.8 | Monitoring & Logging | Metrics, logs, alert | 4 | Infra | 7.5 | Medium |
| Infrastructure & Deployment | 7.9 | Retention Policy | Storage cleanup & retention rules | 3 | Infra | 7.1 | Medium |
| Testing & Quality Assurance | 8.1 | Unit Testing | Backend / worker unit tests | 5 | Backend | Core modules | Medium |
| Testing & Quality Assurance | 8.2 | Integration Testing | ETL end-to-end test | 6 | Backend | 2.x | High |
| Testing & Quality Assurance | 8.3 | Load Testing | Q&A + vector search | 4 | Backend | 3.x | Medium |
| Testing & Quality Assurance | 8.4 | Failure Testing | OCR fail, ETL stuck, LLM error | 4 | Backend | 2.x, 3.x | High |
| Testing & Quality Assurance | 8.5 | UAT | Librarian validation | 3 | Product, QA | All | Medium |
