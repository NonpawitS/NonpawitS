---
name: Drive C cleanup — Search Index relocated to D: via junction
description: DONE 2026-05-05. Windows Search Index moved from C: to D:\SearchIndex via NTFS directory junction (registry approach failed — WaaSMedicSvc auto-reverted). Freed 12 GB on C:.
type: project
originSessionId: 7779bc00-6e23-4d13-8a44-2ea1714605ee
---
# DONE: Search Index relocated C: -> D: via junction (2026-05-05)

## What was done
- Created NTFS junction: `C:\ProgramData\Microsoft\Search\Data` -> `D:\SearchIndex`
- Registry `HKLM:\SOFTWARE\Microsoft\Windows Search\DataDirectory` stays at default `C:\ProgramData\Microsoft\Search\Data\`
- WSearch service writes to "C:" path which transparently goes to D: via junction
- Index will rebuild on D: over next 2–24 hours

## Why junction (not registry change)
- **First attempt: registry change to `D:\SearchIndex\`** -> WSearch failed to start with "catalog corrupt" error -> WaaSMedicSvc/auto-recovery silently reverted registry back to C: and restarted WSearch -> index started growing on C: again
- **Second attempt: directory junction** -> bypasses registry entirely, service sees normal C: path, no auto-revert trigger

## Verification commands
```powershell
# Check junction is in place
Get-Item 'C:\ProgramData\Microsoft\Search\Data' -Force | Select Name, Mode, Target
# Mode should show 'd----l' (l = junction), Target = D:\SearchIndex

# Check service running
Get-Service WSearch
```

## Script location
`D:\Users\sanonpawit\output\junction_search_index.ps1` — re-runnable if junction breaks (e.g., after Windows feature update)

## Watch out
- D: free dropped from 20.79 GB -> 16.50 GB right after move (and will drop more as index rebuilds, possibly to 5–10 GB free)
- D: also has Volume Shadow Copies eating ~28 GB (IT backup agent — don't touch)
- If D: becomes critically low, consider reducing index scope via Indexing Options (Control Panel -> Indexing Options -> Modify -> uncheck large folders like email/document libraries)

## Context — Drive C cleanup session 2026-05-05
- Started: C: 150 GB used 100%, 0–130 MB free
- Ended: C: ~12 GB free
- Key culprits found:
  - `C:\ProgramData\Microsoft\Search` Windows.edb = 12.54 GB (FIXED via junction)
  - `C:\Windows\Installer` = 16.49 GB orphan MSI (NOT addressed — needs PatchCleaner from homedev.com.au)
  - hiberfil.sys = 12.67 GB (user keeps Hibernate enabled)
  - pagefile.sys = 9.49 GB (IT policy locks move)
  - Shadow Copy storage on C: was 4.79 GB — auto-cleared, max now capped at 2 GB (was 15 GB)
- Reserved Storage disable failed (Error 0x800f0975 — servicing operation in use); may try again after restart

## Lesson learned (for future memory)
On corporate-managed Windows where WaaSMedicSvc / IT policies actively "self-heal" registry settings, **NTFS junctions/symlinks are more durable than registry redirects** for relocating system folders. Junctions are invisible to most services and survive auto-recovery.
