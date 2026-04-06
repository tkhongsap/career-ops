# PRD: Career-Ops Phase 1 — Thai Localization

**Author:** Ava (for Ta Khongsap)
**Date:** April 6, 2026
**Status:** Draft
**Repo:** https://github.com/tkhongsap/career-ops
**Upstream:** https://github.com/santifer/career-ops (MIT License)
**Branch:** `feat/thai-localization`

---

## Objective

Transform Career-Ops from a Spanish/English CLI tool into a **bilingual Thai/English** job search pipeline targeting **Thai workers and professionals**. This is Phase 1 (localization + adaptation). Phase 2 (SaaS web app) follows separately.

## Success Criteria

- [ ] All 14 mode files translated to Thai with English fallback
- [ ] Thai job portal scanner configured (JobThai, JobsDB TH, WorkVenture, LinkedIn TH)
- [ ] Thai CV template with photo, Thai formatting norms, Noto Sans Thai font
- [ ] Profile config adapted for Thai market (฿ salary, LINE ID, Thai fields)
- [ ] States/status labels bilingual (Thai + English aliases)
- [ ] Scoring dimensions re-weighted for Thai job market
- [ ] Example files: Thai sample CV, Thai job evaluation report
- [ ] CLAUDE.md rewritten as bilingual system prompt
- [ ] End-to-end test: paste a JobThai URL → get Thai evaluation + Thai PDF

---

## Context

### Why Thailand?
- 38M+ workforce, growing digital economy
- Zero AI-powered job search tools in Thai language
- Thai CVs have unique requirements (photo, personal details, ฿/month salary)
- Job seekers use JobThai (dominant), JobsDB, WorkVenture, LINE for communication
- Bilingual EN/TH is essential — many roles require English CVs for international companies

### What We're Forking
Career-Ops by Santiago Fernandez — an AI job search pipeline built on Claude Code:
- 14 skill modes (evaluation, CV generation, scanning, batch processing)
- A-F scoring system (10 weighted dimensions)
- PDF CV generator (Puppeteer)
- Portal scanner (Playwright)
- Pipeline tracker + Go dashboard

---

## Scope — Phase 1 Deliverables

### 1. Mode Files Translation (`modes/`)

Translate all 14 mode files from Spanish/English → Thai/English bilingual.

| File | Current | Action |
|------|---------|--------|
| `_shared.md` | EN (archetypes, scoring) | Rewrite archetypes for Thai market + translate |
| `oferta.md` | Spanish (evaluation) | → `evaluate.md` — Thai evaluation flow |
| `ofertas.md` | Spanish (multi-eval) | → `evaluate-multi.md` — Thai |
| `contacto.md` | Spanish (contact) | → `contact.md` — Thai |
| `auto-pipeline.md` | EN | Translate to Thai |
| `apply.md` | EN | Translate + adapt for Thai application norms |
| `batch.md` | EN | Translate |
| `deep.md` | EN | Translate |
| `pdf.md` | EN | Translate + Thai font instructions |
| `pipeline.md` | EN | Translate |
| `project.md` | EN | Translate |
| `scan.md` | EN | Translate + Thai portal instructions |
| `tracker.md` | EN | Translate |
| `training.md` | EN | Translate + Thai interview culture |

**Archetype Rewrite (`_shared.md`):**

Replace current AI/ML archetypes with Thai market roles.

**Source:** JobsDB Thailand top 10 job functions (by registered candidates), JobThai categories, Thai 2026 job market trends.

