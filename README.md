# FUTURE_CS_01
Repository for the Cyber Security Internship under the Fellowship Program at Future Interns (August 2026 – September 2026).
Task 1: Vulnerability Assessment Report (demo.testfire.net) vulnerability-assessment-report-demotestfirenet


---

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


