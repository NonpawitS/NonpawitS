# Coding & Data Conventions
> ข้อกำหนดพื้นฐานสำหรับทุก Script, Skill, และ Output บนเครื่องนี้
> Last updated: 2026-03-14

---

## 1. Date & Time

### Format มาตรฐาน

| บริบท | Format | ตัวอย่าง |
|-------|--------|---------|
| แสดงผล (วันที่) | `DD/MM/YYYY` | `14/03/2026` |
| แสดงผล (วันที่ + เวลา) | `DD/MM/YYYY HH:mm` | `14/03/2026 09:30` |
| ชื่อไฟล์ | `YYYYMMDD` | `export_20260314.xlsx` |
| ชื่อไฟล์ + เวลา | `YYYYMMDD_HHmm` | `export_20260314_0930.xlsx` |
| ISO (API / log) | `YYYY-MM-DDTHH:mm:ss+07:00` | `2026-03-14T09:30:00+07:00` |

### กฎหลัก
- **dayfirst = True เสมอ** — วัน/เดือน/ปี ไม่ใช่ เดือน/วัน/ปี
- **Timezone: Asia/Bangkok (UTC+7)** — ทุก datetime ต้องมี timezone aware
- ห้าม naive datetime (ไม่มี timezone) ในระบบ production

### Python — datetime

```python
from datetime import datetime
from zoneinfo import ZoneInfo          # Python 3.9+

TZ_BKK = ZoneInfo("Asia/Bangkok")

# สร้าง datetime ปัจจุบัน (timezone-aware)
now = datetime.now(tz=TZ_BKK)

# แสดงผล
now.strftime("%d/%m/%Y %H:%M")        # → "14/03/2026 09:30"
now.strftime("%d/%m/%Y")              # → "14/03/2026"

# แปลง string → datetime (dayfirst=True)
from dateutil.parser import parse
dt = parse("14/03/2026", dayfirst=True).replace(tzinfo=TZ_BKK)
```

### Python — pandas

```python
import pandas as pd

TZ_BKK = "Asia/Bangkok"

# อ่านคอลัมน์วันที่จาก DataFrame
df["date_col"] = pd.to_datetime(df["date_col"], dayfirst=True)
df["date_col"] = df["date_col"].dt.tz_localize(TZ_BKK)   # ถ้า naive
df["date_col"] = df["date_col"].dt.tz_convert(TZ_BKK)    # ถ้ามี tz อื่นอยู่แล้ว

# เวลาปัจจุบัน (timezone-aware)
now = pd.Timestamp.now(tz=TZ_BKK)

# แสดงผลเป็น string
df["date_str"] = df["date_col"].dt.strftime("%d/%m/%Y")

# Excel serial number → datetime (WDCS / Legacy Excel)
def excel_serial_to_date(serial):
    return pd.Timestamp("1899-12-30") + pd.Timedelta(days=int(serial))
```

### WDCS Date Input Format
```python
# ฟิลด์ txtDODateF / txtDODateT ใช้ format dd/MM/yyyy เสมอ
date_str = now.strftime("%d/%m/%Y")   # → "14/03/2026"
```

---

## 2. ตัวเลขและสกุลเงิน

### Format มาตรฐาน

| ประเภท | Format | ตัวอย่าง |
|--------|--------|---------|
| จำนวนทั่วไป | `#,##0.##` | `1,234,567.5` |
| จำนวนเต็ม | `#,##0` | `1,234,567` |
| ทศนิยม 2 ตำแหน่ง | `#,##0.00` | `1,234,567.50` |
| เปอร์เซ็นต์ | `0.00%` | `98.50%` |
| สกุลเงินบาท | `฿#,##0.00` | `฿1,234,567.50` |

### Python

```python
# จำนวนทั่วไป
f"{value:,.2f}"          # → "1,234,567.50"
f"{value:,}"             # → "1,234,568"  (ปัดเต็ม)
f"{value:.0f}"           # → "1234568"    (ไม่มี comma)

# เปอร์เซ็นต์
f"{rate:.2%}"            # → "98.50%"

# บาท
f"฿{value:,.2f}"         # → "฿1,234,567.50"
```

### Excel (openpyxl)

```python
cell.number_format = '#,##0.##'    # ตัวเลขทั่วไป (default ของระบบนี้)
cell.number_format = '#,##0.00'    # ทศนิยม 2 ตำแหน่งเสมอ
cell.number_format = '฿#,##0.00'   # สกุลเงินบาท
cell.number_format = '0.00%'       # เปอร์เซ็นต์
```

---

## 3. File Encoding

| ไฟล์ | Encoding | หมายเหตุ |
|------|----------|---------|
| Python script (`.py`) | UTF-8 | default |
| CSV output | UTF-8 with BOM (`utf-8-sig`) | Excel เปิดได้ถูกต้อง |
| CSV input (ทั่วไป) | UTF-8 with BOM (`utf-8-sig`) | รองรับทั้ง BOM และ ไม่มี BOM |
| WDCS Export (`.txt`) | `cp874` | Thai Windows encoding |
| Excel (`.xlsx`) | — | openpyxl จัดการให้ |
| Log file | UTF-8 | |

