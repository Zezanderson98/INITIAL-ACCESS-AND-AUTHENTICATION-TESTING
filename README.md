# Mediroza General Hospital

## Security Assessment Engagement

### Milestone 1 - Initial Access & Authentication Testing

**Assessment Type:** Authorized Black-Box Penetration Test  
**Target:** https://medirozahospital.com  
**Analyst:** Anderson Edaiyo Uzezi  
**Assessment Date:** 25 November 2026  
**Status:** Complete

---

# Executive Summary

The objective of this phase was to assess the authentication mechanisms protecting the Mediroza General Hospital web application and determine whether restricted functionality or sensitive resources could be accessed.

During testing, authenticated access was successfully obtained to the Patient Portal. Access to the portal exposed downloadable pathology reports and confirmed the existence of protected patient resources behind authentication controls.

In addition, a publicly accessible database backup discovered during reconnaissance remained accessible and provided access to internal organizational data. Initial review confirmed the presence of employee and business-related records. Detailed analysis of the exposed data was reserved for Milestone 3 as required by the assessment objectives.

No authenticated access to the Staff Portal was obtained during this phase.

---

# Assessment Scope

| Item | Value |
|--------|--------|
| Target Domain | medirozahospital.com |
| Assessment Type | Authorized Black-Box |
| Phase | Initial Access & Authentication Testing |
| Status | Complete |

---

# Assessment Objectives

The objectives of this phase were:

- Assess Patient Portal authentication security
- Assess Staff Portal authentication security
- Determine whether access to restricted resources could be obtained
- Identify protected functionality available after authentication
- Confirm exposure of sensitive resources available to unauthorized users

---

# Authentication Targets

## Target 1 - Patient Portal

### Location

```text
/patient/login.php
```

### Status

✅ Access Achieved

### Evidence

- SS014-PATIENT-LOGIN.png
- SS018-PATIENT-PORTAL.png

### Description

The Patient Portal was identified during application mapping and selected as the primary authentication target.

Testing successfully progressed beyond the login interface and resulted in authenticated access to restricted patient functionality.

---

## Target 2 - Staff Portal

### Location

```text
/staff/login.php
```

### Status

⚪ Identified Only

### Evidence

- SS015-STAFF-LOGIN.png

### Description

The Staff Portal was identified and reviewed during testing.

No authenticated access to the Staff Portal was obtained during Milestone 1.

---

# Finding IA-001

## Authenticated Access to Patient Portal

### Severity

**High**

---

## Description

Authentication testing resulted in successful access to the patient portal located at:

```text
/patient/portal.php
```

Following authentication, a valid session was established and the application exposed resources unavailable to unauthenticated users.

---

## Evidence

- SS018-PATIENT-PORTAL.png
- PDF-PATIENT-DISCOVERED.png

---

## Proof of Access

The authenticated application displayed:

```text
My Reports
```

and presented the message:

```text
Your reports are password protected.
Use the password sent to you by the laboratory
to open each file.
```

The portal also displayed a logout button, confirming a valid authenticated session.

---

## Restricted Resources Discovered

The patient dashboard exposed downloadable medical reports.

| Report | Laboratory Reference | Date |
|----------|----------|----------|
| Pathology Report - S. Dlamini | LR-2024-1187 | 2024-11-04 |
| Pathology Report - P. Reddy | LR-2024-1192 | 2024-11-05 |
| Pathology Report - E. Thompson | LR-2024-1205 | 2024-11-06 |

---

## Retrieved Report References

### Report 1

```text
Pathology Report
Patient: S. Dlamini
Reference: LR-2024-1187
Date: 2024-11-04
```

### Report 2

```text
Pathology Report
Patient: P. Reddy
Reference: LR-2024-1192
Date: 2024-11-05
```

### Report 3

```text
Pathology Report
Patient: E. Thompson
Reference: LR-2024-1205
Date: 2024-11-06
```

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

* `SS016-OLD-DIRECTORY.png`
* `SS017-DATABASE-BACKUP.png`
* `OLD PAGE.png`

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
| **IA-001** | Authenticated Access to Patient Portal | `HIGH` |
| **IA-002** | Review of Exposed Database Backup | `CRITICAL` |

---

## Evidence Inventory

| Evidence | Description |
| :--- | :--- |
| `SS014-PATIENT-LOGIN.png` | Patient Login Portal |
| `SS015-STAFF-LOGIN.png` | Staff Login Portal |
| `SS016-OLD-DIRECTORY.png` | Exposed Directory Listing |
| `SS017-DATABASE-BACKUP.png` | Retrieved SQL Backup |
| `SS018-PATIENT-PORTAL.png` | Authenticated Patient Portal |
| `PDF-PATIENT-DISCOVERED.png` | Downloadable Patient Reports |
| `OLD PAGE.png` | Public Directory Listing |

---

## Conclusion

Milestone 1 objectives were successfully completed, validating two significant attack paths:

* ✅ **Authenticated Patient Portal Access:** Resulted in access to restricted patient functionality and downloadable pathology reports (`IA-001`).
* ✅ **Unauthenticated Database Backup Exposure:** Confirmed that internal organizational and HR records were accessible directly from the web server outside authentication controls (`IA-002`).

Although no authenticated access to the Staff Portal was obtained, employee-related information was observed within the exposed backup. Detailed analysis of this data forms part of **Milestone 3: Critical Data Exposure**.

---

> **Milestone Status:** ✅ Complete  
> **Outcome 1:** Authenticated Patient Portal Access Achieved  
> **Outcome 2:** Public Database Backup Access Confirmed


**Next Phase:** Milestone 2 - Crack the encryption on all 3 retrieved files


