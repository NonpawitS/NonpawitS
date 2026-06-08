# Skill: Automation_WDCS_Fletching

> ดึงข้อมูล ExportDO จากระบบ WDCS (cmgwdcs3.cmg.co.th) อัตโนมัติ
> และ Monitor สถานะ Order เปรียบเทียบกับไฟล์ที่ต้องการ พร้อมแจ้งเตือน Windows Notification

---

## Overview

| รายการ | รายละเอียด |
|--------|-----------|
| **ระบบ** | CMG WDCS — CarTracking Module |
| **URL** | `http://cmgwdcs3.cmg.co.th/CarTracking/ExportDO.aspx` |
| **Login URL** | `http://cmgwdcs3.cmg.co.th/Default.aspx` |
| **Output Format** | Tab-separated Text (`.txt`), encoding `cp874` |
| **Tool ที่ใช้** | Playwright (Chromium headless) + pandas + winotify |
| **Script** | `D:\Users\sanonpawit\Automation\rs_order_monitor.py` |

---

## ⚠️ บทเรียนสำคัญ — ทำไมต้องใช้ Playwright

การ POST form ด้วย `requests` ให้ HTTP 500 เสมอ แม้จะ login ผ่านแล้วก็ตาม
สาเหตุ: Server มี custom error handling + อาจมี JavaScript validation ที่รันก่อน submit
**→ ต้องใช้ Playwright (headless browser) เท่านั้น**

```
requests POST   →  HTTP 500 (ทุก DocType, ทุก date format)
Playwright      →  ดาวน์โหลดสำเร็จ (ExportDO{timestamp}{userID}.txt)
```

---

## Login Flow

```
GET  http://cmgwdcs3.cmg.co.th/Default.aspx
     ↓  form fields ที่ต้องกรอก:
     ctl00$ContentPlaceHolder1$UCWebPickLogin1$txtUserName
     ctl00$ContentPlaceHolder1$UCWebPickLogin1$txtPassword
     ctl00$ContentPlaceHolder1$UCWebPickLogin1$btnLogin  = "เข้าสู่ระบบ"

POST Default.aspx  →  Redirect ไป /PickingWebSite/main.aspx  (= Login สำเร็จ)
GET  /CarTracking/ExportDO.aspx?Mname=Export%20DO&DocType=1
```

---

## ExportDO Form Fields

| Field Name | ความหมาย | ตัวอย่างค่า |
|-----------|---------|------------|
| `txtDODateF` | DOCreate Date (From) | `02/03/2026` (dd/MM/yyyy) |
| `txtDODateT` | DOCreate Date (To) | `03/03/2026` |
| `txtWhsLoc` | Warehouse | `BA01` (บางนา กม.20) หรือ `BA03` |
| `ddlPlantlist` | Plant | `1005`, `1008`, `` (ว่าง = ทุก Plant) |
| `ddlBrandOfArticle` | Brand In Article | `RS`, `BC`, `JB`, `` (ว่าง = ทุก Brand) |
| `btnExport` | ปุ่ม Export | `Export File` |

> **Date range:** ควร set ให้ครอบคลุม DOCreate Date ของทุก order ใน monitoring file
> ดึง `Order Create Date` จาก RS file (Excel serial → แปลงด้วย `pd.Timestamp('1899-12-30') + pd.Timedelta(days=N)`)

---

## ExportDO File Format

ไฟล์ที่ได้เป็น `.txt` tab-separated, encoding `cp874`
ชื่อไฟล์: `ExportDO{YYYYMMDDHHmm}{UserID}.txt`

```python
df = pd.read_csv(filepath, sep='\t', encoding='cp874', low_memory=False)
```

### Columns สำคัญ

| Column | ความหมาย |
|--------|---------|
| `Web Order` | Shopee/TikTok/Lazada Order ID (ใช้ match กับ RS file) |
| `Brand In Article` | ขนส่ง: `RS`=RS Express, `BC`=Shopee Xpress, `JB`=J&T |
| `Delivery` | Delivery Order number (SAP) |
| `Transport_No` | เลขทะเบียนรถ / Transport reference |
| `DOCreate-date` | วันที่สร้าง DO |
| `DOEndPick-date` | วันที่จัด pick เสร็จ |
| `InvoiceDate-date` | วันที่ Invoice |
| `PackingCreate-date` | วันที่ Pack เสร็จ |
| `TransportDate-date` | วันที่ส่งให้ขนส่ง |
| `Ship Date-date` | วันที่ออกจาก WH (ขั้นตอนสุดท้าย) |

