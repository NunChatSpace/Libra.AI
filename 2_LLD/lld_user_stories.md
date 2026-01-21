# Low-Level Design — User Stories

## User Story #1 — Upload Book

**As a Librarian,**  
**I want to upload a book without blocking my work,**  
**so that users can later ask questions about that book.**

### Acceptance Criteria
- บรรณารักษ์สามารถอัปโหลดไฟล์ **PDF ได้ 1 ไฟล์ต่อ 1 หนังสือ**
- ระหว่างที่ไฟล์กำลังอัปโหลด บรรณารักษ์สามารถใช้งานหน้าจออื่น ๆ **ภายในระบบ** ได้ และการอัปโหลดยังคงทำงานอยู่เบื้องหลัง
- ระบบแสดง **สถานะการอัปโหลดและการประมวลผล** (เช่น Uploading, Processing, Ready, Failed)
- ระบบสามารถ **ติดตามความคืบหน้าการประมวลผล** จนกว่าจะเสร็จสมบูรณ์หรือเกิดข้อผิดพลาด

### Functional Scope
- อัปโหลดไฟล์ PDF หนึ่งไฟล์
- จัดเก็บ metadata ของหนังสือ
- ประมวลผลเนื้อหาหนังสือแบบ asynchronous
- แสดงสถานะการประมวลผลให้ frontend

### System Responsibilities
**Frontend**
- แสดง UI สำหรับการอัปโหลด
- อัปโหลดไฟล์ผ่าน **presigned URL**
- แสดงสถานะการประมวลผล

**Backend**
- เริ่มต้นกระบวนการอัปโหลด
- รับสัญญาณหรือการแจ้งเตือนว่าการอัปโหลดเสร็จสมบูรณ์
- จัดการ lifecycle ของหนังสือ
- สร้างงานประมวลผลสำหรับ background worker

**Background Worker**
- ประมวลผลเอกสารที่ถูกอัปโหลด
- อัปเดตผลลัพธ์และสถานะการประมวลผล

### Out of Scope / Assumptions
- รองรับเฉพาะไฟล์ PDF เท่านั้น
- ไม่รองรับการเปลี่ยนไฟล์หลังจากอัปโหลดแล้ว
- ไม่รองรับการอัปโหลดหลายไฟล์พร้อมกัน

---

## User Story #2 — View Uploaded Books

**As a Librarian,**  
**I want to view the list of uploaded books and their processing status,**  
**so that I can monitor upload progress and identify completed or failed books.**

### Acceptance Criteria
- บรรณารักษ์สามารถดู **รายการหนังสือที่อัปโหลดแล้ว**
- หนังสือแต่ละเล่มแสดง **สถานะปัจจุบัน** (Uploading, Processing, Ready, Failed)
- มีปุ่มรีเฟรชเพื่ออัพเดทสถานะปัจจุบัน **โดยไม่ต้องรีเฟรชทั้งหน้า**
- หนังสือที่ประมวลผลล้มเหลวสามารถ **แยกแยะได้ชัดเจน** จากเล่มที่สำเร็จ

### Functional Scope
- แสดงรายการหนังสือในรูปแบบ list view
- แสดงสถานะการประมวลผลของแต่ละเล่ม
- อัปเดตหรือรีเฟรชสถานะเมื่อการประมวลผลเปลี่ยนแปลง

### System Responsibilities
**Frontend**
- แสดงรายการหนังสือพร้อมสถานะ
- มีปุ่มรีเฟรชรายการหนังสือ

**Backend**
- ให้ API สำหรับดึงรายการหนังสือและสถานะ
- เปิดเผยสถานะการประมวลผลปัจจุบันของหนังสือแต่ละเล่ม

### Out of Scope / Assumptions
- ไม่รองรับการแก้ไขหรือเปลี่ยนไฟล์ที่อัปโหลดแล้ว
- ไม่รวมความสามารถในการลบหรือสั่งประมวลผลใหม่

---

## User Story #3 — Ask AI About Books

**As a Guest,**  
**I want to ask AI about uploaded books,**  
**so that I can get answers, summaries, search results, and locations based on book content.**

