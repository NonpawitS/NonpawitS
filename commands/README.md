# Claude Code Skills Library
> `D:\Users\sanonpawit\.claude\commands\`
> ใช้พิมพ์ `/ชื่อ-skill` ใน Claude Code เพื่อเรียกใช้

---

## Skills ทั้งหมด

| Skill | File | คำสั่ง | ความสามารถ |
|-------|------|--------|-----------|
| **WDCS Fletching** | `Automation_WDCS_Fletching.md` | `/Automation_WDCS_Fletching` | ดึง ExportDO จาก cmgwdcs3 อัตโนมัติ + Monitor RS Order + Windows Notification |
| **Check Order Status** | `check_status_order.md` | `/check_status_order` | ตรวจสอบ Stage ของ Order จากไฟล์ Excel Export DO (Picking → Load pipeline) |
| **Study Assistant** | `study_assistant.md` | `/study_assistant` | สอนจากเอกสาร Local แบบ Zero Hallucination (PDF/PPTX/MD) |
| **Python 101** | `python101.md` | `/python101` | ข้อควรรู้การรัน Python บนเครื่องนี้ (`python3` / `py` เท่านั้น) |
| **Watcher Notify** | `watcher_notify.md` | `/watcher_notify` | Pattern Windows toast notification สำหรับ watcher (Python + PowerShell) ผ่าน winotify |
| **CRL Online Content** | `crl_online_content.md` | `/crl_online_content` | เขียนบทความ CRL ฟอร์แมต HTML 1 หน้า TH/EN bilingual + Export PDF (โครงสร้าง, ข้อห้าม, intro pattern menu) |

---

## วิธีสร้าง Skill ใหม่

1. สร้างไฟล์ `.md` ใน folder นี้
2. ตั้งชื่อเป็น `skill_name.md` (ใช้เป็น command `/skill_name`)
3. Format: เริ่มต้นด้วย `# Skill: ...` ตามด้วยรายละเอียด

---

## Automation Scripts

| Script | ตำแหน่ง | วิธีรัน |
|--------|---------|--------|
| RS Order Monitor | `D:\Users\sanonpawit\Automation\rs_order_monitor.py` | `start_rs_monitor.bat` |

---

*อัปเดตล่าสุด: 2026-03-03*
