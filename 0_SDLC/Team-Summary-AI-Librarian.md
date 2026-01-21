# Team Summary — AI Librarian System

## Team Composition

| Role | Key Responsibilities | Why Needed | Estimated Allocation |
|------|----------------------|------------|----------------------|
| Backend Engineer | ออกแบบและพัฒนา API (Books, ETL, Q&A, Metadata)<br>พัฒนา batch import worker<br>ควบคุม ETL flow (status, retry, resume)<br>เชื่อม OCR, pgvector, LLM | เป็นแกนหลักของระบบทั้งหมด โดยเฉพาะ ETL และ Q&A | 1–2 คน (Full-time) |
| AI / ETL Engineer | วางกลยุทธ์ OCR และควบคุมคุณภาพผลลัพธ์<br>วิเคราะห์ layout และ chapter detection<br>ออกแบบ chunking heuristic<br>ออกแบบ prompt และ retrieval strategy<br>tune ระบบจากการใช้งานจริง | งานส่วนนี้ไม่ deterministic หากไม่มี role นี้ backend จะตันเร็ว | 1 คน (Full-time หรือ Part-time หนัก) |
| Frontend Engineer | พัฒนา UI upload / book list / metadata<br>พัฒนา chat UI พร้อม streaming response<br>ออกแบบ UX สำหรับ processing และ error state | ระบบเป็น user-facing หาก UX แย่ ระบบจะไม่ถูกใช้งาน | 1 คน (Full-time) |
| Infrastructure / DevOps | ตั้งค่า Object Storage (S3 / MinIO)<br>ตั้งค่า PostgreSQL + pgvector<br>ดูแล worker runtime และ queue<br>ทำ CI/CD, monitoring, retention policy | ETL และ vector workload หนัก หาก infra ไม่ดี ระบบจะไม่เสถียร | 0.5–1 คน (Shared / Part-time) |
| Product / Librarian Representative | ตรวจสอบผลลัพธ์ AI ว่าใช้งานได้จริง<br>ให้ feedback เรื่อง summary / search / location<br>ทำ User Acceptance Testing (UAT) | ความถูกต้องของ AI เป็นเรื่อง business ไม่ใช่ technical อย่างเดียว | Part-time |

---

## Team Size Summary

| Area | Minimum Setup | Safer Setup |
|------|---------------|-------------|
| Backend | 1 | 2 |
| AI / ETL | 1 | 1 |
| Frontend | 1 | 1 |
| Infrastructure | 0.5 | 1 |
| Product | Part-time | Part-time |

**ทีมขั้นต่ำ**: ประมาณ 4 คน  
**ทีมที่ทำงานได้ไม่เสี่ยงเกินไป**: ประมาณ 5 คน (infra shared)

---

## Notes

- หากไม่มี AI / ETL Engineer → timeline จะคลาดเคลื่อนสูง
- Frontend สามารถทำขนานได้ แต่จะรอ ETL เป็นหลัก
- Infra ไม่จำเป็นต้อง full-time แต่ต้องพร้อมเข้ามาแก้ปัญหาได้เร็ว
