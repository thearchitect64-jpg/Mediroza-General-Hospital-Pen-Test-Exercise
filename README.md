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








