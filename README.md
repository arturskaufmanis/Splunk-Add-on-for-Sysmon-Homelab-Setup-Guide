# 🛡️ Splunk Add-on for Sysmon — Homelab Setup Guide

> **Platform:** Windows Server 2025 · **Splunk:** Free Edition · **Add-on:** v5.0.0  
> **Status:** ✅ Completed Successfully — April 2026
[View my HTML page] (https://arturskaufmanis.github.io/Splunk-Add-on-for-Sysmon-Homelab-Setup-Guide/splunk-sysmon-homelab.html)
---

## 📋 Table of Contents

- [Environment Overview](#environment-overview)
- [Status Before Installation](#status-before-installation)
- [Download & Extraction](#download--extraction)
- [Installation Steps](#installation-steps)
- [Verification](#verification)
- [Lessons Learned](#lessons-learned)
- [Next Steps](#next-steps)

---

## Environment Overview

| Component | Details |
|---|---|
| **Target Server** | Windows Server 2025 |
| **Sysmon** | Installed, running in Smart Mode (low noise config) |
| **Splunk Universal Forwarder** | Installed on Windows Server 2025 |
| **Splunk Instance** | Splunk Free Edition (single-instance) |
| **Index** | `wineventlog` |
| **Sysmon Input Config** | `renderXml = true` already set |

---

## Status Before Installation

Before installing the add-on, the following issues were present:

- Sysmon events were being collected by the Universal Forwarder ✔️
- Events arrived in Splunk under a **generic sourcetype** (`xmlwineventlog`) ❌
- **No field extractions** — `Image`, `CommandLine`, `ParentImage`, `hashes`, etc. were missing ❌
- Splunk Free UI **blocked "Manage Apps"** due to missing `Auth` license feature ❌

---

## Download & Extraction

### Downloaded File

```
C:\Tools\Sysmon\splunk-add-on-for-sysmon_500.tgz
```

Official Splunk Add-on for Sysmon **v5.0.0** from [Splunkbase](https://splunkbase.splunk.com/app/5709).

### Extraction (Two Layers Required)

```
splunk-add-on-for-sysmon_500.tgz
        │
        ▼  Extract (7-Zip or Windows "Extract All")
Splunk_TA_microsoft_sysmon-5.0.0.tar
        │
        ▼  Extract again
Splunk_TA_microsoft_sysmon-5.0.0\
    ├── default\
    └── metadata\
```

> **Note:** There is no `.exe` installer. Splunk add-ons are pure configuration files — this is expected behaviour.

---

## Installation Steps

### Step 1 — Copy the Add-on to Splunk

```powershell
# Copy the extracted folder to the Splunk apps directory
Copy-Item -Recurse "Splunk_TA_microsoft_sysmon-5.0.0" `
  "C:\Program Files\Splunk\etc\apps\"
```

### Step 2 — Restart Splunk (PowerShell, Run as Administrator)

```powershell
cd "C:\Program Files\Splunk\bin"
.\splunk.exe restart
```

### Step 3 — Install on Universal Forwarder *(Optional but Recommended)*

```powershell
Copy-Item -Recurse "Splunk_TA_microsoft_sysmon-5.0.0" `
  "C:\Program Files\SplunkUniversalForwarder\etc\apps\"

# Then restart the forwarder
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe restart
```

> ⚠️ **Why manual copy?** Splunk Free blocks the web-based "Install app from file" UI because it requires the `Auth` license feature. The manual folder copy + restart is the reliable workaround.

---

## Verification

### 1. Confirm Add-on Appears in Splunk Web

Open `http://localhost:8000` → Apps list → confirm **Splunk Add-on for Microsoft Sysmon** is listed.

### 2. Sourcetype Check

```spl
index=wineventlog source=*sysmon*
| stats count by sourcetype source
```

**Expected result:**

```
sourcetype: XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

### 3. Field Extraction Test

```spl
index=wineventlog sourcetype=*sysmon*
| table _time host EventCode Image CommandLine ParentImage ProcessId hashes DestinationIp
```

All listed fields should populate correctly if the add-on is working.

---

## Lessons Learned

| Finding | Detail |
|---|---|
| 🔒 Splunk Free blocks web install | "Manage Apps → Install from file" requires the `Auth` license feature |
| 📁 Manual copy is the workaround | Copy folder to `etc\apps\` and restart Splunk |
| 📦 Two-step extraction | `.tgz` → `.tar` → folder (not a single extract) |
| ⚙️ `renderXml = true` is critical | Must be set in the Forwarder's `inputs.conf` for proper parsing |
| 💻 CLI restart works fine | Both PowerShell and Command Prompt work for `splunk restart` |

---

## Next Steps

- [ ] Tune Sysmon config to stay under Splunk Free's **500 MB/day** indexing limit
- [ ] Create a dedicated `sysmon` index
- [ ] Build dashboards for key Sysmon event codes (1, 3, 7, 11, 22...)
- [ ] Explore [Splunk Security Content](https://github.com/splunk/security_content) detections using Sysmon data

---

## References

- [Splunk Add-on for Microsoft Sysmon — Splunkbase](https://splunkbase.splunk.com/app/5709)
- [Sysmon — Microsoft Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Splunk Universal Forwarder Docs](https://docs.splunk.com/Documentation/Forwarder)
- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config) *(popular Smart Mode base config)*

---

*Documented by: Homelab Project — April 2026*
