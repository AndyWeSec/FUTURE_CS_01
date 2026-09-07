# FUTURE_CS_01
Repository for the Cyber Security Internship under the Fellowship Program at Future Interns (August 2026 – September 2026).

# Future Interns Cyber Security Internship Portfolio
August 2026 – September 2026

## 📌 Project Tasks
* [Task 1: Vulnerability Assessment Report (demo.testfire.net)](#vulnerability-assessment-report-demotestfirenet)
* [Task 2: Phishing Email Analysis](#vulnerability-assessment-report-phishing-email-analysis)



# Vulnerability Assessment Report: demo.testfire.net

**Target Domain:** http://testfire.net / https://testfire.net  
**Date of Assessment:** August 27, 2026  
**Tools Used:** OWASP ZAP, Nmap  
**Classification System:** Low / Medium / High  

---

## 📋 Executive Summary
A comprehensive security assessment of `demo.testfire.net` was conducted using automated and passive analysis techniques. The target site is highly vulnerable due to severe software obsolescence and critical input validation flaws. Immediate structural remediation is required to protect user financial data, fix identity management pathways, and secure the hosting environment.
<img width="1396" height="808" alt="Screenshot 1" src="https://github.com/user-attachments/assets/006baf26-fa13-4bb6-bc55-eb8018476679" />
*Target application: `demo.testfire.net`, HCL's AltoroMutual demo banking site used to demonstrate web application vulnerability scanning.*

---

## 🚨 Detailed Findings & Risk Breakdown
<img width="1465" height="833" alt="Screenshot 4" src="https://github.com/user-attachments/assets/0cb49d6f-e10f-48a8-a0d7-d326dd366d88" />
*OWASP ZAP's completed Automated Scan (Pen Test policy) against `demo.testfire.net`, showing 14 alert categories — including SQL Injection, Reflected XSS, and missing anti-CSRF tokens — the basis for the findings below.*


### 1. SQL Injection (SQLi)
* **What is the issue?** The web application fails to properly clean inputs typed by users into forms before sending them to the backend database.
* **Why does it matter?** Attackers can input malicious database commands to trick the application. This allows them to bypass the login portal without a password, read secret customer records, steal financial data, or completely alter database content.
* **Risk Level:** 🔴 **High**
* **Remediation:** Implement **Parameterized Queries** (Prepared Statements) in the backend code. This treats all user inputs strictly as plain data rather than executable code.
<img width="1446" height="829" alt="Screenshot 3" src="https://github.com/user-attachments/assets/fecb0b16-aa09-4f03-9443-dc08df554d77" />
*OWASP ZAP's Automated Scan actively probing `demo.testfire.net`'s login (`doLogin`) and feedback forms — the process that generated the SQL Injection, XSS, and CSRF findings above.*


### 2. Reflected Cross-Site Scripting (XSS)
* **What is the issue?** The site accepts data from a web request and prints it directly back into the user's browser page without checking if it contains harmful code.
* **Why does it matter?** Attackers can build a malicious link and trick a customer into clicking it. When the page loads, the attacker's script executes automatically inside the customer's browser, allowing the criminal to steal login session tokens or hijack the user's banking account dashboard.
* **Risk Level:** 🔴 **High**
* **Remediation:** Apply strict **Context-Aware Output Encoding** to ensure any text rendered on screen is treated as text, neutralizing any embedded script tags.

### 3. Missing or Expired SSL/TLS Configuration
* **What is the issue?** The standard site relies on insecure, plain HTTP. While an encrypted HTTPS version exists, it serves an expired digital certificate (`NET::ERR_CERT_DATE_INVALID`), triggering privacy warnings in modern browsers.
* **Why does it matter?** All network traffic between users and the bank is unencrypted. Password data and banking details are completely exposed in plaintext to anyone eavesdropping on the network (like a public Wi-Fi hacker). Security-conscious users attempting to use HTTPS are blocked by a frightening privacy warning.
* **Risk Level:** 🔴 **High**
* **Remediation:** 
  * Deploy and maintain a valid, current SSL/TLS certificate.
  * Configure the web server to automatically redirect all traffic from HTTP (Port 80) to HTTPS (Port 443).
  * Enable **HTTP Strict Transport Security (HSTS)** to force browsers to always connect securely.

### 4. Slowloris Denial of Service (DoS) Vulnerability (CVE-2007-6750)
* **What is the issue?** The server's connection settings make it vulnerable to a specialized resource exhaustion attack.
* **Why does it matter?** A single attacker sending extremely slow, incomplete web requests can tie up all available connections on the server. This crashes the website and prevents real customers from logging in or using the banking service.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Configure the web application firewall or load balancer to strictly limit the maximum time allowed for clients to keep a connection open without sending data.

### 5. Outdated Backend Software Stack (Apache-Coyote/1.1)
* **What is the issue?** The server identifies itself as running *Apache-Coyote/1.1*, a highly outdated application framework server element.
* **Why does it matter?** This specific version leaks structural details about the server stack and contains multiple publicly documented bugs that attackers look for during the exploration phase.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Update the underlying server software to a supported, modern version. Configure the production server settings to hide application banners and software version information from public HTTP headers.
<img width="1459" height="831" alt="Screenshot 2026-08-29 at 18 58 22" src="https://github.com/user-attachments/assets/79a4f639-94f4-4cc6-9a1d-75c914d41631" />
*Nmap scan (`--unprivileged -sT -sV --script vuln`) confirming the outdated `Apache Tomcat/Coyote JSP engine 1.1` banner on port 80, and flagging multiple forms lacking anti-CSRF tokens (see Finding 7).*

### 6. Weak Cryptographic Configuration (1024-bit Diffie-Hellman Keys)
* **What is the issue?** The server relies on a weak 1024-bit cryptographic key size for its encrypted channels.
* **Why does it matter?** Modern compute power makes 1024-bit encryption crackable. Sophisticated threat actors can potentially decrypt historical captured traffic to read data they shouldn't see.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Reconfigure the server's TLS cipher suite configuration to exclusively enforce a minimum key length of **2048 bits** (or use modern Elliptic Curve Cryptography).

### 7. Cross-Site Request Forgery (CSRF) & Missing Tokens
* **What is the issue?** Web forms on the application lack unique cryptographic tracking tokens.
* **Why does it matter?** A malicious website can trick a logged-in banking user into loading a hidden script that automatically submits forms on `demo.testfire.net` on their behalf—such as initiating a money transfer without the user realizing it.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Add secure, unique, and unpredictable **Anti-CSRF Tokens** to every state-changing form submission on the site.

### 8. Missing Security Hardening Headers (CSP & Anti-Clickjacking)
* **What is the issue?** Essential defense headers—like Content Security Policy (CSP) and X-Frame-Options—are completely missing from the server's responses.
* **Why does it matter?** Without CSP, malicious code injection from XSS attacks runs with zero browser restrictions. Without clickjacking protection, attackers can frame the bank's login page underneath a deceptive transparent layer on another site to steal click interactions.
* **Risk Level:** 🟡 **Low**
* **Remediation:** Configure the global web server configuration file to include modern security headers:
  * `Content-Security-Policy`
  * `X-Frame-Options: DENY`
  * `X-Content-Type-Options: nosniff`

---

## 🛠️ Summary Action Roadmap for the Business

1. **Immediate (24–48 Hours):** Replace the expired SSL certificate on Port 443, fix the site mapping to force safe redirection from plain HTTP to HTTPS, and update the server configuration to block outdated 1024-bit encryption.
2. **Short Term (1–2 Weeks):** Patch the underlying application code to implement prepared statements for SQL queries and secure output encoding to eliminate SQL Injection and XSS entry points.
3. **Medium Term (Monthly Maintenance):** Schedule a maintenance window to upgrade the outdated Apache-Coyote/1.1 backend and configure standard web security headers to complete the defense-in-depth posture.


# Vulnerability Assessment Report: Phishing Email Analysis
 
**Assessment Type:** Email Security / Social Engineering Analysis
**Date of Assessment:** *August 31, 2026* 
**Tools Used:** MXToolbox Header Analyzer  
**Classification:** 🔴 **Phishing — High Risk**


---

## 📋 Executive Summary
An email purporting to be from an internal "IT Service Desk" was submitted for analysis and confirmed to be a phishing attempt. The message combines technical spoofing techniques — including a typosquatted sender domain and an insecure malicious link — with classic social engineering tactics such as urgency and impersonation. The email should be treated as malicious, reported, and blocked at the mail gateway level.

---

# API Security Risk Analysis 

## 🚨 Detailed Findings & Risk Breakdown

<img width="1470" height="739" alt="Screenshot 2026-09-07 at 12 26 16" src="https://github.com/user-attachments/assets/c0d9577c-b38d-43cc-b9a2-a236dd623f03" />
Figure 1.1: Automated HTTP GET request evaluation targeting the postman-echo.com server. The inspection reveals a successful 200 OK transmission returning raw JSON payload parameters, while highlighting architectural anomalies including public unauthenticated data routing and client software framework exposure

### 1. Spoofed Sender Address
* **What is the issue?** The display name claims to be the official "IT Service Desk," but header analysis (via MXToolbox Header Analyzer) reveals the true sender address as `security-update@micros0ft-support.com` — with the letter "o" replaced by the number "0".
* **Why does it matter?** This is a deliberate typosquatting technique designed to visually resemble a legitimate Microsoft domain, exploiting the tendency of users to trust a familiar display name without checking the underlying address. It is specifically engineered to bypass casual visual inspection.
* **Risk Level:** 🔴 **High**
* **Remediation:** Configure email security gateways to flag or block domains using homoglyph/character-substitution patterns targeting trusted brand names. Train staff to inspect the actual sender address, not just the display name.
  <img width="1334" height="359" alt="Screenshot 2026-08-31 at 10 09 16" src="https://github.com/user-attachments/assets/a1d611a6-9c22-40b0-ba0f-a27f8897ac7c" />
 *MXToolbox Header Analyzer output confirming the true sender address (`security-update@micros0ft-support.com`) behind the "IT Service Desk" display name — note the zero substituted for the letter "o".*


### 2. Insecure URL Protocol
* **What is the issue?** The embedded hyperlink uses unencrypted `http://` (`http://login-microsoft-secure-portal.com`) rather than secure `https://`.
* **Why does it matter?** Legitimate corporate login portals — especially those belonging to major providers like Microsoft — universally use HTTPS. The use of plain HTTP is a strong technical red flag and indicates the link was not built through legitimate infrastructure.
* **Risk Level:** 🔴 **High**
* **Remediation:** Implement email filtering rules that flag outbound links using insecure protocols, and reinforce user training on checking for HTTPS before submitting any credentials.

### 3. Malicious Destination Site
* **What is the issue?** The link redirects to a page designed to harvest sensitive information — including passwords, phone numbers, and credit card details — or to trick visitors into installing malicious software.
* **Why does it matter?** Modern browsers (e.g. Google Chrome) already flag this specific domain as unsafe and display a warning page, independently corroborating that the destination is a known malicious site rather than a false positive.
* **Risk Level:** 🔴 **High**
* **Remediation:** Block the domain at the DNS/firewall level, submit it to threat intelligence feeds (e.g. Google Safe Browsing, PhishTank) if not already flagged, and ensure endpoint protection is active organization-wide.
<img width="1162" height="756" alt="Screenshot 2026-08-31 at 10 18 20" src="https://github.com/user-attachments/assets/4d9ebe68-c890-4524-a959-6bfce3d68874" />
*Google Chrome's built-in Safe Browsing protection flagging `login-microsoft-secure-portal.com` as a dangerous site, independently corroborating the phishing link's malicious destination.*


### 4. Generic Greeting
* **What is the issue?** The email opens with "Dear Customer" rather than addressing the recipient by name.
* **Why does it matter?** Legitimate internal IT communications typically reference the employee by name or account-specific details. A generic greeting suggests a mass-distributed phishing campaign rather than a targeted, authentic internal message.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Encourage staff to treat generically-addressed "urgent" emails from internal departments with suspicion, and verify through a separate communication channel (e.g. phone, Slack) before acting.

### 5. Artificial Urgency
* **What is the issue?** The email threatens permanent account suspension within a strict 2-hour window.
* **Why does it matter?** This is a classic psychological pressure tactic used to provoke hasty action and bypass rational scrutiny, a hallmark of social engineering attacks designed to short-circuit normal verification habits.
* **Risk Level:** 🟠 **Medium**
* **Remediation:** Train staff to recognize artificial time-pressure tactics as a phishing indicator, and establish a "cool-down" policy — any account-security email demanding immediate action should be independently verified before compliance.

### 6. Spelling and Grammar Errors
* **What is the issue?** The email body contains grammatical errors, including a misspelling of the word "corporate."
* **Why does it matter?** Poor spelling and grammar remain a common — if increasingly inconsistent — indicator of phishing, often resulting from mass-produced templates, translation artifacts, or non-native authorship by threat actors.
* **Risk Level:** 🟡 **Low**
* **Remediation:** Include spelling/grammar inconsistencies as one signal (not a sole determinant) in phishing-awareness training, alongside the stronger technical indicators above.

---

## 🛡️ Recommended Mitigation & Action Steps

1. **Verify Senders:** Never rely on display names alone — inspect the exact domain string for deceptive character substitutions (e.g. `0` for `o`).
2. **Inspect Before Interacting:** Always hover over hyperlinks to check the destination domain and confirm the protocol is `https://` before clicking.
3. **Report Suspicious Activity:** Do not reply to the sender or enter any credentials. Forward flagged messages directly to the internal IT security mailbox for isolation and analysis.
4. **Block at Source:** Add the sender domain and malicious link to the organization's email/DNS blocklists to prevent further delivery.

---

## 🛠️ Summary Action Roadmap

1. **Immediate:** Block the sender domain (`micros0ft-support.com`) and malicious link domain at the email gateway and DNS level; report the sample to internal security and external threat intelligence feeds.
2. **Short Term:** Notify any recipients who may have received the email and confirm none have submitted credentials to the phishing site.
3. **Ongoing:** Reinforce phishing-awareness training focused on sender-domain inspection, HTTPS verification, and recognizing urgency-based social engineering tactics.

├── README.md               <-- This documentation file
└── screenshots/
    └── get_request.png     <-- Screenshot of your Postman workspace showing the JSON response

# API Security Risk Analysis: postman-echo.com

**Date of Assessment:** *[insert date]*  
**Tools Used:** Postman  
**Classification:** 🟡 **Low–Medium Risk**

---

## 📋 Executive Summary
An API security review was conducted against a public test endpoint (`postman-echo.com`) using Postman to evaluate authentication controls and information disclosure risks. The endpoint was found to be fully accessible without authentication and to leak internal client runtime version information in its response headers — issues that, on a production API, would materially aid an attacker's reconnaissance and increase the risk of abuse by anonymous third parties.

---

## 🚨 Detailed Findings & Risk Breakdown

<img width="1470" height="739" alt="Postman GET request to postman-echo.com showing 200 OK JSON response and exposed headers" src="https://github.com/user-attachments/assets/c0d9577c-b38d-43cc-b9a2-a236dd623f03" />

*Figure 1.1: Automated HTTP GET request evaluation targeting the postman-echo.com server. The request returns a successful 200 OK response with a raw JSON payload, while the response also reveals architectural anomalies including public unauthenticated data routing and client software framework exposure.*

### 1. Unauthenticated Resource Endpoint with Client Software Reflection
* **What is the issue?** The endpoint accepts and processes GET requests with no authorization check of any kind, and its response headers reflect internal client runtime version information (`PostmanRuntime/7.56.1`).
* **Why does it matter?** Allowing arbitrary third parties to query the endpoint anonymously removes any control over who can access or abuse it. Exposing internal runtime versions in response headers hands reconnaissance data to a potential attacker, helping them map out developer environments and identify likely attack surfaces before attempting further exploitation.
* **Risk Level:** 🟡 **Low to Medium** *(severity is context-dependent — rises significantly if the endpoint sits in front of production data or business logic rather than a public test service)*
* **Remediation:** Implement authorization verification (e.g. OAuth2 or JWT validation) before processing incoming HTTP requests. Strip or sanitize descriptive internal client/server headers so they are not echoed back in responses.

---

## 🛠️ Summary Action Roadmap

1. **Immediate:** Confirm whether this endpoint pattern exists on any production API surfaces; if so, restrict public access immediately pending an authentication fix.
2. **Short Term:** Implement OAuth2 or JWT-based request validation on all endpoints intended for authenticated use only.
3. **Ongoing:** Review server and framework configuration to strip version-revealing headers (e.g. `X-Powered-By`, runtime identifiers) from all outbound responses.
