# Security-Assessment-Portfolio-testfire.net
Web Security Assessment: Manual Triage & Vulnerability Research
Executive Summary

This project demonstrates a comprehensive security assessment of two distinct environments: a legacy financial application (Altoro Mutual) and a modern cloud-based telemetry service (Mozilla Services). By integrating automated DAST (Dynamic Application Security Testing) with manual verification, I identified critical vulnerabilities that allow for full account takeover and data manipulation.

Methodology
    Scanning: Utilized OWASP ZAP 2.17.0 for baseline automated discovery.
    Verification: Performed manual exploitation using browser dev tools and payload injection to confirm findings.
    Triage: Conducted deep-dive analysis into "High" severity alerts to filter out false positives and prioritize remediation for Lead Architects.

I. Critical Finding: SQL Injection & Vertical Privilege Escalation
    Status: Verified.
    Vulnerability: SQL Injection (CWE-89).
    Endpoint: http://altoro.testfire.net/doLogin.
    Discovery: ZAP identified potential SQLi via boolean manipulation on the login form.
    Manual Exploitation: Successfully bypassed authentication using the tautology payload 'OR 1=1--.
    Impact: This resulted in Vertical Privilege Escalation, granting full access to the "Admin User" account and its associated financial data.
    Remediation: Transition to Parameterized Queries (Prepared Statements) to ensure the database treats input as data, not executable code.

II. Manual Triage: The "Mozilla PII" False Positive
One of the most critical responsibilities of a security tester is preventing "alert fatigue" for development teams.
    ZAP Alert: High-severity PII Disclosure (Credit Card numbers).
    Analysis: Automated scans flagged 5 instances of PII on Mozilla Remote Settings endpoints.
    Triage Process: I manually inspected the JSON payload (e.g., https://firefox.settings.services.mozilla.com/v1/buckets/main/collections/crash-reports-ondemand/changeset).
    Finding: The "credit card" strings identified by ZAP were actually Unix timestamps (e.g., 1776428290695) and SHA-256 binary hashes.
    Decision: Classified as a False Positive. De-escalating this saved architect and developer resources that would have been wasted on non-existent risks.

III. Verified Web Vulnerabilities
Vulnerability	Status	Proof of Concept (POC)	Impact
a)Reflected XSS	
Verified
Injected <script>alert(1)</script> into the query parameter on search.jsp.	
Session hijacking and credential theft.

b)CSRF	
Verified
Confirmed on feedback and login pages by successfully delivering payloads to hidden iframes.	
Unauthorized state changes on behalf of authenticated users.

c)Clickjacking	
Verified
Successfully framed the application within a malicious clickjack.html file.	
UI redressing to trick users into performing sensitive actions.

IV. Systemic Architectural Deficiencies
The following issues were identified as systemic, requiring global configuration changes rather than individual bug fixes:
    Server Version Disclosure: The application leaks Apache Tomcat/7.0.92 in error responses (HTTP 405), facilitating targeted exploitation.
    Missing Content Security Policy (CSP): The lack of a CSP header leaves the application vulnerable to script injection even if input validation is bypassed.
    Missing Anti-clickjacking Headers: Absence of X-Frame-Options or frame-ancestors across all endpoints.

V.Mitigation
SQL Injection: Use Parameterized Queries (Prepared Statements) to neutralize injection vectors.
Reflected XSS: Apply Context-Aware Output Encoding to all user-supplied data.
CSRF: Implement unique Anti-CSRF Tokens for every state-changing request.
Clickjacking: Enforce the X-Frame-Options: SAMEORIGIN header to prevent UI redressing.
Information Disclosure: Suppress Server Banners (e.g., Tomcat version) in global configurations.
Missing CSP: Deploy a global Content-Security-Policy header to provide defense-in-depth.
