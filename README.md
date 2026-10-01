**Batch:**  Networkwalks B083 &nbsp;|&nbsp; 

**Week 4:** &nbsp;|&nbsp; Capstone Project

**Tester:** Ekpenyong Peace

**Report Date:** 01 October 2026

**Classification:** Confidential — Authorized Personnel Only

![Assessment](https://img.shields.io/badge/Assessment-Black%20Box-0366d6?style=for-the-badge)
![Overall Risk](https://img.shields.io/badge/Overall%20Risk-Critical-d73a49?style=for-the-badge)
![Findings](https://img.shields.io/badge/Findings-4-orange?style=for-the-badge)
![Critical](https://img.shields.io/badge/Critical-2-d73a49?style=for-the-badge)
![High](https://img.shields.io/badge/High-1-fb8500?style=for-the-badge)
![Medium](https://img.shields.io/badge/Medium-1-ffd60a?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Authorized-2ea043?style=for-the-badge)

> ⚠️ This project was conducted in a controlled, authorized training environment. The techniques documented here must never be applied to any system without explicit written permission from the owner.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Methodology](#2-scope-and-methodology)
3. [Walkthrough — Milestones 1–3](#3-walkthrough--milestones-1-3)
4. [Findings and Proof of Exploitation](#4-findings-and-proof-of-exploitation)
5. [Risk Rating Summary](#5-risk-rating-summary)
6. [Recommendations and Remediation](#6-recommendations-and-remediation)
7. [Conclusion](#7-conclusion)
8. [Tools Used](#8-tools-used)
9. [Screenshots](#9-screenshots)

---

## 1. Executive Summary

NetworkWalks was engaged to conduct a black-box penetration test of Mediroza General Hospital's public-facing web application at `https://medirozahospital.com`. Testing was carried out under written authorization, limited strictly to the target domain, with social engineering and denial-of-service techniques explicitly out of scope.

The engagement identified **four vulnerabilities, two of which are rated Critical**. An unauthenticated SQL injection flaw in the patient portal login allowed complete authentication bypass, granting access to confidential patient pathology reports without valid credentials. Separately, an exposed `/old/` directory on the production web server hosted a full, unauthenticated database backup (`mediroza_db_backup_2019.sql`) containing staff personal information, national ID numbers, salaries, and company shareholder records — the single most severe finding of the engagement.

In addition, all three patient lab report PDFs recovered from the portal were protected with weak, easily-guessable passwords, and one PDF's internal metadata contained an unredacted comment directly disclosing the path to the exposed database backup — forming a clear chain from a minor information leak to a critical data breach.

Taken together, these findings indicate systemic weaknesses in input validation, server configuration hygiene, and secrets/credential management. Immediate remediation is recommended for both Critical findings prior to any production go-live.

### Summary of Findings

| # | Finding | Severity | Location |
|---|---|---|---|
| 1 | SQL Injection — Authentication Bypass | 🔴 Critical | `/patient/login.php` |
| 2 | Unauthenticated Exposure of Full Database Backup | 🔴 Critical | `/old/mediroza_db_backup_2019.sql` |
| 3 | Weak / Guessable PDF Document Passwords | 🟠 High | Patient portal lab reports |
| 4 | Sensitive Information Disclosure via PDF Metadata | 🟡 Medium | `patient_report_3.pdf` properties |

---

## 2. Scope and Methodology

### 2.1 Target

- **Target:** https://medirozahospital.com
- **Client:** Mediroza General Hospital
- **Engagement Type:** Black-box Penetration Test & Vulnerability Assessment
- **Duration:** 5 Days (Batch B083, Week 4)

### 2.2 Rules of Engagement

- Testing was limited strictly to the target domain and its subdomains/endpoints.
- Social engineering against staff or patients was explicitly **out of scope**.
- Denial-of-service (DoS) testing was explicitly **out of scope**.
- Written authorisation for security testing was provided by the client prior to any testing activity.

### 2.3 Methodology

Testing followed a standard black-box methodology:

1. **Reconnaissance** — passive and active enumeration (WhatWeb fingerprinting, manual browsing, page-source and directory review).
2. **Vulnerability Identification** — manual review of authentication forms, input handling, and application logic.
3. **Exploitation** — SQL injection testing against login forms; password auditing of protected document deliverables.
4. **Post-Exploitation** — metadata analysis of recovered documents to identify secondary points of exposure.

---

## 3. Walkthrough — Milestones 1–3

### Milestone 1 — Initial Access

**Goal:** Attack the website and retrieve the 3 confidential patient PDF lab reports.

Ran `whatweb` against the target — initially returned a `403 Forbidden`, later confirmed to be a tool-fingerprint block rather than a real outage (the site loaded normally in a browser):

```
https://medirozahospital.com [403 Forbidden] HTTPServer[LiteSpeed], IP[199.188.201.16]
```

<img width="1440" height="860" alt="whatweb-403-result" src="https://github.com/user-attachments/assets/6beeff15-df3c-4a0e-ab46-4fc6f7913ab0" />


Manual browsing revealed the site's structure: `Home`, `About`, `Doctors`, `Contact`, `Patient Portal`, and a `Staff Login`.


<img width="1440" height="814" alt="Homepage-in-browser" src="https://github.com/user-attachments/assets/be01da1f-24fd-4319-ab35-8a3d8627bc61" />



Viewing page source on both login pages revealed the CMS in use:

```html<img width="1440" height="860" alt="Patient-portal-page-source" src="https://github.com/user-attachments/assets/6ca00786-71f9-409d-b035-b8066c775fa1" />


<meta name="generator" content="Mediroza CMS 1.4.2">
```

<img width="1440" height="860" alt="Staff-login-page-source(CMS version)" src="https://github.com/user-attachments/assets/85e29846-aea1-48ad-b256-cd3fe260854c" />

<img width="1440" height="860" alt="Patient-portal-page-source" src="https://github.com/user-attachments/assets/bace9b72-d6db-4941-adc2-f31b6b9e3820" />


The Patient Portal login (`/patient/login.php`) accepted a classic SQL injection payload in the username field, bypassing the password check entirely:

```
Username: admin'--
Password: 123456   (any value)
```

<img width="1440" height="810" alt="SQLi -bypass-login" src="https://github.com/user-attachments/assets/23a60d8d-ffdb-4382-8ec9-b2bccd2cb476" />


This logged in successfully and landed on the **My lab reports** dashboard, exposing 3 password-protected PDFs:

| Report | Lab Ref | Date |
|---|---|---|
| Pathology Report – S. Dlamini | LR-2024-1187 | 2024-11-04 |
| Pathology Report – P. Reddy | LR-2024-1192 | 2024-11-05 |
| Pathology Report – E. Thompson | LR-2024-1205 | 2024-11-06 |

<img width="1440" height="860" alt="Patient-portal-dashboard" src="https://github.com/user-attachments/assets/4d71dfff-aa4b-4af9-91f4-c0d5ffced365" />


✅ **Deliverable met:** proof of access (SQLi bypass) + all 3 PDF files retrieved.

---

### Milestone 2 — Data Extraction

**Goal:** Crack the encryption on all 3 retrieved PDF files.

<img width="1440" height="860" alt="Downloaded-PDFs" src="https://github.com/user-attachments/assets/27600e52-d567-41a8-bdb9-639f5a6e1db7" />


**Attempt 1 — `pdf2john` + John the Ripper / Hashcat (Kali):** `pdf2john` extracted a hash from each PDF, but both John the Ripper (jumbo 1.9.0) and Hashcat (mode 10500) on Kali consistently rejected the hash with `No password hashes loaded` / `Token length exception`, even with the format forced explicitly. The same result reproduced on a Windows John the Ripper / Johnny GUI install, ruling out a single-tool or single-OS bug.

<img width="1440" height="860" alt="John-hashcat failed" src="https://github.com/user-attachments/assets/9a22daf8-3eb4-4f2e-9901-0d9a5591509e" />


**Attempt 2 — Direct brute-force with `pikepdf` (Windows):** Bypassed hash extraction entirely and wrote a small Python script to attempt opening each PDF directly against `rockyou.txt`:

```python
import pikepdf, sys

pdf_file, wordlist = sys.argv[1], sys.argv[2]
with open(wordlist, "r", encoding="latin-1", errors="ignore") as f:
    for line in f:
        pw = line.strip()
        if not pw:
            continue
        try:
            with pikepdf.open(pdf_file, password=pw):
                print(f"[+] FOUND: {pw}")
                sys.exit(0)
        except pikepdf.PasswordError:
            continue
print("[-] Not found in wordlist")
```

<img width="869" height="617" alt="crack py script-rockyou txt setup" src="https://github.com/user-attachments/assets/bbb7fd75-8f85-4e3f-934d-6860705dce14" />


**Results:**

| File | Password |
|---|---|
| patient_report_1.pdf (S. Dlamini) | `123456` |
| patient_report_2.pdf (P. Reddy) | `password` |
| patient_report_3.pdf (E. Thompson) | `!@#$%^&` |

<img width="1440" height="140" alt="Password cracked-report 1" src="https://github.com/user-attachments/assets/6b7d32d8-1978-4c73-9048-85d2dbefffc9" />

<img width="1440" height="173" alt="Password cracked-report 2-3" src="https://github.com/user-attachments/assets/29560ef2-98b8-4be2-96eb-1ed0b691074a" />

<img width="1440" height="860" alt="Report 1" src="https://github.com/user-attachments/assets/4d2baa6d-a2a7-4800-96e6-f08d579f7995" />

<img width="1440" height="860" alt="Report 2" src="https://github.com/user-attachments/assets/b28fbcc4-6331-4487-805b-07cd4334857f" />

<img width="1440" height="860" alt="report 3" src="https://github.com/user-attachments/assets/9ade2b73-1392-4a77-9ccf-3fdcad8714c8" />



✅ **Deliverable met:** all 3 files decrypted, contents recovered.

---

### Milestone 3 — Attack (Critical Data Exposure)

**Goal:** Find staff salaries and hospital shareholder details.

Per the milestone hint ("examine all file properties carefully"), checked each decrypted PDF's internal document metadata with `pikepdf`:

```python
import pikepdf
pdf = pikepdf.open(r"patient_report_3.pdf", password="!@#$%^&")
print(pdf.docinfo)
```

<img width="1440" height="860" alt="Metadata output- report 1 2" src="https://github.com/user-attachments/assets/1c36877b-7ef1-44f3-8d25-53fedab4978b" />




Reports 1 and 2 had unremarkable metadata. **Report 3 (E. Thompson)** contained a leaked internal comment:

```
/Author:    j.malik
/Comments:  "DB backup moved to /old before site migration, do not delete"
```

<img width="1440" height="860" alt="Metadata output-report 3" src="https://github.com/user-attachments/assets/ef587bfd-ed40-4ae7-a402-af0e928e79c8" />


Browsing to the disclosed path directly returned an **unauthenticated directory listing** containing a full database backup:

```
https://medirozahospital.com/old/

Index of /old/
  mediroza_db_backup_2019.sql
```

<img width="1440" height="826" alt="Old directory listing" src="https://github.com/user-attachments/assets/005979a7-6fb9-46df-a3f1-8cd7f4144b6d" />


The SQL file's own header confirmed the severity:

```sql
-- WARNING: contains confidential staff and shareholder records
```

**`staff` table** — 30 records, including full name, job title, department, email, phone, **national ID number**, monthly salary (ZAR), and date joined.

<img width="1440" height="832" alt="SQL backup-staff table" src="https://github.com/user-attachments/assets/813a456e-0471-4434-8bc0-7569f2b644d9" />


**`shareholders` table** — 10 records:

| Shareholder | Share % |
|---|---|
| Dr. Rajesh Naidoo | 18% |
| Cedar Health Holdings (Pty) Ltd | 15% |
| Dr. Johan van der Merwe | 12% |
| Reddy Family Trust | 11% |
| Thabo Molefe | 10% |
| Sarah Botha | 9% |
| Dr. Ahmed Kara | 8% |
| Naledi Zulu | 7% |
| Michael Roberts | 6% |
| Dr. Vikram Chetty | 4% |

<img width="1440" height="826" alt="SQL backup-shareholders table" src="https://github.com/user-attachments/assets/be56244d-80c1-41a7-8cd8-b76e4c9d0575" />


✅ **Deliverable met:** full documented evidence of the exposure, salaries and shareholder data uncovered.

---

## 4. Findings and Proof of Exploitation

### 4.1 SQL Injection — Authentication Bypass

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1 (approx.)** | 9.8 — `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H` |
| **Location** | `https://medirozahospital.com/patient/login.php` (POST: `username`, `password`) |

**Description:** The patient portal login form fails to sanitise or parameterise the `username` field before using it in a backend SQL query. Submitting a crafted username terminates the intended query early and comments out the password check, allowing authentication to succeed without knowledge of a valid password.

**Proof of Exploitation:**
```
Username:  admin'--
Password:  123456   (any value)
```
This payload authenticated successfully as the "admin" account, bypassing password verification entirely and returning the patient portal dashboard, from which three confidential pathology reports were retrieved.

**Impact:** An unauthenticated attacker can gain access to any patient account, or an administrative context, without valid credentials. This directly exposes confidential medical records and violates data protection obligations for health information.

---

### 4.2 Unauthenticated Exposure of Full Database Backup

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1 (approx.)** | 9.1 — `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` |
| **Location** | `https://medirozahospital.com/old/mediroza_db_backup_2019.sql` (directory listing enabled) |

**Description:** A legacy directory (`/old/`) left on the production web server following a prior site migration is browsable and contains a complete, unauthenticated SQL database export of the hospital's internal HR database (`mediroza_hr`). The file's own header comment explicitly warns: *"contains confidential staff and shareholder records."* No access control, authentication, or even basic obscurity protects this path.

**Proof of Exploitation:**
```
GET https://medirozahospital.com/old/
→ Index of /old/
  mediroza_db_backup_2019.sql
```
The downloaded file contains two tables in full — `staff` (30 records including national ID numbers and salaries) and `shareholders` (10 records including equity stakes). This directly satisfies both Milestone 3 data-exposure objectives in a single, unauthenticated download.

**Impact:** This is the most severe finding of the engagement. Exposure of national ID numbers and salary data constitutes a significant personal-data breach with regulatory implications (e.g. POPIA in South Africa). Exposure of shareholder identities and equity stakes constitutes a serious confidentiality and competitive-harm risk.

---

### 4.3 Weak / Guessable Patient Document Passwords

| | |
|---|---|
| **Severity** | 🟠 High |
| **CVSS 3.1 (approx.)** | 7.5 — `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` |
| **Location** | Patient portal — 3 downloadable pathology report PDFs |

**Description:** All three password-protected PDF lab reports retrieved from the patient portal used weak passwords present in common password wordlists. No organisational password policy appears to govern how the lab reporting system generates per-document passwords.

**Proof of Exploitation:**
```
patient_report_1.pdf (S. Dlamini)   password: 123456
patient_report_2.pdf (P. Reddy)     password: password
patient_report_3.pdf (E. Thompson)  password: !@#$%^&
```
Each password was recovered using a straightforward dictionary attack against `rockyou.txt`, with no need for brute-force or targeted guessing.

**Impact:** Even where document-level encryption is used as a compensating control for confidential medical data, weak passwords render that control ineffective, since intercepted or leaked files can be trivially decrypted.

---

### 4.4 Sensitive Information Disclosure via PDF Metadata

| | |
|---|---|
| **Severity** | 🟡 Medium |
| **CVSS 3.1 (approx.)** | 5.3 — `AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` |
| **Location** | `patient_report_3.pdf` — internal document properties |

**Description:** The internal metadata of `patient_report_3.pdf` contains a non-standard `/Comments` field and an `/Author` field never intended for external distribution. This metadata is not visible when the PDF is opened and read normally, but is trivially extractable with standard PDF tooling.

**Proof of Exploitation:**
```
/Author:    j.malik
/Comments:  "DB backup moved to /old before site migration, do not delete"
```
This comment directly disclosed the exact server path later confirmed to host the exposed database backup in Finding 4.2, forming a clear disclosure chain from a low-effort metadata check to a critical data breach.

**Impact:** Internal operational notes embedded in externally-distributed documents can hand attackers a direct roadmap to further, more severe vulnerabilities, as demonstrated in this engagement. It also discloses an internal staff identifier (`j.malik`) that could support future credential-guessing or phishing attempts.

---

## 5. Risk Rating Summary

Findings are rated using CVSS 3.1 base scores and standard qualitative severity bands. Ratings reflect the ease of exploitation (all findings required no authentication and only standard tooling) and the sensitivity of data exposed (medical, financial, and personally identifiable information).

| # | Finding | Severity | CVSS |
|---|---|---|---|
| 1 | SQL Injection — Authentication Bypass | 🔴 Critical | 9.8 |
| 2 | Unauthenticated Exposure of Full Database Backup | 🔴 Critical | 9.1 |
| 3 | Weak / Guessable PDF Document Passwords | 🟠 High | 7.5 |
| 4 | Sensitive Information Disclosure via PDF Metadata | 🟡 Medium | 5.3 |

---

## 6. Recommendations and Remediation

### 6.1 SQL Injection — Authentication Bypass
- Rewrite all database queries to use parameterised statements / prepared statements; never concatenate user input into SQL.
- Apply strict server-side input validation on all authentication fields.
- Deploy a Web Application Firewall (WAF) as a defence-in-depth control, not a primary fix.
- Conduct a full code review of all other login and search forms for the same class of vulnerability.

### 6.2 Exposed Database Backup
- Immediately remove the `/old/` directory and all legacy files from the production web root.
- Disable directory listing (autoindex) at the web server level (LiteSpeed configuration).
- Store all database backups outside the web-servable directory tree, ideally in encrypted, access-controlled offline storage.
- Rotate all credentials and treat all data in the exposed backup (staff PII, national IDs, shareholder data) as compromised; notify affected individuals and relevant regulators as required by applicable data protection law.

### 6.3 Weak Document Passwords
- Enforce a minimum password complexity/length standard for all system-generated document passwords (12+ characters, non-dictionary, unique per document).
- Consider replacing password-protected PDFs with a secure authenticated download link as the primary distribution mechanism.

### 6.4 Metadata Disclosure
- Strip or sanitise document metadata (author, comments, custom properties) from all patient-facing generated documents before distribution.
- Review internal processes to ensure operational/migration notes are never embedded in production document templates.
- Conduct periodic metadata audits of all externally-distributed document types.

---

## 7. Conclusion

This engagement identified a clear and connected chain of vulnerabilities — from a critical authentication bypass, through weak document security, to an incidental metadata leak that led directly to the single most damaging finding of the assessment: an unauthenticated, fully exposed database backup containing staff and shareholder records.

Mediroza General Hospital is advised to prioritise remediation of both Critical findings (4.1 and 4.2) immediately, followed by the High and Medium findings, and to treat the data contained in the exposed backup as compromised pending a full incident response review.

---

## 8. Tools Used

![Kali Linux](https://img.shields.io/badge/Kali%20Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![VirtualBox](https://img.shields.io/badge/VirtualBox-183A61?style=for-the-badge&logo=virtualbox&logoColor=white)

- `whatweb` — web technology fingerprinting
- Browser DevTools / View Source — manual reconnaissance and form analysis
- Manual SQL injection payloads — authentication logic testing
- `pdf2john` — PDF hash extraction (attempted)
- John the Ripper / Hashcat (Kali & Windows) — attempted offline hash-based password recovery
- `pikepdf` (Python) — direct PDF password brute-forcing and metadata extraction
- `rockyou.txt` — wordlist

---

## 9. Screenshots

All evidence screenshots are embedded inline throughout the walkthrough above, and are stored in the `/Screenshots` folder following this naming convention: `M<milestone>_<step>_<description>.png`.

| # | Filename | Description |
|---|---|---|
| 1 | `M1_01_whatweb-403.png` | whatweb fingerprinting — 403 result |
| 2 | `M1_02_homepage.png` | Homepage in browser |
| 3 | `M1_03_staff-login-source.png` | Staff login page source (CMS version leak) |
| 4 | `M1_04_patient-portal-source.png` | Patient portal page source ("Incorrect password" leak) |
| 5 | `M1_05_sqli-bypass-login.png` | SQLi bypass login (`admin'--`) success |
| 6 | `M1_06_patient-portal-dashboard.png` | Patient portal dashboard — 3 PDFs listed |
| 7 | `M2_01_downloaded-pdfs.png` | Downloaded PDFs in folder |
| 8 | `M2_02_pdf2john-hash.png` | pdf2john hash extraction |
| 9 | `M2_03_johnhashcat-failed.png` | John / Hashcat failed attempts |
| 10 | `M2_04_crack-script-setup.png` | crack.py script + rockyou.txt setup |
| 11 | `M2_05_cracked-report1.png` | Password cracked — report 1 (`123456`) |
| 12 | `M2_06_cracked-report2-3.png` | Passwords cracked — reports 2 & 3 |
| 13 | `M2_07_pdf-content-report1.png` | Opened PDF content — Dlamini report |
| 14 | `M3_01_metadata-report1.png` | Metadata output — report 1 (clean) |
| 15 | `M3_02_metadata-report2.png` | Metadata output — report 2 (clean) |
| 16 | `M3_03_metadata-report3-leak.png` | Metadata output — report 3 (leak found) |
| 17 | `M3_04_old-directory-listing.png` | `/old/` directory listing |
| 18 | `M3_05_db-backup-staff-table.png` | SQL backup — staff table |
| 19 | `M3_06_db-backup-shareholders-table.png` | SQL backup — shareholders table |

---
## 10. Disclaimer

This project was produced as part of a controlled educational exercise by Networkwalks. The target was authorized for security testing, and all activity was performed within the agreed scope. The techniques described here must never be used against systems without explicit written permission from the owner.


**Author:** Ekpenyong Peace Inemesit
 Cybersecurity Professional
