# Skill: Watcher Notification (Windows Toast)

Pattern สำหรับเพิ่ม Windows toast notification ใน watcher script (Python หรือ PowerShell)

## Prerequisite

ติดตั้ง winotify ครั้งเดียวต่อเครื่อง:
```powershell
py -m pip install winotify
```

## ใช้กับ — Python (watchdog)

ใช้กับ watcher ที่ใช้ `watchdog` library (เช่น `appeal_watcher.py`, `order_summary_watcher.py`)

### Pattern code

วาง 2 helper functions ที่ระดับ module (top-level, ไม่ใช่ใน class):

```python
def _notify_toast(title: str, body: str) -> None:
    """Windows toast notification. Falls back to log-only if winotify missing."""
    log = logging.getLogger("YOUR_LOGGER_NAME")
    try:
        from winotify import Notification
        toast = Notification(
            app_id="YOUR APP NAME",
            title=title,
            msg=body,
            duration="short",
        )
        toast.show()
        log.info("Toast sent: %s", title)
    except ImportError:
        log.warning("winotify not installed (run: py -m pip install winotify)")
        log.info("Notification | %s | %s", title, body.replace("\n", " · "))
    except Exception as e:
        log.warning("Toast failed: %s | %s | %s", e, title, body.replace("\n", " · "))


def _notify_update(file_events: dict, parquet_path: Path) -> None:
    """Build + send notification with details of which files changed when."""
    if not file_events:
        return

    lines = []
    for path, info in file_events.items():
        p = Path(path)
        name = p.name
        if p.exists():
            mtime = datetime.fromtimestamp(p.stat().st_mtime)
            ts = mtime.strftime("%d/%m/%Y %H:%M")
        else:
            ts = datetime.fromtimestamp(info["time"]).strftime("%d/%m/%Y %H:%M")
        evt = info["type"].capitalize()
        lines.append(f"• {name} [{evt}] {ts}")

    size_mb = parquet_path.stat().st_size / 1024 / 1024 if parquet_path.exists() else 0
    body = "\n".join(lines) + f"\nParquet: {size_mb:.1f} MB"

    _notify_toast(title="YOUR_WATCHER Parquet Updated", body=body)
```

### Modify Handler class

เพิ่ม `_file_events` dict และ track event type:

```python
class Handler(FileSystemEventHandler):
    def __init__(self):
        self._changed_files: set[str] = set()
        self._file_events: dict[str, dict] = {}  # path -> {type, time}
        self._last_change = 0.0

    def _trigger(self, event, event_type: str):
        # ... existing filter logic ...
        if event.src_path not in self._file_events:
            self._file_events[event.src_path] = {
                "type": event_type,
                "time": time.time(),
            }
        self._changed_files.add(event.src_path)
        self._last_change = time.time()

    def on_created(self, event):
        log.info("New file: %s", event.src_path)
        self._trigger(event, "created")

    def on_modified(self, event):
        log.info("Modified: %s", event.src_path)
        self._trigger(event, "modified")

    def check_pending(self):
        if not self._changed_files:
            return
        if (time.time() - self._last_change) < 30:
            return
        changed = list(self._changed_files)
        events_snapshot = dict(self._file_events)
        self._changed_files.clear()
        self._file_events.clear()
        log.info("Incremental update: %d changed file(s)", len(changed))
        rc = incremental_update(changed)
        if rc == 0:
            _notify_update(events_snapshot, OUTPUT)
        log.info("Ready. Watching for next change...")
```

### หลักการสำคัญ

- **First event wins** — ถ้าไฟล์เดียวมีหลาย event ในช่วง 30s quiet period → keep event แรก (created มี semantic ที่ชัดกว่า)
- **File mtime over event time** — ใช้ `Path.stat().st_mtime` เพราะ reliable กว่า event timestamp
- **Notify only on success** — call `_notify_update` หลัง `incremental_update` return 0 (success)
- **Snapshot before clear** — copy `_file_events` ก่อน clear เพื่อป้องกัน race ถ้ามี event ใหม่มาระหว่าง update

## ใช้กับ — PowerShell (FileSystemWatcher)

สำหรับ PowerShell watcher (เช่น `AutoMove-ExportDO.ps1`) — เรียก winotify ผ่าน Python subprocess

```powershell
function Send-Toast {
    param([string]$Title, [string]$Body)
    $script = @'
import sys
from winotify import Notification
n = Notification(app_id='YOUR_APP_NAME', title=sys.argv[1], msg=sys.argv[2], duration='short')
n.show()
'@
    try {
        & py -c $script $Title $Body 2>$null | Out-Null
        Write-Log 'INFO' "Toast sent: $Title"
    } catch {
        Write-Log 'WARN' "Toast failed: $_"
    }
}
```

### เรียกใช้

```powershell
$mtime = (Get-Item $dest).LastWriteTime.ToString('dd/MM/yyyy HH:mm')
Send-Toast -Title 'YOUR_TITLE' -Body "$name`nAt $mtime"
```

> หมายเหตุ: ใช้ here-string `@'...'@` (single-quoted) — PowerShell จะไม่ expand `$` ในนั้น → safe สำหรับ Python code

## Troubleshooting

### Toast ไม่ขึ้นแม้ log บอก "Toast sent"

| สาเหตุ | วิธีแก้ |
|--------|--------|
| Focus Assist เปิด | Settings → System → Focus → Off |
| Banner notifications ปิด | Settings → System → Notifications → app ของคุณ → Show notification banners ✓ |
| Action Center ตรวจดูได้ | กด `Win + N` → ดู notification list |

### ทดสอบ winotify โดยตรง
```powershell
py -c "from winotify import Notification; Notification(app_id='Test', title='Test', msg='Hello').show()"
```
ถ้าคำสั่งนี้ส่งแล้วยังไม่เห็น → 100% ปัญหา Windows settings

### Restart watcher หลังแก้โค้ด
```powershell
Stop-ScheduledTask -TaskName "YourWatcher"
Start-ScheduledTask -TaskName "YourWatcher"
```

## Watchers ที่ใช้ pattern นี้แล้ว

| Watcher | File | App ID |
|---------|------|--------|
| Appeal Parquet | `D:\Users\sanonpawit\Auto_Data\ExportDO_Loader\appeal_watcher.py` | `Appeal Watcher` |
| Order Summary Parquet | `D:\Users\sanonpawit\Auto_Data\ExportDO_Loader\order_summary_watcher.py` | `Order Summary Watcher` |
| AutoMove ExportDO | `D:\Users\sanonpawit\Claude_Workspace\AutoMove-ExportDO.ps1` | `AutoMove ExportDO` |

## Notification format มาตรฐาน

```
{Watcher Name} Parquet Updated
• filename.xlsx [Created] DD/MM/YYYY HH:MM
• filename2.xlsx [Modified] DD/MM/YYYY HH:MM
Parquet: X.X MB
```

ใช้ format นี้กับ watcher ใหม่เพื่อ consistency
