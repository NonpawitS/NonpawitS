# Memory — sanonpawit @ D:\Users\sanonpawit

## Skills Library
- **Location:** `D:\Users\sanonpawit\.claude\commands\`
- **Index:** `README.md` ในโฟลเดอร์เดียวกัน
- เรียกใช้ใน Claude Code ด้วย `/skill-name`

| Skill | Command | สรุป |
|-------|---------|------|
| WDCS Fletching | `/Automation_WDCS_Fletching` | ดึง ExportDO จาก WDCS + Monitor RS Orders |
| Check Order Status | `/check_status_order` | ตรวจ Stage Order จาก Excel Export DO |
| Study Assistant | `/study_assistant` | สอนจากเอกสาร Local Zero Hallucination |
| Python 101 | `/python101` | ข้อควรรู้ Python บนเครื่องนี้ |

## Automation Scripts
- `D:\Users\sanonpawit\Automation\rs_order_monitor.py` — RS Express Order Monitor (loop 2 ชม.)
- `D:\Users\sanonpawit\Automation\start_rs_monitor.bat` — launcher
- `D:\Users\sanonpawit\Automation\clear_temp.ps1` — ล้าง %TEMP% ตอน logon (Scheduled Task `ClearTempOnLogon`), log ที่ `OneDrive - Central Group\log\clear_temp_YYYY-MM.log`

## WDCS System (cmgwdcs3.cmg.co.th)
- Login URL: `http://cmgwdcs3.cmg.co.th/Default.aspx`
- ExportDO URL: `http://cmgwdcs3.cmg.co.th/CarTracking/ExportDO.aspx?Mname=Export%20DO&DocType=1`
- **ต้องใช้ Playwright เท่านั้น** (requests POST ให้ HTTP 500 เสมอ)
- File format: TSV `.txt`, encoding `cp874`
- Match key: `Web Order` (ExportDO) ↔ column[1] เลขที่คำสั่งซื้อ (Shopee Export)

## Python บนเครื่องนี้
- ใช้ `python3` หรือ `py` เท่านั้น (ไม่ใช่ `python`)
- Path: `D:\Users\sanonpawit\AppData\Local\Programs\Python\Python313\python.exe`

## Key Directories
- OneDrive: `D:\Users\sanonpawit\OneDrive - Central Group\`
- Downloads: `D:\Users\sanonpawit\Downloads\`
- Output: `D:\Users\sanonpawit\output\`
- Automation scripts: `D:\Users\sanonpawit\Automation\`

## Troubleshooting References
- [ThinkPad L14 Gen 4 keyboard fix](thinkpad_keyboard_fix.md) — built-in keyboard stops responding while Device Manager shows OK; fix via `D:\Users\sanonpawit\output\restart-keyboard.bat`

## Drive C Cleanup
- [Search Index relocated to D: via junction](drive_c_pending_search_index_move.md) — DONE 2026-05-05. Junction approach (registry change failed due to WaaSMedicSvc auto-revert). Re-runnable script at `output\junction_search_index.ps1`.
