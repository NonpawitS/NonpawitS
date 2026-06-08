# Skill: check_status_order

> ตรวจสอบสถานะ Order จากไฟล์ Export DO ของระบบ Warehouse

---

## วิธีใช้งาน

### ขั้นตอนที่ 1 — เตรียมไฟล์
- Export ไฟล์ DO จากระบบ Warehouse (format `.xlsx`)
- บันทึกไว้ในเครื่อง เช่น `D:\Users\sanonpawit\Downloads\Export DO Test.xlsx`

### ขั้นตอนที่ 2 — รัน Script

แก้ค่า 2 ส่วนใน script ด้านล่าง:
1. `FILE_PATH` = path ของไฟล์ที่ต้องการ
2. `ORDER_NUMBERS` = วาง Order Number ที่ต้องการตรวจสอบ (ใส่หรือไม่ใส่ `-01` ก็ได้)

---

## Python Script

```python
import pandas as pd
import re

# ===== ตั้งค่าตรงนี้ =====
FILE_PATH = r"D:\Users\sanonpawit\Downloads\Export DO Test.xlsx"

ORDER_NUMBERS = """
582790340365419951
582790290799691761
582790202883343810
""".strip().splitlines()
# =========================

# Stage Pipeline (ลำดับ ascending — สูงสุด = ขั้นตอนสุดท้าย)
STAGE_COLUMNS = [
    ("Create Date",        "Created / Pending"),
    ("Runwave Date",       "Run Wave"),
    ("Start Picking Date", "Start Picking"),
    ("End Picking Date",   "End Picking"),
    ("QC Date",            "QC Done"),
    ("Sent To SAP Date",   "Sent To SAP"),
    ("Weighted Date",      "Weighted"),
    ("Hand Over Date",     "Hand Over"),
    ("Palletize Date",     "Palletize"),
    ("Load Date",          "Load"),
]

EXCLUDE_DOC_TYPES = {"wholesale", "warehousetransfer", "transfer"}

# ===== 1. Load & Normalize =====
df = pd.read_excel(FILE_PATH, dtype=str, header=1)
df.columns = df.columns.str.strip()

# Filter Document Type
if "Document Type Name" in df.columns:
    before = len(df)
    df = df[~df["Document Type Name"].str.lower().str.strip().isin(EXCLUDE_DOC_TYPES)].copy()
    print(f"[Filter] {before} → {len(df)} rows (ตัด Wholesale/Transfer ออก)\n")

# Normalize order key: ตัด suffix -01, -02 ฯลฯ ออกทั้งใน df และ input
order_col = "DO (without -01)" if "DO (without -01)" in df.columns else None
if order_col:
    df["_key"] = df[order_col].str.strip()
else:
    df["_key"] = df["Order no"].str.replace(r"-\d+$", "", regex=True).str.strip()

def normalize_order(o: str) -> str:
    return re.sub(r"-\d+$", "", o.strip())

# Normalize input orders & deduplicate (preserve order)
seen = set()
input_orders = []
for raw in ORDER_NUMBERS:
    key = normalize_order(raw)
    if key and key not in seen:
        seen.add(key)
        input_orders.append(key)

# ===== 2. Vectorized Stage Detection =====
stage_rank_map = {col: rank for rank, (col, _) in enumerate(STAGE_COLUMNS)}
stage_name_map = {rank: name for rank, (_, name) in enumerate(STAGE_COLUMNS)}
stage_date_col_map = {rank: col for rank, (col, _) in enumerate(STAGE_COLUMNS)}

def get_best_rank(row):
    best = 0
    for rank, (col, _) in enumerate(STAGE_COLUMNS):
        val = row.get(col, "")
        if pd.notna(val) and str(val).strip() not in ["nan", "", "NaT"]:
            best = rank
    return best

df["_stage_rank"] = df.apply(get_best_rank, axis=1)
df["_stage_name"] = df["_stage_rank"].map(stage_name_map)

def get_best_date(row):
    col = stage_date_col_map.get(row["_stage_rank"], "")
    if col and col in row:
        val = row[col]
        if pd.notna(val) and str(val).strip() not in ["nan", "", "NaT"]:
            return str(val).strip()
    return "-"

df["_last_date"] = df.apply(get_best_date, axis=1)

# Aggregate: ถ้า order มีหลายแถว (split shipment) เอา rank สูงสุด
df_agg = (
    df.groupby("_key", as_index=False)
    .agg(
        stage_rank=("_stage_rank", "max"),
        doc_type=("Document Type Name", "first"),
        short_reason=("Short Reason", lambda x: x.dropna().iloc[0] if not x.dropna().empty else "-"),
        last_date=("_last_date", "last"),
    )
)
df_agg["stage_name"] = df_agg["stage_rank"].map(stage_name_map)
df_agg = df_agg.set_index("_key")

# ===== 3. Build Result =====
results = []
for order in input_orders:
    if order in df_agg.index:
        row = df_agg.loc[order]
        short = str(row["short_reason"]) if str(row["short_reason"]) != "nan" else "-"
        results.append({
            "Order Number": order,
            "Stage": row["stage_name"],
            "Document Type": row["doc_type"],
            "Short Reason": short,
            "Last Updated": row["last_date"],
        })
    else:
        results.append({
            "Order Number": order,
            "Stage": "⚠️ ไม่พบในไฟล์",
            "Document Type": "-",
            "Short Reason": "-",
            "Last Updated": "-",
        })

# ===== 4. Display =====
result_df = pd.DataFrame(results)
pd.set_option("display.max_colwidth", 45)
pd.set_option("display.width", 220)
print(result_df.to_string(index=False))

print("\n--- สรุป Stage ---")
print(result_df["Stage"].value_counts().to_string())

# Export (uncomment ถ้าต้องการบันทึก)
# result_df.to_excel(r"D:\Users\sanonpawit\output\order_status_result.xlsx", index=False)
# print("\nบันทึกไฟล์แล้ว")
```