| # | Archetype (EN) | สายงาน (TH) | What employers buy | Example roles |
|---|----------------|-------------|--------------------|----|  
| 1 | Sales & Business Development | ขาย/พัฒนาธุรกิจ | Revenue growth, client relationships, B2B/B2C | Sales Executive, BD Manager, Account Manager |
| 2 | IT & Software Engineering | ไอที/วิศวกรซอฟต์แวร์ | Build & maintain systems, full-stack, cloud | Developer, Programmer, DevOps, QA |
| 3 | Data & AI | ข้อมูลและ AI | Analytics, ML, data pipelines, BI | Data Analyst, Data Engineer, AI Engineer |
| 4 | Digital Marketing | การตลาดดิจิทัล | Performance marketing, content, SEO/SEM | Performance Marketer, Content Creator, SEO Specialist |
| 5 | Accounting & Finance | บัญชีและการเงิน | Financial reporting, audit, tax, banking | Accountant, Financial Analyst, Auditor |
| 6 | Engineering (non-software) | วิศวกรรม | Manufacturing, civil, electrical, mechanical | Production Engineer, QC Engineer, Project Engineer |
| 7 | Admin, HR & Operations | ธุรการ/ทรัพยากรบุคคล | Recruitment, training, office ops | HR Officer, Admin, Recruiter, Coordinator |
| 8 | Product & Project Management | ผู้จัดการผลิตภัณฑ์/โครงการ | Product discovery, delivery, stakeholders | Product Manager, Project Manager, Scrum Master |
| 9 | Design & Creative | ออกแบบ/สร้างสรรค์ | UX/UI, graphic, branding, video | UX Designer, Graphic Designer, Video Editor |
| 10 | Healthcare & Services | สุขภาพและบริการ | Patient care, hospitality, food service | Nurse, Physical Therapist, Hotel Manager |

**Notes:**
- Top 3 (Sales, IT, Data/AI) are the highest-demand and highest-salary categories in Thailand 2026
- Original Career-Ops had 6 archetypes (all AI/ML focused) — we expand to 10 for Thai market breadth
- Each archetype maps to JobThai and JobsDB filter categories for portal scanning
- Users customize during onboarding (pick 2-3 that match their profile)

### 2. Thai Job Portals (`portals.yml`)

Replace US/EU companies with Thai job sources:

```yaml
title_filter:
  positive:
    - "วิศวกร"          # Engineer
    - "นักพัฒนา"        # Developer
    - "นักวิเคราะห์"     # Analyst
    - "ผู้จัดการ"        # Manager
    - "AI"
    - "Data"
    - "Software"
    - "Developer"
    - "Engineer"
    - "Marketing"
  negative:
    - "Intern"
    - "ฝึกงาน"          # Internship

search_queries:
  - site: "jobthai.com"
    query: "{role} {location}"
  - site: "th.jobsdb.com"
    query: "{role}"
  - site: "workventure.com"
    query: "{role}"
  - site: "linkedin.com/jobs"
    query: "{role} Thailand"

tracked_companies:
  # Thai tech companies
  - name: "Agoda"
    careers_url: "https://careersatagoda.com/jobs/"
    platform: "custom"
    enabled: true
  - name: "LINE Man Wongnai"
    careers_url: "https://lmwn.com/careers"
    platform: "custom"
    enabled: true
  - name: "SCB 10X"
    careers_url: "https://www.scb10x.com/careers"
    platform: "custom"
    enabled: true
  - name: "True Digital"
    careers_url: "https://careers.truedigital.com/"
    platform: "custom"
    enabled: true
  - name: "KBTG"
    careers_url: "https://www.kbtg.tech/en/careers"
    platform: "custom"
    enabled: true
  - name: "Bitkub"
    careers_url: "https://www.bitkub.com/careers"
    platform: "custom"
    enabled: true
  - name: "Ascend Group"
    careers_url: "https://www.ascendgroup.com/careers"
    platform: "custom"
    enabled: true
  - name: "AIS"
    careers_url: "https://www.ais.th/careers/"
    platform: "custom"
    enabled: true
  # International companies with Thailand presence
  - name: "Grab Thailand"
    careers_url: "https://grab.careers/"
    platform: "greenhouse"
    enabled: true
  - name: "Shopee Thailand"
    careers_url: "https://careers.shopee.co.th/"
    platform: "custom"
    enabled: true
```

### 3. Thai CV Template (`templates/cv-template-th.html`)

Create a Thai-specific CV template:

- **Photo placeholder** (top-right, 3.5x4.5cm — Thai standard)
- **Thai font:** Add Noto Sans Thai (`fonts/noto-sans-thai.woff2`)
- **Personal details section** (Thai norm):
  - ชื่อ-นามสกุล (Full name in Thai)
  - Name in English
  - วันเกิด / อายุ (Date of birth / Age)
  - LINE ID
  - เบอร์โทร (Phone)
  - อีเมล (Email)
  - ที่อยู่ (Address)