### Acceptance Criteria
- ผู้ใช้สามารถถามคำถามได้ **เฉพาะหนังสือที่มีสถานะ `Ready` เท่านั้น**
- หนังสือที่ยังไม่อยู่ในสถานะ `Ready` จะ **ไม่ถูกนำมาใช้ในการตอบคำถาม**
- ระบบสามารถ:
  - ตอบคำถามจากเนื้อหาหนังสือ (Q&A)
  - สรุปเนื้อหาหนังสือหรือบางส่วน
  - ค้นหาหนังสือจากเนื้อหา
  - แสดงตำแหน่งที่มา (หน้า/ส่วน) ของคำตอบ เมื่อมีข้อมูล
- หากไม่พบเนื้อหาที่เกี่ยวข้อง ระบบจะตอบกลับอย่างเหมาะสม

### Functional Scope
- โต้ตอบกับเนื้อหาหนังสือผ่าน AI
- รองรับหลายโหมดการใช้งาน (Q&A, Summary, Search, Location)
- จัดเส้นทางคำขอไปยังโหมดประมวลผลที่เหมาะสม

### System Responsibilities
**Frontend**
- แสดง UI สำหรับการสนทนา
- แสดงคำตอบ สรุป ผลการค้นหา และตำแหน่งอ้างอิง

**Backend**
- จัดเส้นทางคำขอตามโหมดการใช้งาน
- ทำ retrieval จากเนื้อหาหนังสือ
- ประสานการทำงานกับ AI เพื่อสร้างคำตอบ

**Database / Storage**
- จัดเก็บเนื้อหาหนังสือที่ผ่านการประมวลผลและ metadata

### Out of Scope / Assumptions
- ไม่รองรับคำถามที่อยู่นอกเหนือจากเนื้อหาหนังสือ

---

## User Story #4 — แก้ไขmetadataของหนังสือ

**As a Librarian,**  
**I want to edit book metadata without re-uploading the file,**  
**so that I can correct or update book information easily.**

### Acceptance Criteria
- บรรณารักษ์สามารถแก้ไข **metadataของหนังสือ** ได้ (เช่น ชื่อหนังสือ, คำอธิบาย, แท็ก, หมวดหมู่)
- การแก้ไขmetadata **ไม่จำเป็นต้องอัปโหลดไฟล์ PDF ใหม่**
- การแก้ไขmetadata **ไม่กระตุ้นให้เกิดการประมวลผลเอกสารหรือ ETL ใหม่**
- metadataที่แก้ไขแล้วถูกแสดงผลในระบบทันที

### Functional Scope
- แก้ไขและบันทึกmetadataของหนังสือ
- จัดเก็บการเปลี่ยนแปลงโดยไม่กระทบไฟล์ต้นฉบับหรือสถานะการประมวลผล
- เปิดเผยmetadataที่อัปเดตแล้วให้ส่วนอื่นของระบบ (เช่น หน้าแสดงรายการ, การค้นหา)

### System Responsibilities
**Frontend**
- แสดง UI สำหรับการแก้ไขmetadataของหนังสือ
- ส่งข้อมูลที่แก้ไขไปยัง backend
- แสดงผลmetadataที่อัปเดตแล้วใน UI

**Backend**
- ตรวจสอบความถูกต้องและบันทึกmetadataที่ถูกแก้ไข
- รับประกันว่าการแก้ไขmetadataจะไม่กระทบสถานะการประมวลผลของเอกสาร
- เปิดเผยmetadataที่อัปเดตผ่าน API ที่มีอยู่

### Out of Scope / Assumptions
- ไม่รองรับการแก้ไขหรือเปลี่ยนไฟล์ PDF ที่อัปโหลดแล้ว
- การแก้ไขmetadataไม่ก่อให้เกิดการประมวลผลใหม่ การสร้าง embedding ใหม่

---

## User Story #5 — Background Worker (ETL Processing)

**As a System,**  
**I want to process uploaded book files asynchronously in the background,**  
**so that uploaded books become searchable and usable for Q&A without blocking user operations.**

---

### Acceptance Criteria

