# Mediroza General Hospital

## Security Assessment Engagement

### Milestone 1 - Initial Access & Authentication Testing

**Assessment Type:** Authorized Black-Box Penetration Test  
**Target:** https://medirozahospital.com  
**Analyst:** Anderson Edaiyo Uzezi  
**Assessment Date:** 25 November 2026  
**Status:** Complete

---
## Executive Summary

The objective of this phase was to assess the authentication mechanisms protecting the Mediroza General Hospital web application and determine whether restricted functionality or sensitive resources could be accessed by unauthorized users.

During testing, the Patient Portal authentication mechanism was found to be vulnerable to **SQL Injection**. Exploitation of this vulnerability allowed authentication controls to be bypassed and resulted in unauthorized access to the Patient Portal without valid credentials. Following successful authentication bypass, restricted patient functionality became available, including downloadable pathology reports intended only for authenticated users.

In addition, a publicly accessible database backup identified during the Application Mapping phase remained accessible during testing. Preliminary review confirmed the presence of internal organizational records. Detailed analysis of the exposed data was reserved for **Milestone 3** in accordance with engagement objectives.

No authenticated access to the Staff Portal was obtained during this phase.

---
---

# Assessment Scope

| Item | Value |
|--------|--------|
| Target Domain | medirozahospital.com |
| Assessment Type | Authorized Black-Box |
| Phase | Initial Access & Authentication Testing |
| Status | Complete |

---
## Assessment Objectives

* ✅ Assess Patient Portal authentication security
* ⬜ Assess Staff Portal authentication security
* ✅ Identify authentication weaknesses
* ✅ Determine whether restricted resources could be accessed
* ✅ Validate the impact of authentication bypass vulnerabilities
* ✅ Confirm exposure of sensitive resources available to unauthorized users

---

## Authentication Targets

### Target 1: Patient Portal (`/patient/login.php`)

* **Status:** ✅ Authentication Bypass Successful


* **Evidence:**



  SS014-PATIENT-LOGIN.png




<img width="1366" height="768" alt="SS014-PATIENT -LOGIN" src="https://github.com/user-attachments/assets/d81dc928-7a33-4d77-be51-0e785a530243" />








  









  SS014B-PDF-PATIENT-DISCOVERED.png



<img width="1366" height="768" alt="SS014B-PDF -PATIENT-DISCOVRED" src="https://github.com/user-attachments/assets/7f463fcc-d903-4f6f-a077-299263580b57" />









#### Description
The Patient Portal was identified during the Application Mapping phase and selected as the primary authentication target. Testing identified a SQL injection vulnerability within the login mechanism that allowed authentication controls to be bypassed, granting unauthorized access to restricted patient functionality.

#### Authentication Testing Details

| Parameter | Value / Outcome |
| :--- | :--- |
| **Target** | Patient Portal Login (`/patient/login.php`) |
| **Input Tested** | Username Field |
| **Payload (`Username`)** | `admin'--` |
| **Payload (`Password`)** | `12345` |
| **Result** | Input was not sanitized; SQL comment (`--`) truncated password verification |
| **Outcome** | Authenticated as `admin` without a valid password; full Patient Portal access obtained |

#### Root Cause
* User-controlled input concatenated directly into backend SQL queries
* Absence of parameterized queries or prepared statements
* Lack of server-side input validation on the login form

#### Impact
* Complete authentication bypass without valid credentials
* Unauthorized Patient Portal session establishment
* Access to restricted patient functionality and downloadable pathology reports

---

### Target 2: Staff Portal (`/staff/login.php`)

* **Status:** ⬜ Identified Only (No Bypass Achieved)
* **Evidence:**


 SS015-STAFF-LOGIN-FAIL.png`


<img width="1366" height="768" alt="SS015-STAFF-LOGIN-FAIL" src="https://github.com/user-attachments/assets/c2bdc118-3bf0-4289-8079-7ffcb50b5c92" />







#### Description
The Staff Portal was identified during Application Mapping and included in authentication testing. No successful authentication bypass or authenticated access was achieved against the Staff Portal during this phase.

---

## Finding IA-001: Authentication Bypass via SQL Injection

* **Severity:** `CRITICAL`
* **Status:** Confirmed
* **Affected Endpoint:** `/patient/login.php`

---

### Description

The Patient Portal authentication mechanism was vulnerable to SQL injection. Testing demonstrated that user-controlled input supplied to the username field was incorporated directly into backend SQL queries without sufficient validation or sanitization. Manipulation of the username field (`admin'--`) commented out the remainder of the query, allowing password verification to be bypassed entirely and granting unauthorized access to `/patient/portal.php`.