- **Salary in ฿/month** (not annual USD)
- **Bilingual headers**: Thai primary, English in parentheses
  - ประสบการณ์ทำงาน (Work Experience)
  - การศึกษา (Education)
  - ทักษะ (Skills)
  - โปรเจกต์ (Projects)
- Keep the existing EN template as `cv-template-en.html` for international applications

### 4. Profile Config (`config/profile.example.yml`)

```yaml
candidate:
  full_name_th: "สมชาย ใจดี"
  full_name_en: "Somchai Jaidee"
  email: "somchai@example.com"
  phone: "+66-81-234-5678"
  line_id: "somchai.j"
  location: "กรุงเทพมหานคร"      # Bangkok
  linkedin: "linkedin.com/in/somchai"
  portfolio_url: ""
  github: ""
  photo: "photo.jpg"              # Thai CV standard

target_roles:
  primary:
    - "Senior Software Engineer"
    - "วิศวกรซอฟต์แวร์อาวุโส"
  archetypes:
    - name: "Software Engineer"
      name_th: "วิศวกรซอฟต์แวร์"
      level: "Senior"
      fit: "primary"
    - name: "Data Analyst"
      name_th: "นักวิเคราะห์ข้อมูล"
      level: "Mid"
      fit: "secondary"

compensation:
  target_range: "฿80,000-120,000"   # Per month (Thai standard)
  currency: "THB"
  minimum: "฿60,000"
  period: "monthly"                  # Thai jobs quote monthly
  location_flexibility: "Bangkok preferred, remote possible"

location:
  country: "Thailand"
  city: "Bangkok"
  timezone: "Asia/Bangkok"
  visa_status: "Thai national"
```

### 5. States / Status Labels (`templates/states.yml`)

Add Thai aliases to each state:

```yaml
states:
  - id: evaluated
    label: Evaluated
    label_th: ประเมินแล้ว
    aliases: [evaluada, ประเมินแล้ว]

  - id: applied
    label: Applied
    label_th: สมัครแล้ว
    aliases: [aplicado, enviada, สมัครแล้ว, ส่งแล้ว]

  - id: responded
    label: Responded
    label_th: ได้รับตอบกลับ
    aliases: [respondido, ตอบกลับแล้ว]

  - id: interview
    label: Interview
    label_th: สัมภาษณ์
    aliases: [entrevista, สัมภาษณ์, นัดสัมภาษณ์]

  - id: offer
    label: Offer
    label_th: ได้รับข้อเสนอ
    aliases: [oferta, ได้ออฟเฟอร์]

  - id: rejected
    label: Rejected
    label_th: ไม่ผ่าน
    aliases: [rechazado, ไม่ผ่าน, ถูกปฏิเสธ]

  - id: discarded
    label: Discarded
    label_th: ไม่สนใจ
    aliases: [descartado, ไม่สนใจ, ข้าม]

  - id: skip
    label: SKIP
    label_th: ข้าม
    aliases: [no_aplicar, skip, ข้าม, ไม่สมัคร]
```

### 6. Scoring Re-weighting

Adapt the 10 scoring dimensions for Thai market realities:

| Dimension | Original Weight | Thai Weight | Rationale |
|-----------|:-:|:-:|-----------|
| Role fit | 15% | 15% | Same |
| Tech stack match | 15% | 15% | Same |
| Seniority alignment | 10% | 10% | Same |
| Compensation match | 15% | 20% | **Salary negotiation culture is different in TH — more critical** |
| Remote/location | 10% | 10% | Same |
| Company reputation | 10% | 10% | Same |
| Growth potential | 10% | 5% | Less emphasized in Thai market |
| Culture fit | 5% | 5% | Same |
| Interview likelihood | 5% | 5% | Same |
| Strategic value | 5% | 5% | Same |

### 7. Example Files (`examples/`)

Create Thai examples:
- `examples/cv-example-th.md` — Sample Thai developer CV (bilingual)
- `examples/sample-report-th.md` — Sample evaluation of an Agoda Bangkok role
- `examples/article-digest-example-th.md` — Thai proof points format

