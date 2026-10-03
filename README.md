# Mediroza-General-Hospital-Pen-Test-Exercise
Block-Box Penetration Test on Medirozahospital.com


# **Mediroza General Hospital Penetration Testing Report**

NetworkWalks – Batch B083 | Week 4

Engagement Type: Black-Box Penetration Test

Target: https://medirozahospital.com

Testing Period: 5 Days

Purpose: Authorized educational penetration-testing exercise


# 1. Scope and Recon


| Item | Result |
|---|---|
| Target IP | `199.188.201.16` |
| Web Server | LiteSpeed |
| WAF | LiteSpeed WAF |
| HTTP | HTTP/2 |
| Open Ports | 80, 26, 587, 110, 143 |
| Test Type | Black-box |
| Authorization | Granted |


I began with DNS, web-server, WAF, and network-service enumeration. The web application was prioritized because the engagement required access to the patient portal and three confidential reports. The project limited testing to the target domain and prohibited social engineering and denial-of-service activity.



# 2. Web Application Enumeration

| Path | Finding |
|---|---|
| `/patient/` | Patient application |
| `/staff/` | Staff login |
| `/old/` | Legacy directory |

Further enumeration of /patient/ revealed:

| Resource | Function |
|---|---|
| `login.php` | Authentication |
| `portal.php` | Patient portal |
| `download.php` | Report download |
| `reports/` | Report directory |
| `error_log` | Log exposure |
| `logout.php` | Session termination |

At this point, it is important for the tester to identify exposed entry points and analyze authentication and application behavior to report a full picture of vulnerabilities.


# 3. Authentication Testing

The patient login accepted username and password through a POST request to:
```
/patient/login.php
```
I tested the authentication mechanism using controlled invalid input and then SQL injection.
| Test | Result |
|---|---|
| Invalid credentials | Authentication failed |
| SQL injection payload | Authentication bypass |
| Payload | `admin ' --` |
| Response | `HTTP/2 302` |
| Redirect | `portal.php` |

The successful redirect demonstrated that the application's authentication query could be manipulated to bypass the password check.
### Risk: Critical 🛑
### Recommendation: Use parameterized queries/prepared statements and implement secure authentication error handling and monitoring.


# 4. Patient Portal and Report Retrieval

The authenticated portal exposed three patient-report download functions.
I retrieved all three encrypted PDF reports, satisfying the primary M1 objective.
| Report | Identifier |
|---|---|
| Report 1 | `download.php?id=1` |
| Report 2 | `download.php?id=2` |
| Report 3 | `download.php?id=3` |

### Attack Chain

<img width="1204" height="552" alt="mermaid-diagram" src="https://github.com/user-attachments/assets/68089744-9526-4638-ac53-aafbc0edde96" />

### Risk: Critical 🛑
### Recommendation: Enforce server-side authorization for every document request and verify that the authenticated user is permitted to access the requested report. 


# 5. PDF Password Recovery
The three reports were encrypted. I extracted the PDF password hashes and used an online password-cracking tool with wordlists selected based on the assumed relevance of the exercise.

| Item | Result |
|---|---|
| Report 1 hash | Extracted |
| Report 2 hash | Extracted |
| Report 3 hash | Extracted |
| Password recovery | Successful |
| PDFs unlocked | 3 |
| SHA-256 checksums | Recorded |

I did not use John the Ripper for the successful cracking step. The project requires recovery of all three encrypted reports and recognizes that different approaches may be required.     WK4 Mediroza v1
### Risk: High ❗
### Recommendation: Use strong, unique document passwords and combine document encryption with application-level access controls.


# 6. PDF Metadata Analysis
| Field | Report 3 Result |
|---|---|
| Author | `j.malik` |
| Creator | `Mediroza CMS 1.4.2` |
| Producer | `Mediroza Lab Reporting Module` |
| Comments | `DB backup moved to /old before site migration, do not delete` |

The comment provided a direct technical clue to the legacy /old/ directory.    M3_reports_metadata
### Risk: Medium 🟡
### Recommendation: Remove internal usernames, migration notes, filesystem references, and other operational information from document metadata.


# 7. Legacy Database Exposure
Following the metadata clue led to:
```
/old/mediroza_db_backup_2019.sql
```
The back contained sensitive organizational tables
| Table | Information Exposed |
|---|---|
| `staff` | Employee information and monthly salaries |
| `shareholders` | Shareholder and ownership information |

### Exposure Chain
<img width="1896" height="352" alt="mermaid-diagram-2" src="https://github.com/user-attachments/assets/0b6e5d7b-8618-49dc-9126-0cc5eebd1efc" />

### Risk: Critical 🛑
### Recommendation: Remove database backups from the web root, store them outside publicly accessible directories, apply access controls, encrypt backups, and remove obsolete legacy files.

# 8. Findings, Risks, and Remediation
| ID | Finding | Risk |
|---|---|---|
| F-01 | `robots.txt` information disclosure | Low |
| F-02 | Directory listing/application exposure | Medium |
| F-03 | SQL injection authentication bypass | Critical |
| F-04 | Unauthorized patient report access | Critical |
| F-05 | Weak PDF password protection | High |
| F-06 | Sensitive PDF metadata disclosure | Medium |
| F-07 | Exposed legacy database backup | Critical |

### Remediation Priorities
| Priority | Remediation |
|---|---|
| Immediate | Fix SQL injection |
| Immediate | Enforce document authorization |
| Immediate | Remove exposed database backup |
| High | Disable directory listing |
| High | Remove sensitive PDF metadata |
| High | Strengthen PDF password policies |
| Medium | Remove legacy files and exposed logs |
| Medium | Store backups outside the web root |

# Conclusion
<img width="520" height="1552" alt="mermaid-diagram-3" src="https://github.com/user-attachments/assets/ac5196d7-c816-4cee-97da-9f5815d2b907" />

This Mediroza Hospital penetration-testing exercise demonstrated how a series of individually identifiable weaknesses can be connected into a broader attack
path. I progressed from reconnaissance and web enumeration to an SQL injection authentication bypass, gained access to the patient portal, retrieved and
unlocked three protected PDF reports, and analyzed their metadata for additional technical information. The metadata from Report 3 led to the exposed legacy 
database backup containing staff and shareholder information.   

The exercise reinforced the importance of methodical reconnaissance, validating findings before moving to the next stage, and recognizing how information
disclosed by one vulnerability can support the discovery of another. It also provided practical experience with tools such as Nmap, WhatWeb, WAFW00F, curl, 
ExifTool, PDF analysis tools, and password-recovery techniques. Overall, the exercise provided hands-on practice in following a penetration-testing methodology from initial discovery through exploitation, evidence collection, and remediation reporting.