- ระบบต้องเริ่มประมวลผล **เฉพาะหนังสือที่มี ETL job สถานะ `pending`**
- Background worker ต้องประมวลผล **ทีละ ETL job ต่อ worker หนึ่งตัว**
- ระบบต้องอัปเดต **สถานะและความคืบหน้า** ของงานประมวลผลอย่างต่อเนื่อง
- เมื่อประมวลผลสำเร็จ หนังสือต้องอยู่ในสถานะ `ready`
- เมื่อประมวลผลล้มเหลว หนังสือต้องอยู่ในสถานะ `failed` และมีข้อมูล error

---

### Functional Scope

- ดึงไฟล์ PDF จาก object storage
- ทำ OCR เพื่อแปลงไฟล์เป็นข้อความต่อหน้า
- แยกข้อความออกเป็น chunks
- สร้าง embeddings สำหรับแต่ละ chunk
- บันทึกผลลัพธ์และสถานะการประมวลผล

---

### System Responsibilities

**Background Worker**
- ดึง ETL job จากฐานข้อมูล
- ประมวลผลเอกสารตามลำดับขั้น (OCR → Chunk → Embedding)
- อัปเดตความคืบหน้าและสถานะของ ETL job
- อัปเดตสถานะของหนังสือ

**Backend**
- จัดเตรียม ETL job ให้ background worker
- เป็นแหล่งข้อมูลสถานะและความคืบหน้าสำหรับระบบอื่น

**Storage / Database**
- จัดเก็บไฟล์ PDF ต้นฉบับ
- จัดเก็บผลลัพธ์การประมวลผล (pages, chunks, embeddings)

---

### Out of Scope / Assumptions

- Background worker ไม่รับ request จากภายนอกโดยตรง
- ไม่รวมกลไก retry อัตโนมัติของ ETL job
- ไม่รวมการ scale worker แบบ dynamic
- ไม่รองรับการประมวลผลไฟล์ที่ไม่ใช่ PDF

---

# User Story #6 — Batch Import Books from Storage Prefix

**As a Librarian,**  
**I want to place many book PDF files into a designated storage location,**  
**so that the system can automatically create and process books in batch without manual uploads.**

---

## Acceptance Criteria
- บรรณารักษ์สามารถอัปโหลดไฟล์ PDF หลายไฟล์ไปยัง **storage prefix ที่กำหนดไว้** (เช่น S3 / MinIO)
- ระบบมี background worker ที่:
  - list ไฟล์จาก storage prefix ตามรอบเวลา
  - ประมวลผลเฉพาะไฟล์ที่ยังไม่เคยถูกนำเข้า
- แต่ละไฟล์ถูกสร้างเป็น BOOK แยกกัน
- ใช้ชื่อไฟล์ (ตัด `.pdf`) เป็นชื่อหนังสือเริ่มต้น
- **ไม่อนุญาตให้มีชื่อไฟล์ซ้ำ**
  - ถ้าพบชื่อซ้ำ ระบบไม่สร้าง BOOK ใหม่
  - บันทึก error และย้ายไฟล์ไปยัง prefix `failed/`
- ความล้มเหลวของไฟล์หนึ่ง **ไม่กระทบไฟล์อื่น**
- บรรณารักษ์สามารถแก้ไข metadata ภายหลังได้ (US#4)

---

## Functional Scope
- กำหนด storage prefix สำหรับ batch import (เช่น `import/`)
- Background worker:
  - list objects จาก storage prefix ตาม schedule
  - ตรวจจับไฟล์ใหม่แบบ idempotent
  - สร้าง BOOK และ ETL_JOB ต่อไฟล์
  - ย้ายไฟล์ไป `processed/` หรือ `failed/` prefix
- หนังสือที่ถูก import แสดงผลในระบบเดียวกับ upload ปกติ

---

## Out of Scope / Assumptions
- ไม่รองรับการ upload ผ่าน browser ใน flow นี้
- ไม่ merge หลายไฟล์เป็นหนังสือเดียว
- ไม่มี concept branch / สาขา
- ไม่รองรับโครงสร้าง prefix ซ้อนหลายระดับ
- สิทธิ์การเข้าถึง storage ถูกจัดการนอกระบบ (admin / ops)