```python
# CSV output — ใช้ utf-8-sig เสมอ
df.to_csv("output.csv", encoding="utf-8-sig", index=False)

# CSV input — อ่านด้วย utf-8-sig (รองรับทั้งสองแบบ)
df = pd.read_csv("input.csv", encoding="utf-8-sig")

# WDCS file
df = pd.read_csv("exportdo.txt", sep="\t", encoding="cp874", low_memory=False)
```

---

## 4. File & Path

### Output Directory
```
D:\Users\sanonpawit\output\
```
- **ทุก output ต้องบันทึกใน `/output/` เท่านั้น**
- ห้ามแก้ไข Source File ต้นฉบับโดยตรง (ตาม CLAUDE.md §5)

### ชื่อไฟล์
```python
from datetime import datetime
from zoneinfo import ZoneInfo

now = datetime.now(tz=ZoneInfo("Asia/Bangkok"))

# Pattern มาตรฐาน
filename = f"report_{now.strftime('%Y%m%d')}.xlsx"          # report_20260314.xlsx
filename = f"export_{now.strftime('%Y%m%d_%H%M')}.xlsx"     # export_20260314_0930.xlsx
```

### กฎชื่อไฟล์
- ใช้ `snake_case` — ห้ามมีช่องว่าง
- ใช้ timestamp suffix เมื่อ output เป็น batch (กันไฟล์ทับกัน)
- นามสกุลตัวเล็กทั้งหมด (`.xlsx`, `.csv`, `.txt`)

---

## 5. Python Environment

```bash
# คำสั่งที่ถูกต้อง
py script.py
python3 script.py

# ห้ามใช้
python script.py      # ❌ ไม่พบ interpreter บนเครื่องนี้
```

| รายการ | ค่า |
|--------|-----|
| Version | Python 3.13 |
| Interpreter path | `D:\Users\sanonpawit\AppData\Local\Programs\Python\Python313\python.exe` |
| Package manager | `pip` (หรือ `uv` สำหรับ analyzing-data skill) |

### Import Order (PEP 8)
```python
# 1. Standard library
import os
import re
from datetime import datetime
from zoneinfo import ZoneInfo

# 2. Third-party
import pandas as pd
import openpyxl

# 3. Local
from my_module import helper
```

---

## 6. Excel Output

| รายการ | ค่า |
|--------|-----|
| Font | Calibri 14pt |
| Header font | Calibri 14pt Bold, สีขาว (#FFFFFF) บน พื้นน้ำเงิน (#4472C4) |
| Number format | `#,##0.##` (comma separator) |
| Date format | `DD/MM/YYYY` |
| Datetime format | `DD/MM/YYYY HH:mm` |

```python
from openpyxl.styles import Font, PatternFill, Alignment

BASE_FONT    = Font(name="Calibri", size=14)
HEADER_FONT  = Font(name="Calibri", size=14, bold=True, color="FFFFFF")
HEADER_FILL  = PatternFill(start_color="4472C4", end_color="4472C4", fill_type="solid")
NUM_FORMAT   = "#,##0.##"
DATE_FORMAT  = "DD/MM/YYYY"
```

---

## 7. Logging & Error Message

```python
import logging
from zoneinfo import ZoneInfo
from datetime import datetime

# Format มาตรฐาน
logging.basicConfig(
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%d/%m/%Y %H:%M:%S",
    level=logging.INFO,
)

# Print ทั่วไป — ใส่ timestamp เสมอ
def log(msg: str):
    now = datetime.now(tz=ZoneInfo("Asia/Bangkok")).strftime("%d/%m/%Y %H:%M:%S")
    print(f"[{now}] {msg}")
```

---

## 8. DataFrame Best Practices

```python
# อ่าน Excel ที่มี header แถวที่ 2 (Export DO)
df = pd.read_excel(file, dtype=str, header=1)
df.columns = df.columns.str.strip()

# แปลง date column มาตรฐาน
def parse_date_col(series: pd.Series) -> pd.Series:
    return pd.to_datetime(series, dayfirst=True, errors="coerce")\
             .dt.tz_localize("Asia/Bangkok")

# ไม่ใช้ inplace=True (ทำให้ debug ยาก)
df = df.dropna(subset=["key_col"])    # ✅
df.dropna(subset=["key_col"], inplace=True)  # ❌ หลีกเลี่ยง
```

---

## 9. สรุป Quick Reference

```
วันที่แสดงผล    →  DD/MM/YYYY
Timezone        →  Asia/Bangkok (UTC+7)
dayfirst        →  True เสมอ
ตัวเลข          →  #,##0.## (comma)
เงินบาท         →  ฿#,##0.00
CSV encoding    →  utf-8-sig
WDCS encoding   →  cp874
Output folder   →  D:\Users\sanonpawit\output\
Python command  →  py / python3
Excel font      →  Calibri 14pt
```