---

## โครงสร้าง Stage (ลำดับขั้นตอน)

```
Created / Pending    ← รอ Run Wave
    ↓ Run Wave       ← Wave ถูก Release แล้ว รอ Picker
    ↓ Start Picking  ← Picker รับงานแล้ว
    ↓ End Picking    ← จัดสินค้าเสร็จ
    ↓ QC Done        ← ตรวจสอบคุณภาพแล้ว
    ↓ Sent To SAP    ← ยืนยันออก SAP แล้ว
    ↓ Weighted       ← ชั่งน้ำหนักแล้ว
    ↓ Hand Over      ← ส่งมอบทีม Dispatch แล้ว
    ↓ Palletize      ← ทีม Dispatch เตรียมขนส่ง
    ↓ Load           ← ส่งมอบให้ขนส่ง (ขั้นตอนสุดท้าย)
```

---

## Document Type ที่ถูก Filter ออก

| Document Type | เหตุผล |
|---|---|
| Wholesale | ไม่ใช่ Online Order |
| WarehouseTransfer | ไม่ใช่ Order ของลูกค้า |
| Transfer | ไม่ใช่ Order ของลูกค้า |

---

## หมายเหตุ

- ไฟล์ Excel ต้องมี Header อยู่ที่ **แถวที่ 2** (index 1) ตามรูปแบบ Export ของระบบ
- Column สำคัญ: `DO (without -01)`, `Document Type Name`, `Runwave Date`, `End Picking Date`, `QC Date`, `Sent To SAP Date`, `Weighted Date`, `Hand Over Date`, `Palletize Date`, `Load Date`, `Short Reason`
- Order Number ใน `ORDER_NUMBERS` **ใส่หรือไม่ใส่ `-01` ก็ได้** — script normalize ให้อัตโนมัติ
- รองรับ Order ซ้ำในไฟล์ (split shipment) — ใช้ค่า Stage สูงสุดของ group

---

## Roadmap: Control Tower Upgrade

| Feature | แนวทาง |
|---|---|
| **Detect "Order Stuck"** | คำนวณ `time_in_stage = now - last_updated` แล้ว flag ถ้าเกิน SLA |
| **SLA Threshold per Stage** | กำหนด config `SLA_HOURS = {"Run Wave": 2, "End Picking": 4, ...}` |
| **Batch Monitoring** | รัน Script อัตโนมัติทุก N นาที แล้ว alert เมื่อ Order stuck |
| **Multi-file Compare** | รับหลายไฟล์ → track การเปลี่ยน Stage ข้าม snapshot |
| **Dashboard Export** | Export เป็น Excel พร้อม conditional formatting สีตาม Stage |

---

*Last updated: 2026-02-26 | Migrated to .claude/commands: 2026-03-03*
