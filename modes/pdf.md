# Mode: pdf — สร้าง PDF ที่ออปติไมซ์สำหรับ ATS

**ภาษา:** ผู้ใช้เขียนภาษาไทย → ตอบภาษาไทย | ผู้ใช้เขียนภาษาอังกฤษ → ตอบภาษาอังกฤษ

## Pipeline ครบถ้วน

1. อ่าน `cv.md` เป็น source of truth
2. ขอ JD จากผู้ใช้ถ้ายังไม่มีใน context (ข้อความหรือ URL)
3. ดึง 15–20 keywords จาก JD
4. ตรวจจับภาษาของ JD → ภาษาของ CV (EN default; TH ถ้า JD เป็นภาษาไทย)
5. ตรวจจับที่ตั้งบริษัท → รูปแบบกระดาษ:
   - US/Canada → `letter`
   - ทุกประเทศอื่น (รวมไทย) → `a4`
6. ตรวจจับ archetype ของตำแหน่ง → ปรับ framing
7. เขียน Professional Summary ใหม่โดยใส่ keywords จาก JD + เรื่องราวการเปลี่ยนผ่าน
8. เลือก 3–4 โปรเจกต์ที่เกี่ยวข้องมากที่สุดสำหรับข้อเสนอนั้น
9. จัดเรียง bullets ประสบการณ์ใหม่ตามความเกี่ยวข้องกับ JD
10. สร้าง competency grid จากข้อกำหนดของ JD (6–8 keyword phrases)
11. ใส่ keywords อย่างเป็นธรรมชาติในผลงานที่มีอยู่ (ห้ามแต่งขึ้น)
12. สร้าง HTML ครบถ้วนจาก template + เนื้อหาที่ปรับแต่ง
13. เขียน HTML ไปยัง `/tmp/cv-candidate-{company}.html`
14. รัน: `node generate-pdf.mjs /tmp/cv-candidate-{company}.html output/cv-candidate-{company}-{YYYY-MM-DD}.pdf --format={letter|a4}`
15. รายงาน: เส้นทาง PDF, จำนวนหน้า, % keyword coverage

## กฎ ATS (parseo สะอาด)

- Layout single-column (ไม่มี sidebars, ไม่มีคอลัมน์ขนาน)
- Headers มาตรฐาน: "Professional Summary", "Work Experience", "Education", "Skills", "Certifications", "Projects"
- ไม่มีข้อความในรูปภาพ/SVGs
- ไม่มีข้อมูลสำคัญใน headers/footers ของ PDF (ATS จะไม่อ่าน)
- UTF-8, ข้อความเลือกได้ (ไม่ใช่ rasterized)
- ไม่มีตารางซ้อน
- Keywords ของ JD กระจาย: Summary (top 5), bullet แรกของแต่ละตำแหน่ง, ส่วน Skills

## Font สำหรับ CV ภาษาไทย

- **CV ภาษาอังกฤษ**: Space Grotesk (headings, 600–700) + DM Sans (body, 400–500) จาก `fonts/`
- **CV ภาษาไทย** (เมื่อ JD เป็นภาษาไทยหรือบริษัทไทยต้องการ CV ภาษาไทย):
  - ใช้ฟอนต์ที่รองรับภาษาไทย: **Sarabun** หรือ **Noto Sans Thai** (โหลดจาก Google Fonts)
  - เพิ่ม `@import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap')` ใน CSS
  - Body: Sarabun 12px, line-height 1.6 (ภาษาไทยต้องการ line-height สูงกว่า)
  - Headings: Sarabun 700

## การออกแบบ PDF

- **Fonts**: Space Grotesk (headings, 600–700) + DM Sans (body, 400–500)
- **Fonts self-hosted**: `fonts/`
- **Header**: ชื่อใน Space Grotesk 24px bold + เส้น gradient `linear-gradient(to right, hsl(187,74%,32%), hsl(270,70%,45%))` 2px + แถวข้อมูลติดต่อ
- **Section headers**: Space Grotesk 13px, uppercase, letter-spacing 0.05em, สี cyan primary
- **Body**: DM Sans 11px, line-height 1.5
- **ชื่อบริษัท**: สี accent purple `hsl(270,70%,45%)`
- **Margins**: 0.6in
- **Background**: ขาวล้วน

