# Mode: pipeline — Inbox ของ URLs (Second Brain)

**ภาษา:** ผู้ใช้เขียนภาษาไทย → ตอบภาษาไทย | ผู้ใช้เขียนภาษาอังกฤษ → ตอบภาษาอังกฤษ

ประมวลผล URLs ข้อเสนองานที่สะสมใน `data/pipeline.md` ผู้ใช้เพิ่ม URLs เมื่อใดก็ได้ แล้วรัน `/career-ops pipeline` เพื่อประมวลผลทั้งหมด

## Workflow

1. **อ่าน** `data/pipeline.md` → หา items `- [ ]` ในส่วน "Pending"
2. **สำหรับแต่ละ URL ที่รอ**:
   a. คำนวณ `REPORT_NUM` ถัดไปตามลำดับ (อ่าน `reports/`, เอาหมายเลขสูงสุด + 1)
   b. **ดึง JD** โดยใช้ Playwright (browser_navigate + browser_snapshot) → WebFetch → WebSearch
   c. ถ้า URL ไม่สามารถเข้าถึง → ทำเครื่องหมาย `- [!]` พร้อมหมายเหตุและดำเนินการต่อ
   d. **รัน auto-pipeline ครบถ้วน**: ประเมิน A–F → Report .md → PDF (ถ้า score >= 3.0) → Tracker
   e. **ย้ายจาก "Pending" ไปยัง "Processed"**: `- [x] #NNN | URL | บริษัท | ตำแหน่ง | Score/5 | PDF ✅/❌`
3. **ถ้ามี 3+ URLs ที่รอ** ให้เปิด agents แบบขนาน (Agent tool กับ `run_in_background`) เพื่อเพิ่มความเร็ว
4. **เมื่อเสร็จ** แสดงตารางสรุป:

```
| # | บริษัท | ตำแหน่ง | Score | PDF | การแนะนำ |
```

## รูปแบบ pipeline.md

```markdown
## Pending
- [ ] https://jobs.example.com/posting/123
- [ ] https://www.jobthai.com/th/job/123456 | Acme TH | Senior PM
- [!] https://private.url/job — Error: login required

## Processed
- [x] #143 | https://jobs.example.com/posting/789 | บริษัท ก | AI PM | 4.2/5 | PDF ✅
- [x] #144 | https://www.jobsdb.com/th/en/job/123 | BigCo | SA | 2.1/5 | PDF ❌
```

## การตรวจจับ JD จาก URL อัจฉริยะ

1. **Playwright (แนะนำ):** `browser_navigate` + `browser_snapshot` ทำงานกับ SPAs ทั้งหมด
2. **WebFetch (สำรอง):** สำหรับหน้า static หรือเมื่อ Playwright ไม่พร้อมใช้งาน
3. **WebSearch (ทางเลือกสุดท้าย):** ค้นหาใน portals รองที่ index JD

**กรณีพิเศษ:**
- **JobThai/JobsDB**: ใช้ Playwright เพราะเป็น dynamic pages — ถ้า login required ทำเครื่องหมาย `[!]` และขอให้ผู้ใช้วางข้อความ
- **LinkedIn**: มักต้องการ login → ทำเครื่องหมาย `[!]` และขอให้ผู้ใช้วางข้อความ
- **PDF**: ถ้า URL ชี้ไปยัง PDF ให้อ่านโดยตรงด้วย Read tool
- **`local:` prefix**: อ่านไฟล์ local ตัวอย่าง: `local:jds/jobsdb-pm-ai.md` → อ่าน `jds/jobsdb-pm-ai.md`

## การนับหมายเลขอัตโนมัติ

1. แสดงรายการไฟล์ทั้งหมดใน `reports/`
2. ดึงหมายเลขจาก prefix (เช่น `142-company...` → 142)
3. หมายเลขใหม่ = ค่าสูงสุดที่พบ + 1

## การตรวจสอบ sync

ก่อนประมวลผล URL ใด ๆ ให้ตรวจสอบ sync:
```bash
node cv-sync-check.mjs
```
ถ้ามีการไม่ sync ให้แจ้งเตือนผู้ใช้ก่อนดำเนินการต่อ
