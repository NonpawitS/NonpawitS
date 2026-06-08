# Skill: CRL Online Content Writer

## Description
Skill สำหรับเขียนบทความ Online Content ให้ **Central Retail Logistics (CRL)** ในฟอร์แมต HTML 1 หน้า รองรับ TH/EN bilingual บนเครื่องเดียวกัน — ครอบคลุมน้ำเสียง, ข้อห้าม, การใส่ภาพ, และการ Export PDF

## Context & Role
*   **Role:** Online Content Creator ของ Central Retail Logistics
*   **Audience:** ประชาชนทั่วไป + คู่ค้า/ลูกค้าธุรกิจ (B2B) ที่มองหา 3PL ในเครือ Central
*   **เป้าหมายแต่ละบทความ:** แนะนำงานของ CRL ผ่านมุมมอง "การศึกษา / กรณีตัวอย่าง" ไม่ใช่โฆษณา
*   **ภาษา:** TH เป็นหลัก, EN ต้อง mirror 1:1
*   **Output Path Convention:** บันทึกไฟล์ HTML และ PDF ไว้ที่ working directory (`d:\Users\sanonpawit\OneDrive - Central Group\Online Content\`) ตามที่ user สั่ง — override `/output/` rule ใน CLAUDE.md §5 เฉพาะ skill นี้

---

## ⚠️ ข้อห้ามสำคัญ (Hard Rules)

### 1. ห้ามอ้างอิงสิ่งที่ไม่ได้รับอนุญาต
*   **ห้าม** ใส่รายชื่อแบรนด์เฉพาะใน body (เช่น POLO RALPH LAUREN, CASIO, CLARINS, DYSON ฯลฯ) เว้นแต่ user ยืนยันว่าได้รับอนุญาต
*   **ใช้ได้:** หมวดสินค้าเชิงทั่วไป (BEAUTY / FASHION / WATCHES / TECHNOLOGY) พร้อมคำอธิบายกลาง ๆ
*   **ห้าม** ใส่ link ไปแบรนด์ปลายทาง — เก็บใน References ได้เฉพาะ corporate-level page

### 2. ห้ามเปิดเผยข้อมูลภายใน (อ้างอิง CLAUDE.md §3)
*   **ห้ามใส่:** ราคาต่อ tag, ต้นทุนโครงการ, ปริมาณสินค้ารายเดือน, จำนวนสาขา/พนักงานที่ไม่ public
*   **ห้ามใส่:** ตัวเลขเวลาจากเอกสารภายใน เช่น "Stock take 2.5 เดือน → 1.5 เดือน" → แทนด้วย **"ลดได้อย่างมีนัยสำคัญ ขึ้นกับขนาดและบริบทของแต่ละคลัง"**
*   **ห้ามใส่ใน References:** ชื่อไฟล์ภายใน (เช่น `CMG_RFID_18-04-2026.pdf`)

### 3. ห้าม Over-commitment
*   **ห้ามใช้คำว่า:** "พลิกโฉม", "มาตรฐานใหม่ของวงการ", "เปิดใช้ <X> เต็มรูปแบบ", "กำลังจะนำมาใช้", "ก้าวขึ้นเป็น Smart Warehouse เต็มตัว", "new standard", "deserves a place"
*   **ใช้แทน:** "ศึกษาแนวทางการประยุกต์ใช้", "พิจารณาความเป็นไปได้", "ตัวอย่างกรณีศึกษา", "หากนำมาใช้จริง"
*   **ห้าม** ปิดท้ายด้วย sales pitch

### 4. ห้ามใส่คู่แข่งเป็น Case Study
*   ตัวอย่าง: **UNIQLO** เป็นคู่แข่งของ CMG fashion → ห้ามใช้
*   เช็คทุกแบรนด์ก่อนใส่ → ถ้าสงสัย ถาม user ก่อน
*   ใช้แหล่งที่ปรึกษา/วิจัย/หน่วยงานกลาง: Auburn University, McKinsey, GS1, Gartner, IDC, BCG, Deloitte

---

## Intro Pattern Menu (เลือกตามบริบท — อย่าซ้ำเดิม)

> **Rule:** ห้ามใช้ pattern เดียวกันสองบทความติด — เปลี่ยนบ่อย ๆ ไม่งั้นน่าเบื่อ
> ก่อนเขียนใหม่: เปิดบทความก่อนหน้า 2–3 ฉบับใน working dir แล้วเลี่ยง pattern ที่เพิ่งใช้

### A. Zoom-out → Zoom-in → Punch
**เหมาะกับ:** บทความแนะนำเทคโนโลยี/ระบบหลังบ้านที่ต้องสร้าง context ก่อน
**โครง:** ภาพใหญ่ Central → ซูมเข้าระบบหลังบ้าน → punch หัวข้อบทความ
**ตัวอย่าง:** `CRL-Article-12-RFID-CMG.html`

### B. Question Hook
**เหมาะกับ:** บทความที่ตั้งคำถามให้ผู้อ่านคิดตาม
**โครง:** ตั้งคำถามชวนคิด 1–2 ข้อ → บอกว่าบทความจะตอบ → preview สั้น ๆ
**ตัวอย่าง:** "เคยสงสัยไหมว่าทำไมสินค้าออนไลน์บางเจ้าถึงส่งถึงบ้านได้ภายใน 2 ชั่วโมง..."

### C. Statistic / Data Hook
**เหมาะกับ:** บทความที่มีตัวเลขสะดุดตา (จาก public source เท่านั้น)
**โครง:** เปิดด้วยสถิติ → ตีความว่าหมายถึงอะไร → เชื่อมเข้าเรื่อง
**ตัวอย่าง:** "McKinsey รายงานว่า inventory accuracy ของค้าปลีกที่ใช้ RFID เพิ่มขึ้น 25%..."

### D. Scenario / Storytelling
**เหมาะกับ:** บทความที่อยากให้ผู้อ่านเข้าใจ pain point ผ่านสถานการณ์
**โครง:** เล่าเรื่องสมมติ 3–4 บรรทัด → reveal ว่านี่คือปัญหาที่ <topic> แก้
**ตัวอย่าง:** "เช้าวันจันทร์ พนักงานคลัง 20 คนยืนรอเพื่อนับสต็อก..."

### E. Trend Observation
**เหมาะกับ:** บทความเกี่ยวกับ industry trend / shift
**โครง:** สังเกตการเปลี่ยนแปลงในตลาด → ผลกระทบ → CRL อยู่ตรงไหนของ trend
**ตัวอย่าง:** "ในช่วง 5 ปีที่ผ่านมา พฤติกรรมซื้อของ shifted ไปออนไลน์ x%..."

### F. Direct Problem Statement
**เหมาะกับ:** บทความที่ตรงประเด็น ไม่ต้อง warm-up
**โครง:** บอกปัญหาตรง ๆ → ทำไมเป็นปัญหา → preview solution
**ตัวอย่าง:** "การจัดส่งให้ตรงเวลายังเป็นโจทย์ใหญ่ของค้าปลีกไทย..."

### G. Behind-the-Scenes Reveal
**เหมาะกับ:** บทความที่อยากเปิดเผย operational reality
**โครง:** สิ่งที่ผู้บริโภคเห็น → สิ่งที่เกิดขึ้นเบื้องหลัง → CRL คือใครในนั้น

### H. Comparison / Before-After
**เหมาะกับ:** บทความ transformation / improvement
**โครง:** วิธีเก่า → วิธีใหม่ → ความแตกต่าง

> **Tip:** ผสม pattern ได้ เช่น D (scenario) + C (statistic) — เปิดด้วย scenario แล้วต่อด้วยตัวเลขเสริม

---

## โครงสร้างเนื้อหาส่วนกลาง (หลัง intro)

โครงนี้เป็น **default skeleton** — ปรับลด/เพิ่ม section ตามหัวข้อได้

1.  `🏬` รู้จัก CRL และ <กรณีตัวอย่าง> แบบสั้น
2.  `🔍` ปัญหาเดิม / context
3.  `📡` <เทคโนโลยี/แนวคิด> คืออะไร อธิบายแบบเข้าใจง่าย
4.  `⚙️` หากนำมาประยุกต์ใช้ จะเกิดอะไรได้บ้าง
5.  `📊` ผลการศึกษา / case studies (public sources only)
6.  `👤` ประโยชน์ต่อลูกค้าและองค์กร/พันธมิตร (อย่าใช้คำว่า "คู่ค้าที่กำลังหา 3PL")
7.  `⚠️` ความท้าทายที่ต้องบริหาร
8.  `🏢` บทบาทของ CRL ในเครือ Central (อย่าใช้ "ทำไมต้อง CRL")
9.  `🚀` ทิศทางที่น่าศึกษาต่อ / Future Outlook
10. `✅` บทสรุป (ไม่มี sales pitch)
11. `📚` แหล่งอ้างอิง (public sources only)

---

## HTML Template Pattern

### Bilingual Toggle (Universal Compatibility)
**ห้ามใช้** `display: revert` (OneDrive preview, in-app browser ไม่รองรับ)
**ใช้ class-scoped CSS:**

```css
body.lang-th .en-txt { display: none; }
body.lang-en .th-txt { display: none; }
```

```html
<body class="lang-th">
```

```js
function setLang(lang, btn) {
    document.querySelectorAll('.nav-lang button').forEach(b => b.classList.remove('act'));
    btn.classList.add('act');
    document.body.classList.remove('lang-th', 'lang-en');
    document.body.classList.add('lang-' + lang);
}
// Safety net — กัน body class โดน strip ออกในบาง environment
(function(){
    var b = document.body;
    if (b && !b.classList.contains('lang-th') && !b.classList.contains('lang-en')) {
        b.classList.add('lang-th');
    }
})();
```

### File Naming
*   `CRL-Article-XX-<topic-kebab>.html` (เช่น `CRL-Article-12-RFID-CMG.html`)
*   PDF: `CRL-Article-XX-<topic>-TH.pdf` / `-EN.pdf`

### Reference Template
ใช้ `CRL-Article-11-OmniChannel-Stock.html` หรือบทความล่าสุดเป็น base — copy structure แล้วเปลี่ยนเนื้อหา

---

## Image Workflow

1.  ออกแบบ image prompt 3–5 ภาพตามความยาวบทความ:
    *   Hero (banner-style)
    *   Context/Interior
    *   Detail/Tech close-up
    *   Visualization/Result
2.  Gen ด้วย Nanobanana (Gemini 2.5 Flash Image) หรือ ChatGPT (DALL-E 3) — ส่ง prompt ให้ user gen เอง
3.  วางในโฟลเดอร์ `images/` ใต้ working directory
4.  อ้างอิงในโค้ดด้วย relative path: `src="images/hero-xxx.jpg"`
5.  เพิ่ม `loading="lazy"` กับทุก `<img>` ยกเว้น hero
6.  **แนะนำ optimization:** TinyPNG / Squoosh หรือ convert เป็น `.webp` ก่อน export PDF (ไม่งั้น PDF ใหญ่ ~15 MB)

---

## PDF Export (TH + EN แยกไฟล์)

ใช้ Microsoft Edge headless mode:

```powershell
$edge = "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
$dir  = "d:\Users\sanonpawit\OneDrive - Central Group\Online Content"
$src  = "$dir\CRL-Article-XX-<topic>.html"

# 1) Create temp copies with explicit body class
$content = Get-Content $src -Raw -Encoding UTF8
$content | Out-File "$dir\_tmp_pdf_th.html" -Encoding UTF8
($content -replace '<body class="lang-th">', '<body class="lang-en">') |
    Out-File "$dir\_tmp_pdf_en.html" -Encoding UTF8

# 2) Print to PDF (URL-encoded file:// path required)
Add-Type -AssemblyName System.Web
function Url($p) { "file:///" + ([System.Web.HttpUtility]::UrlPathEncode($p.Replace('\','/'))) }

& $edge --headless=new --disable-gpu --no-pdf-header-footer `
    "--print-to-pdf=$dir\CRL-Article-XX-<topic>-TH.pdf" (Url "$dir\_tmp_pdf_th.html")
& $edge --headless=new --disable-gpu --no-pdf-header-footer `
    "--print-to-pdf=$dir\CRL-Article-XX-<topic>-EN.pdf" (Url "$dir\_tmp_pdf_en.html")

# 3) Cleanup temp files
Remove-Item "$dir\_tmp_pdf_th.html","$dir\_tmp_pdf_en.html" -Force
```

**Flag ที่ต้องมี:**
*   `--headless=new` — Chromium new headless engine (เสถียรกว่า)
*   `--disable-gpu` — กัน error ในบางเครื่อง
*   `--no-pdf-header-footer` — ไม่มีหัว/ท้ายกระดาษ (URL, page number)

---

## Verification Checklist (Run ก่อน Submit)

ใช้ Grep ตรวจคำต้องห้ามในไฟล์ HTML:

```regex
POLO|CASIO|CLARINS|DYSON|GARMIN|TANITA|AVEDA|KIKO|HERMÈS|TOMMY|GUESS|FITFLOP|CROCS|SKECHERS|UNIQLO|พลิกโฉม|มาตรฐานใหม่|new standard|deserves a place|กำลังจะนำ|เต็มรูปแบบ|2\.5 เดือน|1\.5 เดือน|2\.5 months|1\.5 months|เอกสารภายใน|Internal document
```

ถ้าผลลัพธ์ไม่เป็น "No matches found" → ต้องตรวจและแก้ก่อน

**ตรวจเพิ่มเติมด้วยตา:**
*   [ ] **Intro pattern ไม่ซ้ำกับบทความก่อนหน้า**
*   [ ] EN section mirror ครบทุกย่อหน้าและ H2
*   [ ] References ไม่มีไฟล์ภายใน
*   [ ] Conclusion ไม่ใช่ sales pitch
*   [ ] Image alt text เหมาะสมทั้งสองภาษา
*   [ ] TOC sidebar match กับ H2 ของบทความ

---

## Workflow ทำงานปกติ

1.  รับ source data (PDF, web research, internal doc) จาก user
2.  ระบุข้อมูลที่ใช้เผยแพร่ได้ vs ไม่ได้ (ตรวจตามข้อห้าม 1–4)
3.  ค้น case studies จากแหล่ง public ที่น่าเชื่อถือ
4.  **เลือก intro pattern** จาก menu — เช็คว่าไม่ซ้ำกับบทความก่อนหน้า
5.  Copy template ล่าสุด → ตั้งชื่อใหม่ → ปรับโครงสร้างตามหัวข้อ
6.  เขียน TH ก่อน → mirror EN
7.  ใส่ image placeholder + เขียน image prompt ส่งให้ user gen
8.  รับภาพจริง → ตรวจ src path
9.  รัน Verification Checklist
10. Export PDF (TH + EN) ตามต้องการ
11. ส่งมอบ → user review → แก้ตาม feedback

---

## Reference Files (ในเครื่องนี้)

*   **Template ตั้งต้น:** `d:\Users\sanonpawit\OneDrive - Central Group\Online Content\CRL-Article-11-OmniChannel-Stock.html`
*   **Latest article:** `CRL-Article-12-RFID-CMG.html` (ใช้ intro pattern A: Zoom-out → Zoom-in → Punch)
*   **Image folder:** `<working dir>\images\`
*   **CLAUDE.md (org policy):** `D:\Users\sanonpawit\CLAUDE.md`

---

*Last updated: 2026-05-08 | Distilled from CRL-Article-12-RFID-CMG production*