### 8. Interview Prep — Thai Culture (`interview-prep/`)

Add `interview-prep/thai-culture.md`:
- การไหว้ (Wai greeting) norms in professional settings
- Hierarchy awareness (พี่/น้อง system)
- Appropriate ครับ/ค่ะ usage
- Common Thai interview questions
- Salary negotiation in Thai context (don't negotiate too aggressively)
- Group interview norms at Thai corporations
- Follow-up etiquette (LINE message vs email)

### 9. CLAUDE.md — Bilingual System Prompt

Rewrite the root `CLAUDE.md` to:
- Detect user language (Thai or English) and respond accordingly
- Default to Thai for Thai portal results
- Generate bilingual CVs when requested
- Use Thai scoring/archetypes from `_shared.md`
- Reference Thai interview culture for prep blocks

### 10. PDF Generator — Thai Font Support

Update `generate-pdf.mjs`:
- Add Noto Sans Thai font files to `fonts/`
- Support `lang: "th"` parameter for Thai CV template
- Handle Thai line-breaking (no spaces between words — needs ICU)

---

## Architecture (unchanged from upstream)

```
User pastes JobThai URL
  → Claude extracts JD (Thai or English)
  → Classifies archetype (Thai market roles)
  → Evaluates A-F (bilingual output)
  → Generates Thai CV PDF (with photo, Thai formatting)
  → Tracks in pipeline
```

No backend changes needed. The AI modes are the product — translation is the work.

---

## File Change Summary

| Action | Files | Estimated LOC |
|--------|:-----:|:------------:|
| Translate | 14 mode files | ~1,200 |
| Create | `portals.yml` (Thai) | ~150 |
| Create | `cv-template-th.html` | ~200 |
| Modify | `config/profile.example.yml` | ~60 |
| Modify | `templates/states.yml` | ~30 |
| Modify | `modes/_shared.md` (archetypes) | ~100 |
| Create | `examples/*-th.md` (3 files) | ~200 |
| Create | `interview-prep/thai-culture.md` | ~100 |
| Rewrite | `CLAUDE.md` | ~250 |
| Modify | `generate-pdf.mjs` + fonts | ~50 |
| **Total** | **~25 files** | **~2,340** |

---

## Execution Plan

### Wave 1 — Core (Builder can do in one session)
1. Rewrite `_shared.md` — Thai archetypes + scoring
2. Translate all 14 mode files (rename Spanish → English filenames)
3. Rewrite `CLAUDE.md` bilingual system prompt

### Wave 2 — Templates & Config
4. Create `portals.yml` with Thai job portals
5. Adapt `config/profile.example.yml` for Thai fields
6. Update `templates/states.yml` with Thai aliases
7. Create `cv-template-th.html` with Thai fonts + photo

### Wave 3 — Examples & Polish
8. Create Thai example files (CV, report, digest)
9. Create `interview-prep/thai-culture.md`
10. Update `generate-pdf.mjs` for Thai font support
11. Download Noto Sans Thai font files

### Wave 4 — Test & Ship
12. End-to-end test: JobThai URL → evaluation → PDF
13. Push to `tkhongsap/career-ops`
14. Write README for Thai version

---

## Out of Scope (Phase 2 — SaaS)
- Web app frontend (Next.js)
- User accounts + auth
- Database (Postgres/Supabase)
- Stripe/PromptPay billing
- Multi-user support
- LINE bot integration
- API layer

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Thai line-breaking in PDFs | Use Noto Sans Thai + CSS `word-break: keep-all` |
| JobThai blocks scraping | Use WebSearch fallback + manual URL paste |
| Thai salary data inaccurate | Use ranges from JobThai/Glassdoor TH, let user override |
| Upstream breaking changes | `upstream` remote set, can cherry-pick updates |

---

## Definition of Done

- [ ] A Thai professional can clone the repo, run `claude`, paste a JobThai link, and get:
  - A Thai-language A-F evaluation report
  - A Thai-formatted PDF CV with photo
  - A pipeline tracker entry
- [ ] All mode commands work in Thai
- [ ] Portal scanner finds jobs from Thai sites
- [ ] README explains setup for Thai users