---

## Status Logic (Order Pipeline)

```
DOCreate  →  EndPick  →  Invoice  →  Packing  →  Transport  →  Ship ✓
```

```python
def get_status(row):
    if pd.notna(row['Ship Date-date']) and pd.notna(row['Transport_No']):
        return 'shipped'           # ✅ ส่งแล้ว (ครบ)
    elif pd.notna(row['Ship Date-date']):
        return 'shipped_no_tn'    # ⚠️ ส่งแล้ว แต่ยังไม่มี Transport No
    elif pd.notna(row['TransportDate-date']):
        return 'transport_assigned'
    elif pd.notna(row['PackingCreate-date']):
        return 'packed'
    elif pd.notna(row['InvoiceDate-date']):
        return 'invoiced'         # 🔴 ยังไม่ส่ง
    else:
        return 'pending'
```

---

## Matching กับ Shopee Export File

ไฟล์ RS/Shopee (เช่น `RS_EXpress_delivery_03032026.xlsx`) มี:
- **Column [1]** = เลขที่คำสั่งซื้อ (Shopee Order ID เช่น `2603024H53PAGU`)
- ตรง match กับ `Web Order` column ใน ExportDO

```python
df_rs  = pd.read_excel(rs_file, sheet_name='Sheet1')
order_id_col = df_rs.columns[1]          # เลขที่คำสั่งซื้อ
rs_ids = df_rs[order_id_col].dropna().unique()

matched = df_do[df_do['Web Order'].isin(rs_ids)]
```

### รูปแบบ Shopee Order ID
- `260302XXXXXXXX` = ปี26 เดือน03 วัน02 + random
- `260303XXXXXXXX` = ปี26 เดือน03 วัน03

---

## Auto-Monitor Script

```
D:\Users\sanonpawit\Automation\
├── rs_order_monitor.py     ← main script (loop ทุก 2 ชั่วโมง)
└── start_rs_monitor.bat    ← double-click เพื่อเปิด monitor window
```

### รันด้วยตนเอง
```bat
# เปิด window ใหม่ (แนะนำ)
start_rs_monitor.bat

# หรือสั่งตรงจาก terminal
python D:\Users\sanonpawit\Automation\rs_order_monitor.py
```

### Windows Notification (winotify)
```python
from winotify import Notification, audio

toast = Notification(app_id="RS Order Monitor", title=title, msg=msg, duration="long")
toast.set_audio(audio.Default, loop=False)
toast.show()
```

---

## Packages ที่ต้องติดตั้ง

```bash
pip install playwright winotify beautifulsoup4 pandas openpyxl
python -m playwright install chromium
```

---

## การขยาย Skill นี้ (Roadmap)

| Feature | แนวทาง |
|---------|--------|
| **Monitor หลายไฟล์** | รับ path ของ Shopee/Lazada/TikTok export หลายไฟล์พร้อมกัน |
| **Alert เมื่อ Order Stuck** | คำนวณเวลาค้างใน stage แล้ว alert ถ้าเกิน SLA |
| **Line Notify / Teams** | เพิ่ม webhook นอกจาก Windows Notification |
| **Dashboard** | Export เป็น Excel พร้อม conditional formatting ตาม status |
| **ใช้กับ Brand อื่น** | เปลี่ยน `ddlBrandOfArticle` เป็น `BC`, `JB` ฯลฯ |
| **Multi-WH** | เปลี่ยน `txtWhsLoc` เป็น `BA03` หรือ loop ทั้งสอง WH |

---

## Troubleshooting

| ปัญหา | สาเหตุ | วิธีแก้ |
|------|--------|--------|
| Download ไม่ได้ / แค่ 17 orders | Date filter แคบเกินไป | ขยาย `txtDODateF` ให้ครอบ DOCreate date ของทุก order |
| Script ค้างที่ Playwright | Network timeout | เพิ่ม `timeout=60000` ใน `expect_download` |
| ไม่มี Toast notification | winotify ไม่ได้ install | `pip install winotify` |
| Session หมดอายุ | ปล่อยค้างนานเกิน | Script login ใหม่ทุกรอบอยู่แล้ว (ไม่ reuse session) |

---

*Last updated: 2026-03-03*
*Created from: session ดึงข้อมูล ExportDO + Monitor RS Express orders*
