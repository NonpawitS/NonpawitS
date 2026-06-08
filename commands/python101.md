# Python101 — ข้อควรรู้สำหรับเครื่องนี้

## การรัน Python บนเครื่องนี้

เครื่องนี้ **ต้องใช้ `py` หรือ `python3`** ในการรัน Python เท่านั้น
ห้ามใช้ `python` เฉยๆ เพราะจะไม่พบ interpreter

```bash
# ✅ ถูกต้อง
py script.py
python3 script.py

# ❌ ผิด — ไม่ทำงาน
python script.py
```

## Python Path บนเครื่องนี้

```
D:\Users\sanonpawit\AppData\Local\Programs\Python\Python313\python.exe
```

## ใช้ใน Batch File / Script

```bat
"D:\Users\sanonpawit\AppData\Local\Programs\Python\Python313\python.exe" script.py
```

## ตรวจสอบ version

```bash
py --version
# Python 3.13.x
```
