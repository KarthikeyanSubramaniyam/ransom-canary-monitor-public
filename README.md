# Ransom Canary Monitor

<p align="center">
  <img src="https://img.shields.io/badge/Security-Endpoint%20Defense-red?style=flat" alt="Security Domain">
  <img src="https://img.shields.io/badge/MITRE%20ATT%26CK-T1486-orange?style=flat" alt="MITRE ATT&CK">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white" alt="Python">
</p>

An automated, defensive endpoint detection engine that identifies ransomware activity and mass file encryption in progress. Evaluates byte-level Shannon entropy shifts on modified files, flags known ransomware extension changes (`.locked`, `.crypto`, `.enc`), and identifies dropped extortion notes to trigger rapid incident containment.

---

## 🎯 The Problem It Solves

Modern ransomware variants evade traditional antivirus signatures by employing novel packers and polymorphic binaries. However, all encryption-based ransomware exhibits an inescapable operational footprint: rapid mass modification of files and the replacement of structured data with near-maximum entropy ciphertext.

**Ransom Canary Monitor** solves this by monitoring behavioral heuristics:
1. **Detects In-Progress Encryption**: Calculates Shannon Byte Entropy (0.0 to 8.0 bits/byte) on modified files, alerting whenever entropy spikes into ciphertext territory ($\ge 7.5$).
2. **Recognizes Strain Extension Signatures**: Matches known ransomware extension patterns (`.locked`, `.crypto`, `.enc`, `.crypted`, `.ransom`, `.locky`, `.blackmatter`).
3. **Catches Extortion Notes**: Identifies dropped recovery instructions and extortion notices (`HOW_TO_DECRYPT`, `README_RESTORE_FILES`).
4. **Enables Rapid Blast-Radius Containment**: Generates structured MITRE ATT&CK `T1486` alerts for endpoint detection and automated quarantine response.

---

## 🏗️ System Architecture & Workflow

```
[ Target Filesystem / Canary Folders ]
                   |
                   v
+------------------------------------+
|    1. Recursive Inode Scanner      | -> Discovers files, reads header metadata,
|   (Directory Traversal Engine)     |    and tracks modification timestamps
+------------------+-----------------+
                   |
                   v
+------------------------------------+
|    2. Extension & Naming Filter    | -> Checks extensions against strain database
|  (Strain Signatures & Extortion)   |    and identifies dropped ransom notes
+------------------+-----------------+
                   |
                   v
+------------------------------------+
|    3. Shannon Byte Entropy Engine  | -> Computes raw byte entropy (0.0 - 8.0)
|    (Ciphertext Spike Detection)    |    flagging encrypted non-archive payloads
+------------------+-----------------+
                   |
                   v
+------------------------------------+
|     4. Severity Classification     | -> Correlates findings, categorizes threat
|    (CRITICAL / HIGH / SUSPICIOUS)  |    level, and maps to MITRE ATT&CK T1486
+------------------+-----------------+
                   |
                   v
+------------------------------------+
|     5. Alert & Forensics Report    | -> Outputs structured JSON / CLI tables
|      (Automated SOAR Quarantine)   |    with file paths and remediation steps
+------------------------------------+
```

---

## 🚀 Technical Specifications & Rules

| Rule Identifier | Severity | Target Signal | MITRE ATT&CK | Description |
| :--- | :--- | :--- | :--- | :--- |
| **`RANSOM_EXTENSION_MATCH`** | `CRITICAL` | File Extensions | [T1486](https://attack.mitre.org/techniques/T1486/) | Extension matches known ransomware family (`.locked`, `.crypto`, `.enc`) |
| **`RANSOM_NOTE_DROPPED`** | `CRITICAL` | File Names | [T1486](https://attack.mitre.org/techniques/T1486/) | Filename matches ransom recovery instructions |
| **`HIGH_BYTE_ENTROPY_ENCRYPTED`** | `HIGH` | Raw Byte Stream | [T1486](https://attack.mitre.org/techniques/T1486/) | Shannon byte entropy $\ge 7.5$ bits/byte indicates encrypted payload |

---

## 💻 CLI Usage & Quickstart

### Audit a Directory for Encryption Indicators
```bash
python -m src.ransom_monitor.cli ./data
```

### Scan with Custom Entropy Threshold & JSON Output
```bash
python -m src.ransom_monitor.cli /shared/storage --entropy 7.6 --json
```

### Run Unit Tests
```bash
python -m unittest discover -s tests -p "test_*.py"
```

---

## 🔒 Production Code & Architecture Notice

* **Public Repository**: Contains behavioral detection specifications, entropy calculation models, and MITRE threat mappings.
* **Private Repository**: Maintained in `KarthikeyanSubramaniyam/ransom-canary-monitor` with kernel filesystem event hooks, automated process isolation daemons, and VSS shadow copy integrity protectors.
