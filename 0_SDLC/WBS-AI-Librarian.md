# Work Breakdown Structure (WBS) — AI Librarian System

---

## 1. Data Preparation & Ingestion
### 1.1 Source Preparation
- จัดเก็บไฟล์ลง Object Storage (MinIO / S3)
- กำหนด naming convention ของไฟล์
- ตรวจสอบ duplicate จากชื่อไฟล์
- กำหนดสิทธิ์การเข้าถึงโฟลเดอร์ (read / write / scan source)
- ออกแบบ idempotency สำหรับการกวาดไฟล์ซ้ำ

### 1.2 Batch Import
- สร้าง background job สำหรับกวาดไฟล์
- สร้าง BOOK + ETL_JOB จากไฟล์ที่พบ
- จัดการ error case (ไฟล์เสีย, format ไม่ถูกต้อง, permission ไม่พอ)

---

## 2. ETL & Document Processing
- OCR เอกสาร PDF (รองรับ scan)
- แยกหน้า (pages)
- วิเคราะห์ layout (ตำแหน่ง, ขนาดตัวอักษร, block text)
- ตรวจจับ candidate chapter
- แบ่งเนื้อหาเป็น chunks
- tuning (heuristic / คุณภาพผลลัพธ์)
- สร้าง embeddings
- จัดเก็บ vector ลง Vector Store (pgvector)
- อัปเดตสถานะ ETL job
- รองรับ retry / resume กรณี job ล้ม

---

## 3. AI Question & Answer System
- ออกแบบ prompt หลัก (system + instruction)
- ทำ retrieval จาก Vector Store (เฉพาะ book status = ready)
- ส่ง context + history เข้า LLM
- รองรับ streaming response
- ให้ LLM ตัดสินใจโหมด (Q&A / Summary / Search / Location)
- ส่งผลลัพธ์กลับผู้ใช้แบบ real-time
- ปรับ prompt และ retrieval strategy จากผลใช้งานจริง

---

## 4. Metadata Management
- แก้ไขชื่อหนังสือ / คำอธิบาย / หมวด / tag
- แยก metadata ออกจากไฟล์และ ETL
- รับประกันว่าไม่กระทบการประมวลผล
- แสดงผล metadata ทันทีใน UI

---

## 5. Feedback & Incident Tracking
- บันทึกเฉพาะ feedback เชิงลบ
- เก็บ QNA Incident Log
- วิเคราะห์ปัญหาเพื่อปรับปรุง AI
- ใช้ข้อมูลสำหรับปรับ prompt หรือ retrieval ในอนาคต

---

## 6. Frontend Development
- หน้าจอ upload หนังสือ
- หน้ารายการหนังสือ + สถานะ
- หน้าแก้ไข metadata
- หน้าถาม AI (chat UI)
- แสดงแหล่งอ้างอิง (book / page)
- UX สำหรับ error และสถานะ processing

---

## 7. Infrastructure & Deployment
- Setup Object Storage
- Setup Database (OLTP)
- Setup Vector Store (pgvector)
- ออกแบบรองรับการแยก Vector DB เป็น instance อิสระในอนาคต
- ทำ index / partition สำหรับ vector table
- Setup Worker
- CI/CD pipeline
- Environment separation (dev / staging / prod)
- Monitoring & logging
- วางนโยบาย storage cleanup และ retention
  (ไฟล์ / ข้อมูลอะไรต้องเก็บนานแค่ไหน และอะไรควรถูกลบเมื่อไม่จำเป็นแล้ว)

---

## 8. Testing & Quality Assurance
- Unit test (Backend / Worker)
- Integration test (ETL flow)
- Load test (Q&A / vector search)
- Failure scenario testing (OCR fail, ETL stuck, LLM error)
- User acceptance testing (บรรณารักษ์)

---


# Estimation