---

### Authentication Testing Summary

| Field / Attribute | Value |
| :--- | :--- |
| **Target** | Patient Portal Login (`/patient/login.php`) |
| **Input Tested** | Username Field |
| **Username Entered** | `admin'--` |
| **Password Entered** | `12345` |
| **Outcome** | Authenticated as `admin` without a valid password (`12345` ignored) |

---

### Proof of Access

Following the authentication bypass, the application redirected to `/patient/portal.php` and established an active session (confirmed by the presence of a **Logout** option). The authenticated dashboard displayed the **My Reports** section along with the following notice:

> *"Your reports are password protected. Use the password sent to you by the laboratory to open each file."*

#### Restricted Resources Discovered
The authenticated dashboard exposed downloadable pathology reports for multiple patients:

| Report | Laboratory Reference | Date |
| :--- | :--- | :--- |
| Pathology Report - S. Dlamini | `LR-2024-1187` | `2024-11-04` |
| Pathology Report - P. Reddy | `LR-2024-1192` | `2024-11-05` |
| Pathology Report - E. Thompson | `LR-2024-1205` | `2024-11-06` |

---

### Root Cause Analysis

Investigation indicates the vulnerability was caused by:
1. User input concatenated directly into dynamic SQL queries.
2. Lack of parameterized queries or prepared statements (`PDO` / `mysqli` bound parameters).
3. Absence of effective server-side validation on authentication parameters.

---

### Security Impact

* Full Patient Portal access as `admin` without valid credentials
* Complete circumvention of primary authentication controls
* Unauthorized access to restricted patient dashboards and downloadable medical/pathology reports

---


* SQL backup file (`mediroza_db_backup_2019.sql`)

---

### Remediation

1. **Parameterized Queries:** Replace all dynamic SQL queries in `/patient/login.php` with prepared statements and bound parameters (`PDO` or `mysqli`).
2. **Input Validation:** Implement strict server-side allowlist validation on username/ID and password fields.
3. **Password Verification:** Ensure authentication relies on secure password hashing (`password_hash()` / `password_verify()`) rather than raw SQL comparisons.
4. **Source Code Audit:** Conduct a full security review of all authentication and database interaction endpoints across the application.
5. **Regression Testing:** Perform post-remediation verification testing to confirm SQL injection payloads are rejected.

---

### Risk Rating

| Metric | Rating |
| :--- | :--- |
| **Likelihood** | High |
| **Impact** | Critical |
| **Overall Risk** | **CRITICAL** |






---
## Finding IA-002: Review of Exposed Database Backup

* **Severity:** `CRITICAL`
* **Status:** Confirmed — Escalated to Milestone 3
* **Exposed Resource:** `/old/mediroza_db_backup_2019.sql`

---

### Description

A publicly accessible database backup file was originally identified during the Application Mapping & Surface Discovery phase following analysis of information disclosed through `robots.txt`.

#### Discovery Path

```text
robots.txt
    │
    ▼
/old/
    │
    ▼
Directory Listing Enabled
    │
    ▼
mediroza_db_backup_2019.sql
```

The exposed file was subsequently reviewed during **Milestone 1** to assess the impact of the exposure and determine whether sensitive organizational information was accessible. The file remained retrievable without authentication, authorization, or session validation.

---

### Evidence

SS011B-OLD-DIRECTORY.png`

<img width="1366" height="768" alt="SS011B-OLD-DIRECTORY" src="https://github.com/user-attachments/assets/0dd66b39-e7b3-48e2-8396-75eed6df1804" />
















SS011C-DATABASE-BACKUP.png`



<img width="1366" height="768" alt="SS011C-DATABASE-BACKUP" src="https://github.com/user-attachments/assets/0db22f02-4295-4154-9806-b3511476e7e5" />








---

### Proof of Access

A direct HTTP request to `/old/mediroza_db_backup_2019.sql` successfully returned the SQL backup. Metadata inside the dump identified it as an internal organizational database backup:

