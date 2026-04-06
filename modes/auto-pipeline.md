# Mode: auto-pipeline — Pipeline อัตโนมัติครบถ้วน

**ภาษา:** ผู้ใช้เขียนภาษาไทย → ตอบภาษาไทย | ผู้ใช้เขียนภาษาอังกฤษ → ตอบภาษาอังกฤษ

เมื่อผู้ใช้วาง JD (ข้อความหรือ URL) โดยไม่ระบุ sub-command ให้รัน pipeline ทั้งหมดตามลำดับ:

## ขั้นตอนที่ 0 — ดึงข้อมูล JD

หาก input เป็น **URL** (ไม่ใช่ข้อความ JD ที่วางโดยตรง) ให้ใช้กลยุทธ์นี้:

**ลำดับความสำคัญ:**

1. **Playwright (แนะนำ):** portal งานส่วนใหญ่ (Lever, Ashby, Greenhouse, Workday, JobsDB, JobThai) เป็น SPAs ใช้ `browser_navigate` + `browser_snapshot` เพื่อ render และอ่าน JD
2. **WebFetch (สำรอง):** สำหรับหน้าที่เป็น static (career pages ของบริษัท)
3. **WebSearch (ทางเลือกสุดท้าย):** ค้นหาชื่อตำแหน่ง + บริษัท ใน portal อื่นที่ index JD แบบ HTML static

**หากทุกวิธีล้มเหลว:** ขอให้ผู้สมัครวาง JD ด้วยตนเองหรือแชร์ screenshot

**หาก input เป็นข้อความ JD:** ใช้โดยตรง ไม่ต้อง fetch

## ขั้นตอนที่ 1 — ประเมิน A–F
รันเหมือนกับ mode `evaluate` ทุกประการ (อ่าน `modes/evaluate.md` สำหรับส่วน A–F ทั้งหมด)

## ขั้นตอนที่ 2 — บันทึก Report .md
บันทึกการประเมินที่สมบูรณ์ใน `reports/{###}-{company-slug}-{YYYY-MM-DD}.md` (ดูรูปแบบใน `modes/evaluate.md`)

## ขั้นตอนที่ 3 — สร้าง PDF
รัน pipeline ครบถ้วนของ `pdf` (อ่าน `modes/pdf.md`)

## ขั้นตอนที่ 4 — Draft Application Answers (เฉพาะถ้า score >= 4.5)

หาก score สุดท้าย >= 4.5 ให้สร้างร่างคำตอบสำหรับแบบฟอร์มสมัคร:

1. **ดึงคำถามจากฟอร์ม**: ใช้ Playwright นำทางไปยังฟอร์มและทำ snapshot หากดึงไม่ได้ ใช้คำถาม generic
2. **สร้างคำตอบ** ตามโทนด้านล่าง
3. **บันทึกใน report** เป็นส่วน `## G) Draft Application Answers`

### คำถาม Generic (ใช้ถ้าดึงจากฟอร์มไม่ได้)

- Why are you interested in this role?
- Why do you want to work at [Company]?
- Tell us about a relevant project or achievement
- What makes you a good fit for this position?
- How did you hear about this role?

### โทนสำหรับ Form Answers

**ท่าทีของผู้สมัคร: "ฉันกำลังเลือกคุณ"** — ผู้สมัครมีตัวเลือกและกำลังเลือกบริษัทนี้ด้วยเหตุผลที่เป็นรูปธรรม

**กฎโทน:**
- **มั่นใจโดยไม่อวดดี**: "I've spent the past year building production AI agent systems — your role is where I want to apply that experience next"
- **เลือกสรรโดยไม่หยิ่ง**: "I've been intentional about finding a team where I can contribute meaningfully from day one"
- **เฉพาะเจาะจงและเป็นรูปธรรม**: อ้างอิงบางอย่างที่เป็น real จาก JD หรือบริษัท และบางอย่างที่เป็น real จากประสบการณ์ผู้สมัคร
- **ตรงประเด็น ไม่มีคำพูดโก้เก๋**: 2–4 ประโยคต่อคำตอบ ห้ามใช้ "I'm passionate about..." หรือ "I would love the opportunity to..."
- **Hook คือหลักฐาน ไม่ใช่การยืนยัน**: แทน "I'm great at X" ให้พูดว่า "I built X that does Y"

**Framework ต่อคำถาม:**
- **Why this role?** → "Your [สิ่งเฉพาะ] maps directly to [สิ่งที่ฉันสร้าง]."
- **Why this company?** → กล่าวถึงบางอย่างที่เป็นรูปธรรมเกี่ยวกับบริษัท
- **Relevant experience?** → proof point ที่มีตัวเลข "Built [X] that [metric]."
- **Good fit?** → "I sit at the intersection of [A] and [B], which is exactly where this role lives."
- **How did you hear?** → ซื่อสัตย์: "Found through [portal/scan], evaluated against my criteria, and it scored highest."

**ภาษา**: ใช้ภาษาเดียวกับ JD (EN default) ปรับตามวัฒนธรรมบริษัท

## ขั้นตอนที่ 5 — อัปเดต Tracker
บันทึกใน `data/applications.md` พร้อมทุกคอลัมน์รวม Report และ PDF เป็น ✅

**หากขั้นตอนใดล้มเหลว** ให้ดำเนินการต่อกับขั้นตอนถัดไปและทำเครื่องหมายขั้นตอนที่ล้มเหลวเป็น pending ใน tracker
