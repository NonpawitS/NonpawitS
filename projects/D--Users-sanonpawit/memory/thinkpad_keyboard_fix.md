---
name: ThinkPad L14 Gen 4 keyboard stops responding
description: Known recurring issue — keyboard shows "OK" in Device Manager but typing produces nothing. Fix by restarting the keyboard PnP driver.
type: reference
originSessionId: 0946435f-9b19-4841-a069-eaf1c69eb1b9
---
# อาการ
คีย์บอร์ดในตัว ThinkPad L14 Gen 4 (Model 21H2S0SM00) พิมพ์ไม่ออกทั้งตัว แต่ Device Manager แสดง Status = OK ทุกตัว และไม่มี scancode remap / filter driver แปลก / process hook

# วิธีแก้ที่ใช้ได้ผล (Apr 2026)
รัน `D:\Users\sanonpawit\output\restart-keyboard.bat` → คลิก Yes ที่ UAC → คีย์บอร์ดกลับมาทันที ไม่ต้อง reboot

สคริปต์ทำ:
```
pnputil /restart-device "ACPI\LEN0071\4&16F63C76&0"            # built-in PS/2
pnputil /restart-device "HID\INTC816&Col01\3&2361649&0&0000"   # Intel HID
```

# Device Instance IDs (เครื่องนี้)
- Built-in keyboard: `ACPI\LEN0071\4&16F63C76&0`
- Intel HID (function keys): `HID\INTC816&COL01\3&2361649&0&0000`
- Phantom (ignore): `HID\VID_0581&PID_011A&MI_00\...` — CM_PROB_PHANTOM

# สาเหตุที่เดาไว้
1. Sleep/resume cycle ทำ driver state ค้าง
2. AnyDesk RIS (IT enterprise tool) hook ค้างหลัง remote session
3. FilterKeys/StickyKeys ถูกเผลอเปิด (Shift ค้าง 8 วิ / Shift 5 ครั้ง) — ครั้งนี้เจอเปิดอยู่ด้วย flags 126/510/62 แก้กลับเป็น 122/506/58

# Troubleshooting checklist ที่ควรรัน (ลำดับ)
1. เช็ค FilterKeys/StickyKeys flags → ปิดถ้าเปิด
2. เช็ค scancode map → ไม่มี (ปกติ)
3. เช็ค upper filter drivers (kbdclass เท่านั้น = ปกติ)
4. เช็ค hook process (autohotkey/powertoys/logi) → ไม่มี
5. เช็ค AnyDesk active session → ไม่มี incoming จริง
6. รัน restart-keyboard.bat ← วิธีนี้หายในครั้งนี้

# หมายเหตุ AnyDesk
เครื่องมี AnyDesk 2 ตัว — ตัวที่ชื่อ `AnyDesk-ad_42f4937a_msi` คือ **"AnyDesk Client by RIS Service"** = IT ของ Central Group ห้ามปิด