```text
Mediroza General Hospital - Internal Database Backup
Generated by Mediroza CMS 1.4.2 Backup Module
Database: mediroza_hr
Backup Date: 2019-08-27
```

The successful retrieval of the file confirmed that sensitive backup material was available outside the application's authentication controls.

---

### Initial Data Exposure Observed

A preliminary review confirmed the presence of internal organizational records, including:
* Employee information
* Human resources records
* Internal email addresses
* Internal contact information
* Organizational records
* Shareholder-related information

> **Note:** Detailed analysis of the exposed records was intentionally deferred to **Milestone 3: Critical Data Exposure** in accordance with engagement objectives.

---

### Security Impact

The exposed backup bypasses all application authentication mechanisms and provides direct access to internal data through the web server. Unlike the Patient Portal, which requires authentication, the backup file was accessible directly through a web request and exposed organizational information without requiring valid credentials. This significantly increases the potential impact of a compromise and provides intelligence useful for further targeted attacks against the organization.

---

### Recommendation

1. **Immediate Removal:** Remove publicly accessible backup files immediately.
2. **Disable Indexing:** Disable directory indexing on `/old/`.
3. **Secure Storage:** Store backups outside the web root.
4. **Access Controls:** Restrict access to backup resources using server-level controls.
5. **Log Audit:** Review historical access logs to determine prior exposure of the backup file.
6. **Impact Assessment:** Conduct a full data exposure assessment.
7. **Governance:** Review backup management and retention procedures.

---

## Milestone 1 Risk Summary

| Finding ID | Finding | Severity |
| :--- | :--- | :--- |
| **IA-001** | Authentication Bypass via SQL Injection | **CRITICAL** |
| **IA-002** | Review of Exposed Database Backup | **CRITICAL** |

---

### Risk Overview

Milestone 1 identified two Critical security findings.

**IA-001** demonstrated that the Patient Portal authentication mechanism could be bypassed through SQL injection, allowing unauthorized access to restricted patient functionality without valid credentials.

**IA-002** confirmed that a database backup identified during the Application Mapping phase remained publicly accessible and could be retrieved without authentication, exposing internal organizational information outside normal application security controls.

Together, these findings indicate significant weaknesses in both authentication security and sensitive data protection, requiring immediate remediation.
---

## Evidence Inventory

| Evidence | Description |
| :--- | :--- |
| `SS014-PATIENT-LOGIN.png` | Patient Login Portal |
| `SS014B-PDF-PATIENT-DISCOVERED.png` | Downloadable Patient Reports |
| `SS015-STAFF-LOGIN-FAIL.png` | Staff Login Portal |
| `SS011B-OLD-DIRECTORY.png` | Exposed Directory Listing |
| `SS011C-DATABASE-BACKUP.png` | Retrieved SQL Backup |



---

## Conclusion

Milestone 1 objectives were successfully completed, validating two critical attack paths.

* ✅ **Authentication Bypass via SQL Injection:** Testing confirmed that the Patient Portal authentication mechanism was vulnerable to SQL injection. Successful exploitation bypassed password verification and resulted in unauthorized access to restricted patient functionality, including downloadable pathology reports (`IA-001`).

* ✅ **Unauthenticated Database Backup Exposure:** Review of the database backup identified during the Application Mapping phase confirmed that internal organizational information remained accessible directly through the web server without authentication (`IA-002`).

Although no authenticated access to the Staff Portal was obtained, employee-related information was observed within the exposed database backup. Detailed analysis of those records, including employee and shareholder data, forms part of **Milestone 3: Critical Data Exposure**.

The combination of an authentication bypass vulnerability and a publicly accessible database backup represents a significant compromise of both authentication security and sensitive data protection mechanisms.

---

> **Milestone Status:** ✅ Complete  
>
> **Outcome 1:** Authentication Bypass Successfully Achieved (`IA-001`)  
>
> **Outcome 2:** Public Database Backup Access Confirmed (`IA-002`)  


**Next Phase:** Milestone 2 - Data Extraction and Recovery of Encrypted Patient Records


## 👤 Author
**Anderson Edaiyo Uzezi**  
Cybersecurity Professional B084  
LinkedIn: https://www.linkedin.com/in/anderson-edaiyo-bab74117b

