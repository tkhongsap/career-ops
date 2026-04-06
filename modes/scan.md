# Mode: scan — Portal Scanner (ค้นหาข้อเสนองาน)

**ภาษา:** ผู้ใช้เขียนภาษาไทย → ตอบภาษาไทย | ผู้ใช้เขียนภาษาอังกฤษ → ตอบภาษาอังกฤษ

สแกน portals หางานที่กำหนดค่าไว้ กรองตามความเกี่ยวข้องของชื่อตำแหน่ง และเพิ่มข้อเสนอใหม่ไปยัง pipeline สำหรับการประเมิน

## การรันที่แนะนำ

รันเป็น subagent เพื่อไม่ให้ใช้ context ของ main:

```
Agent(
    subagent_type="general-purpose",
    prompt="[เนื้อหาของไฟล์นี้ + ข้อมูลเฉพาะ]",
    run_in_background=True
)
```

## การกำหนดค่า

อ่าน `portals.yml` ที่มี:
- `search_queries`: รายการ queries WebSearch พร้อม `site:` filters ต่อ portal (การค้นพบในวงกว้าง)
- `tracked_companies`: บริษัทเฉพาะพร้อม `careers_url` สำหรับการนำทางโดยตรง
- `title_filter`: Keywords positive/negative/seniority_boost สำหรับกรองชื่อตำแหน่ง

## กลยุทธ์การค้นพบ (3 ระดับ)

### ระดับ 1 — Playwright โดยตรง (หลัก)

**สำหรับแต่ละบริษัทใน `tracked_companies`:** นำทางไปยัง `careers_url` ด้วย Playwright (`browser_navigate` + `browser_snapshot`) อ่าน job listings ที่มองเห็นทั้งหมด และดึง title + URL จากแต่ละอัน วิธีนี้น่าเชื่อถือที่สุดเพราะ:
- เห็นหน้าแบบ real-time (ไม่ใช่ผลลัพธ์ที่ cache จาก Google)
- ทำงานกับ SPAs (Ashby, Lever, Workday)
- ตรวจจับข้อเสนอใหม่ทันที
- ไม่ขึ้นอยู่กับ Google indexing

**แต่ละบริษัท ต้องมี `careers_url` ใน portals.yml** ถ้าไม่มีให้ค้นหาหนึ่งครั้ง บันทึก และใช้ในการสแกนครั้งต่อไป

### ระดับ 2 — Greenhouse API (เสริม)

สำหรับบริษัทที่ใช้ Greenhouse, API JSON (`boards-api.greenhouse.io/v1/boards/{slug}/jobs`) ให้ข้อมูลที่สะอาด ใช้เป็น complement ด่วนของระดับ 1 — เร็วกว่า Playwright แต่ทำงานเฉพาะกับ Greenhouse

### ระดับ 3 — WebSearch queries (การค้นพบในวงกว้าง)

`search_queries` พร้อม `site:` filters ครอบคลุม portals แบบ cross-section มีประโยชน์สำหรับค้นพบบริษัทใหม่ที่ยังไม่มีใน `tracked_companies` แต่ผลลัพธ์อาจล้าหลัง

**Portal ไทยที่รองรับ:**
- **JobThai**: `site:jobthai.com`
- **JobsDB TH**: `site:th.jobsdb.com`
- **WorkVenture**: `site:workventure.com`
- **LinkedIn TH**: `site:linkedin.com/jobs`

**ลำดับการรัน:**
1. ระดับ 1: Playwright → `tracked_companies` ทั้งหมดที่มี `careers_url`
2. ระดับ 2: API → `tracked_companies` ทั้งหมดที่มี `api:`
3. ระดับ 3: WebSearch → `search_queries` ทั้งหมดที่มี `enabled: true`

ระดับเป็น additive — รันทั้งหมด ผสมผลลัพธ์ และ dedup

## Workflow

1. **อ่าน config**: `portals.yml`
2. **อ่านประวัติ**: `data/scan-history.tsv` → URLs ที่เห็นแล้ว
3. **อ่าน dedup sources**: `data/applications.md` + `data/pipeline.md`

4. **ระดับ 1 — Playwright scan** (ขนานใน batches ละ 3–5):
   สำหรับแต่ละบริษัทใน `tracked_companies` ที่มี `enabled: true` และ `careers_url` ที่กำหนด:
   a. `browser_navigate` ไปยัง `careers_url`
   b. `browser_snapshot` เพื่ออ่าน job listings ทั้งหมด
   c. ถ้าหน้ามี filters/departments ให้นำทางส่วนที่เกี่ยวข้อง
   d. สำหรับแต่ละ job listing ดึง: `{title, url, company}`
   e. ถ้าหน้าแบ่งหน้า ให้นำทางหน้าเพิ่มเติม
   f. สะสมในรายการ candidates
   g. ถ้า `careers_url` ล้มเหลว (404, redirect) ลอง `scan_query` เป็น fallback และจดเพื่ออัปเดต URL

5. **ระดับ 2 — Greenhouse APIs** (ขนาน):
   สำหรับแต่ละบริษัทใน `tracked_companies` ที่มี `api:` และ `enabled: true`:
   a. WebFetch URL ของ API → JSON พร้อมรายการ jobs
   b. สำหรับแต่ละ job ดึง: `{title, url, company}`
   c. สะสมในรายการ candidates (dedup กับระดับ 1)