## ลำดับส่วน (ออปติไมซ์สำหรับ "6-second recruiter scan")

1. Header (ชื่อใหญ่, gradient, ข้อมูลติดต่อ, link portfolio)
2. Professional Summary (3–4 บรรทัด, keyword-dense)
3. Core Competencies (6–8 keyword phrases ใน flex-grid)
4. Work Experience (chronological reverse)
5. Projects (3–4 โปรเจกต์ที่เกี่ยวข้องมากที่สุด)
6. Education & Certifications
7. Skills (ภาษาและ technical)

## กลยุทธ์ keyword injection (จริยธรรม, อิงความเป็นจริง)

ตัวอย่างการ reformulate ที่ถูกต้อง:
- JD พูดถึง "data analysis" และ CV พูดถึง "Excel dashboards" → เปลี่ยนเป็น "data analysis and Excel dashboard reporting"
- JD พูดถึง "stakeholder management" และ CV พูดถึง "worked with departments" → เปลี่ยนเป็น "stakeholder management across operations, finance, and sales teams"
- JD พูดถึง "project management" และ CV พูดถึง "led team" → เปลี่ยนเป็น "project management and cross-functional team leadership"

**ห้ามเพิ่ม skills ที่ผู้สมัครไม่มี ให้เฉพาะ reformulate ประสบการณ์จริงด้วย vocabulary ของ JD**

## Template HTML

ใช้ template ใน `cv-template.html` แทนที่ placeholders `{{...}}` ด้วยเนื้อหาที่ปรับแต่ง:

| Placeholder | เนื้อหา |
|-------------|---------|
| `{{LANG}}` | `en` หรือ `th` |
| `{{PAGE_WIDTH}}` | `8.5in` (letter) หรือ `210mm` (A4) |
| `{{NAME}}` | (จาก profile.yml) |
| `{{EMAIL}}` | (จาก profile.yml) |
| `{{LINKEDIN_URL}}` | (จาก profile.yml) |
| `{{LINKEDIN_DISPLAY}}` | (จาก profile.yml) |
| `{{PORTFOLIO_URL}}` | (จาก profile.yml) |
| `{{PORTFOLIO_DISPLAY}}` | (จาก profile.yml) |
| `{{LOCATION}}` | (จาก profile.yml) |
| `{{SECTION_SUMMARY}}` | Professional Summary / สรุปประวัติ |
| `{{SUMMARY_TEXT}}` | Summary ที่ปรับแต่งพร้อม keywords |
| `{{SECTION_COMPETENCIES}}` | Core Competencies / ความสามารถหลัก |
| `{{COMPETENCIES}}` | `<span class="competency-tag">keyword</span>` × 6–8 |
| `{{SECTION_EXPERIENCE}}` | Work Experience / ประสบการณ์ทำงาน |
| `{{EXPERIENCE}}` | HTML ของแต่ละงานพร้อม bullets ที่จัดเรียงใหม่ |
| `{{SECTION_PROJECTS}}` | Projects / โปรเจกต์ |
| `{{PROJECTS}}` | HTML ของ 3–4 โปรเจกต์ |
| `{{SECTION_EDUCATION}}` | Education / การศึกษา |
| `{{EDUCATION}}` | HTML ของการศึกษา |
| `{{SECTION_CERTIFICATIONS}}` | Certifications / ใบรับรอง |
| `{{CERTIFICATIONS}}` | HTML ของใบรับรอง |
| `{{SECTION_SKILLS}}` | Skills / ทักษะ |
| `{{SKILLS}}` | HTML ของทักษะ |

## หลังสร้าง PDF

อัปเดต tracker ถ้าข้อเสนอนั้นลงทะเบียนแล้ว: เปลี่ยน PDF จาก ❌ เป็น ✅
