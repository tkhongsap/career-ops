# Mode: batch — ประมวลผลข้อเสนองานจำนวนมาก

**ภาษา:** ผู้ใช้เขียนภาษาไทย → ตอบภาษาไทย | ผู้ใช้เขียนภาษาอังกฤษ → ตอบภาษาอังกฤษ

2 modes การใช้งาน: **conductor --chrome** (นำทาง portals แบบ real-time) หรือ **standalone** (script สำหรับ URLs ที่รวบรวมแล้ว)

## สถาปัตยกรรม

```
Claude Conductor (claude --chrome --dangerously-skip-permissions)
  │
  │  Chrome: นำทาง portals (sessions ที่ login แล้ว)
  │  อ่าน DOM โดยตรง — ผู้ใช้เห็นทุกอย่างแบบ real-time
  │
  ├─ ข้อเสนอที่ 1: อ่าน JD จาก DOM + URL
  │    └─► claude -p worker → report .md + PDF + tracker-line
  │
  ├─ ข้อเสนอที่ 2: คลิกถัดไป, อ่าน JD + URL
  │    └─► claude -p worker → report .md + PDF + tracker-line
  │
  └─ จบ: merge tracker-additions → applications.md + สรุป
```

แต่ละ worker คือ `claude -p` ลูกที่มี context สะอาด 200K tokens conductor ทำหน้าที่แค่ orchestrate

## ไฟล์

```
batch/
  batch-input.tsv               # URLs (โดย conductor หรือ manual)
  batch-state.tsv               # ความคืบหน้า (auto-generated, gitignored)
  batch-runner.sh               # Script orchestrator แบบ standalone
  batch-prompt.md               # Prompt template สำหรับ workers
  logs/                         # Log หนึ่งไฟล์ต่อข้อเสนอ (gitignored)
  tracker-additions/            # บรรทัด tracker (gitignored)
```

## Mode A: Conductor --chrome

1. **อ่าน state**: `batch/batch-state.tsv` → รู้ว่าประมวลผลอะไรแล้ว
2. **นำทาง portal**: Chrome → URL ค้นหา
3. **ดึง URLs**: อ่าน DOM ผลการค้นหา → ดึงรายการ URLs → append ไปยัง `batch-input.tsv`
4. **สำหรับแต่ละ URL ที่รอ**:
   a. Chrome: คลิกข้อเสนอ → อ่านข้อความ JD จาก DOM
   b. บันทึก JD ไปยัง `/tmp/batch-jd-{id}.txt`
   c. คำนวณ REPORT_NUM ถัดไปตามลำดับ
   d. รันผ่าน Bash:
      ```bash
      claude -p --dangerously-skip-permissions \
        --append-system-prompt-file batch/batch-prompt.md \
        "Process this offer. URL: {url}. JD: /tmp/batch-jd-{id}.txt. Report: {num}. ID: {id}"
      ```
   e. อัปเดต `batch-state.tsv` (completed/failed + score + report_num)
   f. Log ไปยัง `logs/{report_num}-{id}.log`
   g. Chrome: กลับไป → ข้อเสนอถัดไป
5. **Pagination**: ถ้าไม่มีข้อเสนออีก → คลิก "Next" → ทำซ้ำ
6. **จบ**: Merge `tracker-additions/` → `applications.md` + สรุป

## Mode B: Script Standalone

```bash
batch/batch-runner.sh [OPTIONS]
```

ตัวเลือก:
- `--dry-run` — แสดงรายการที่รอโดยไม่รัน
- `--retry-failed` — ลองใหม่เฉพาะที่ล้มเหลว
- `--start-from N` — เริ่มจาก ID N
- `--parallel N` — N workers พร้อมกัน
- `--max-retries N` — จำนวนครั้งที่ลองต่อข้อเสนอ (default: 2)

## รูปแบบ batch-state.tsv

```
id	url	status	started_at	completed_at	report_num	score	error	retries
1	https://...	completed	2026-...	2026-...	002	4.2	-	0
2	https://...	failed	2026-...	2026-...	-	-	Error msg	1
3	https://...	pending	-	-	-	-	-	0
```

## การกู้คืน (Resumability)

- ถ้า process ตาย → รัน re-execute → อ่าน `batch-state.tsv` → ข้ามที่เสร็จแล้ว
- Lock file (`batch-runner.pid`) ป้องกันการรันซ้ำ
- แต่ละ worker เป็นอิสระ: ความล้มเหลวที่ข้อเสนอ #47 ไม่กระทบข้ออื่น

## Workers (claude -p)

แต่ละ worker รับ `batch-prompt.md` เป็น system prompt และเป็น self-contained

Worker สร้าง:
1. Report `.md` ใน `reports/`
2. PDF ใน `output/`
3. บรรทัด tracker ใน `batch/tracker-additions/{id}.tsv`
4. JSON ผลลัพธ์ผ่าน stdout

## การจัดการข้อผิดพลาด

| ข้อผิดพลาด | การกู้คืน |
|------------|-----------|
| URL ไม่สามารถเข้าถึง | Worker ล้มเหลว → conductor ทำเครื่องหมาย `failed`, ข้ออื่นต่อ |
| JD อยู่หลัง login | Conductor พยายามอ่าน DOM ถ้าล้มเหลว → `failed` |
| Portal เปลี่ยน layout | Conductor วิเคราะห์ HTML, ปรับตัว |
| Worker crash | Conductor ทำเครื่องหมาย `failed`, ข้ออื่นต่อ Retry ด้วย `--retry-failed` |
| Conductor ตาย | Re-run → อ่าน state → ข้ามที่เสร็จ |
| PDF ล้มเหลว | Report .md ถูกบันทึก PDF ค้างรอ |