6. **ระดับ 3 — WebSearch queries** (ขนานถ้าเป็นไปได้):
   สำหรับแต่ละ query ใน `search_queries` ที่มี `enabled: true`:
   a. รัน WebSearch ด้วย `query` ที่กำหนด
   b. จากแต่ละผลลัพธ์ดึง: `{title, url, company}`
   c. สะสมในรายการ candidates (dedup กับระดับ 1+2)

7. **กรองตาม title** โดยใช้ `title_filter` ของ `portals.yml`:
   - อย่างน้อย 1 keyword จาก `positive` ต้องปรากฏในชื่อ (case-insensitive)
   - 0 keywords จาก `negative` ต้องปรากฏ
   - Keywords `seniority_boost` ให้ความสำคัญแต่ไม่บังคับ

8. **Dedup** กับ 3 แหล่ง:
   - `scan-history.tsv` → URL นั้นเห็นแล้ว
   - `applications.md` → บริษัท + ตำแหน่งที่ normalize แล้วประเมินแล้ว
   - `pipeline.md` → URL นั้นมีอยู่ใน pending หรือ processed แล้ว

9. **สำหรับแต่ละข้อเสนอใหม่ที่ผ่านการกรอง**:
   a. เพิ่มไปยัง `pipeline.md` ส่วน "Pending": `- [ ] {url} | {company} | {title}`
   b. บันทึกใน `scan-history.tsv`: `{url}\t{date}\t{query_name}\t{title}\t{company}\tadded`

10. **ข้อเสนอที่กรองตาม title**: บันทึกใน `scan-history.tsv` พร้อมสถานะ `skipped_title`
11. **ข้อเสนอซ้ำ**: บันทึกพร้อมสถานะ `skipped_dup`

## การดึง title และ company จากผลลัพธ์ WebSearch

ผลลัพธ์ WebSearch มาในรูปแบบ: `"ชื่อตำแหน่ง @ บริษัท"` หรือ `"ชื่อตำแหน่ง | บริษัท"` หรือ `"ชื่อตำแหน่ง — บริษัท"`

Pattern การดึงตาม portal:
- **JobThai**: `"วิศวกรซอฟต์แวร์ | บริษัท ABC"` → title: `วิศวกรซอฟต์แวร์`, company: `ABC`
- **JobsDB TH**: `"Data Analyst at XYZ Company"` → title: `Data Analyst`, company: `XYZ Company`
- **Ashby**: `"Senior PM (Remote) @ EverAI"` → title: `Senior PM`, company: `EverAI`
- **Greenhouse**: `"AI Engineer at Anthropic"` → title: `AI Engineer`, company: `Anthropic`

Regex ทั่วไป: `(.+?)(?:\s*[@|—–-]\s*|\s+at\s+|\s*\|\s*)(.+?)$`

## URLs ส่วนตัว

ถ้าพบ URL ที่ไม่สามารถเข้าถึงสาธารณะ:
1. บันทึก JD ไปยัง `jds/{company}-{role-slug}.md`
2. เพิ่มใน pipeline.md เป็น: `- [ ] local:jds/{company}-{role-slug}.md | {company} | {title}`

## Scan History

`data/scan-history.tsv` ติดตาม URLs ทั้งหมดที่เห็น:

```
url	first_seen	portal	title	company	status
https://...	2026-02-10	JobThai — IT	วิศวกรซอฟต์แวร์	บริษัท ก	added
https://...	2026-02-10	JobsDB — Data	Junior Dev	BigCo TH	skipped_title
```

## สรุป output

```
Portal Scan — {YYYY-MM-DD}
━━━━━━━━━━━━━━━━━━━━━━━━━━
Queries ที่รัน: N
ข้อเสนอที่พบ: N รายการ
กรองตาม title: N รายการที่เกี่ยวข้อง
ซ้ำ: N (ประเมินแล้วหรืออยู่ใน pipeline)
ใหม่ที่เพิ่มเข้า pipeline.md: N

  + {company} | {title} | {query_name}
  ...

→ รัน /career-ops pipeline เพื่อประเมินข้อเสนอใหม่
```

## การจัดการ careers_url

แต่ละบริษัทใน `tracked_companies` ต้องมี `careers_url` เพื่อหลีกเลี่ยงการค้นหาทุกครั้ง

**Patterns ที่รู้จักตาม platform:**
- **Ashby:** `https://jobs.ashbyhq.com/{slug}`
- **Greenhouse:** `https://job-boards.greenhouse.io/{slug}`
- **Lever:** `https://jobs.lever.co/{slug}`
- **JobThai:** `https://www.jobthai.com/th/company/{id}/jobs`
- **JobsDB TH:** `https://th.jobsdb.com/th-th/{company-slug}`
- **Custom:** URL หน้า careers ของบริษัทเอง

**ถ้า `careers_url` ไม่มี** สำหรับบริษัท:
1. ลอง pattern ของ platform ที่รู้จัก
2. ถ้าล้มเหลว ทำ WebSearch: `"{company}" careers jobs สมัครงาน`
3. นำทางด้วย Playwright เพื่อยืนยันว่าใช้งานได้
4. **บันทึก URL ที่พบใน portals.yml** สำหรับการสแกนครั้งต่อไป

## การบำรุงรักษา portals.yml

- **บันทึก `careers_url` เสมอ** เมื่อเพิ่มบริษัทใหม่
- เพิ่ม queries ใหม่เมื่อค้นพบ portals หรือตำแหน่งที่น่าสนใจ
- ปิดใช้ queries ด้วย `enabled: false` ถ้าสร้าง noise มากเกินไป
- ปรับ keywords ตามที่ target roles เปลี่ยนไป
- ตรวจสอบ `careers_url` เป็นระยะ — บริษัทเปลี่ยน ATS platform
