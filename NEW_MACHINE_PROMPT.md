# New Machine Onboarding Prompt
> Copy ข้อความด้านล่างไปวางใน Claude Code Session แรก บนเครื่องใหม่

---

```
สวัสดีครับ นี่คือเครื่องใหม่ของผม ขอให้คุณทำความเข้าใจ Context ต่อไปนี้ก่อนเริ่มงาน:

## ตัวตนและบทบาท
- ผมทำงานที่ Central Group Enterprise
- ใช้ Claude Code ช่วยงาน Data, Automation, และ Scripting
- ภาษาหลัก: ภาษาไทย (ตาม CLAUDE.md §7)

## Environment ของเครื่องนี้
- OS: Windows 11 Pro
- Shell: PowerShell (primary), Bash ใช้ได้ผ่าน Bash tool
- Python: ใช้ `py` หรือ `python3` เท่านั้น (ไม่ใช้ `python`)
- Output folder: `D:\Users\<username>\output\`
- OneDrive: `D:\Users\<username>\OneDrive - Central Group\`

## Policy & Conventions
- อ่าน `CLAUDE.md` ที่ root ของ working directory เสมอ
- อ่าน `CONVENTIONS.md` สำหรับกฎ Coding & Data ทั้งหมด
- Date format: DD/MM/YYYY, dayfirst=True, Timezone: Asia/Bangkok (UTC+7)
- CSV encoding: utf-8-sig | WDCS files: cp874
- ตัวเลข: format #,##0.## มี comma คั่น
- Output บันทึกใน `/output/` เท่านั้น — ห้ามแก้ source file ต้นฉบับ

## Skills ที่ใช้งานได้ (ใน .claude/commands/)
| Command | หน้าที่ |
|---------|---------|
| `/Automation_WDCS_Fletching` | ดึง ExportDO จาก WDCS + Monitor RS Orders |
| `/check_status_order` | ตรวจ Stage Order จาก Excel Export DO |
| `/study_assistant` | สอนจากเอกสาร Local (Zero Hallucination) |
| `/python101` | ข้อควรรู้ Python บนเครื่องนี้ |

## WDCS System
- URL: http://cmgwdcs3.cmg.co.th
- ต้องใช้ Playwright เท่านั้น (requests POST ให้ HTTP 500)
- File format: TSV .txt, encoding cp874
- Match key: `Web Order` (ExportDO) ↔ เลขที่คำสั่งซื้อ (Shopee Export)

## Automation Scripts
- `Automation\rs_order_monitor.py` — RS Express Order Monitor (loop 2 ชม.)
- `Automation\start_rs_monitor.bat` — launcher

## สิ่งที่ต้องทำก่อนเริ่มงาน (ถ้ายังไม่ได้ทำ)
1. ตรวจสอบว่า `CLAUDE.md` และ `CONVENTIONS.md` อยู่ที่ `D:\Users\<username>\` แล้ว
2. ตรวจสอบว่า folder `output\` มีอยู่แล้ว (ถ้าไม่มีให้สร้าง)
3. ตรวจสอบ Python: `py --version` ควรได้ 3.13+

พร้อมแล้วครับ ช่วยยืนยันว่าอ่าน Context ครบแล้ว และแจ้งว่ามีอะไรที่ต้องตั้งค่าเพิ่มเติมไหม
```