| WBS Title | Scope Summary | Est. Man-Day | Role(s) | Dependency | Risk |
|----------|---------------|--------------|---------|------------|------|
| Data Preparation & Ingestion | Storage setup, naming convention, duplicate handling, permission model, idempotent import | 12 | Backend, Infra | - | Medium |
| ETL & Document Processing | OCR, page split, layout analysis, chapter detection, chunking, embedding, ETL control | 57 | Backend, AI | Data Preparation & Ingestion | High |
| AI Question & Answer System | Prompt design, retrieval, LLM integration, streaming response, mode decision | 29 | Backend, AI | ETL & Document Processing | High |
| Metadata Management | Edit book title, description, category, tag without reprocessing | 7 | Backend, Frontend | - | Low |
| Feedback & Incident Tracking | Log negative feedback, incident analysis for future tuning | 5 | Backend, AI | AI Question & Answer System | Low |
| Frontend Development | Upload, book list, metadata edit, chat UI, reference display | 21 | Frontend | AI Question & Answer System | Medium |
| Infrastructure & Deployment | Storage, OLTP DB, pgvector, worker, CI/CD, monitoring, retention policy | 28 | Infra | - | Medium |
| Testing & Quality Assurance | Unit, integration, load, failure, librarian UAT | 22 | Backend, QA | All | High |

---

# Timeline

<table>
  <thead>
    <tr>
      <th>เดือน</th>
      <th>หัวข้องาน (อิง WBS)</th>
      <th>รายละเอียดงานหลัก</th>
      <th>ผลลัพธ์ที่ผู้ใช้ได้ (Deliverables)</th>
      <th>ความเสี่ยงหลัก</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Month 1</td>
      <td>Data Preparation & Ingestion<br>Infrastructure Setup</td>
      <td>
        - ตั้งค่า Object Storage (S3 / MinIO)<br>
        - กำหนด naming convention และ duplicate rule<br>
        - ออกแบบสิทธิ์การเข้าถึงไฟล์<br>
        - สร้าง Batch Import Worker
      </td>
      <td>
        - สามารถนำไฟล์ PDF จำนวนมากเข้า storage ได้<br>
        - ระบบเพิ่มหนังสือเข้าไปในอัตโนมัติจากไฟล์ที่วางไว้<br>
        - สร้าง job สำหรับ ETL
      </td>
      <td>Medium</td>
    </tr>
    <tr>
      <td>Month 2</td>
      <td>ETL & Document Processing</td>
      <td>
        - OCR หนังสือที่สแกนมา<br>
        - แยกหน้า วิเคราะห์ layout<br>
        - แบ่ง chunk และสร้าง embedding<br>
        - ปรับจูนคุณภาพผลลัพธ์ (tuning)
      </td>
      <td>
        - หนังสือถูกประมวลผลจนพร้อมใช้งาน (status = ready)<br>
        - เนื้อหาถูกแปลงเป็นข้อมูลที่ AI ใช้งานได้
      </td>
      <td><b>High (Critical Path)</b></td>
    </tr>
    <tr>
      <td>Month 3</td>
      <td>AI Question & Answer System<br>Metadata Management<br>Frontend Core</td>
      <td>
        - ระบบถาม–ตอบด้วย AI (Q&A / Summary / Search)<br>
        - Streaming chat แบบ real-time<br>
        - แก้ไข metadata หนังสือ<br>
        - หน้าจอใช้งานหลัก
      </td>
      <td>
        - ผู้ใช้ถาม AI เกี่ยวกับหนังสือได้จริง<br>
        - บรรณารักษ์แก้ไขข้อมูลหนังสือได้
      </td>
      <td>Medium</td>
    </tr>
    <tr>
      <td>Month 4</td>
      <td>System Hardening & QA</td>
      <td>
        - บันทึก feedback เชิงลบ<br>
        - Load / Failure testing<br>
        - ปรับ infrastructure และ vector search<br>
        - ทดสอบกับบรรณารักษ์ (UAT)
      </td>
      <td>
        - ระบบพร้อมใช้งานจริง<br>
        - รู้จุดอ่อนเพื่อพัฒนาต่อในอนาคต
      </td>
      <td>Medium</td>
    </tr>
  </tbody>
</table>
