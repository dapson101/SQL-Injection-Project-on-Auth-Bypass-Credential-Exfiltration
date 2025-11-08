SQL Injection Project on Auth Bypass & Credential Exfiltration 
Report  
Target: Intentionally vulnerable login endpoint for training (https://issauga.lt/login-1/). 
Objective: Demonstrate authentication bypass via SQL Injection and exfiltrate stored credential records 
under explicit authorization. 
1. Executive Summary 
A critical SQL Injection vulnerability was identified in the login functionality. An attacker can bypass 
authentication and retrieve stored user credentials. The risk is Critical due to the impact (account 
compromise, privacy breach) and low exploitation complexity. 
Business Impact: Unauthorized access to user accounts and disclosure of sensitive information, 
potential regulatory exposure, and reputational damage. 
2. Scope & Rules of Engagement (RoE) 
● In-Scope: The training instance login endpoint and underlying DB reachable through the app. 
● Out-of-Scope: Third-party services, production systems, unrelated subdomains. 
● Allowed Actions: Non-destructive SQLi validation, schema enumeration, and sanitized 
credential dump for proof-of-concept. No service disruption. 
3. Methodology 
● Standards Referenced: OWASP Testing Guide (OTG-INPVAL), OWASP Top 10 A03:2021 – 
Injection, CWE-89. 
● Approach: 
1. Recon: Enumerate parameters at login; capture baseline request with Burp Proxy. 
2. Manual SQLi: Payload testing via Burp Repeater/Intruder using Boolean-based and 
error-based techniques, plus Kali wordlists (SQL.txt). 
3. Automated SQLi: Validate and enumerate with sqlmap, progressively increasing 
risk/level as permitted. 
4. Evidence: Collect requests/responses, DB enumeration outputs, and sanitized data. 
4. Environment & Tools 
● Browser proxied through Burp Suite  
● Kali Linux with sqlmap installed. 
● Wordlists (\usr\share\wfuzz\wordlist\Injections\SQL.txt). 
5. Step-by-Step Procedures  
5.1 Manual SQLi via Burp (Boolean/Error-Based) 
Goal: Identify injectable parameter(s) and validate auth bypass. 
● In Repeater, test payloads in username and/or password fields. Start safe and observe 
responses: 
' OR '1'='1 
' OR 1=1-- - 
') OR ('1'='1'-- - 
admin'-- - 
● Use Intruder with a simple payload list to fuzz systematically: 
○ Positions: select value of username (or password). 
○ Payload list: from Kali wordlists (e.g., SQL.txt). 
○ Grep-Extract/Grep-Match: look for markers of success (e.g., Location: /dashboard, 
presence of a user profile element, or longer content length). 
Kali Linux SQL Injection Scripts: (\usr\share\wfuzz\wordlist\Injections\SQL.txt) 
Auth_bypass (Successful login/redirect) 
Auth Bypass Check: If setting username=' OR '1'='1 (with appropriate comment) changes the response 
to a dashboard or a different status code (e.g., 302 to /home), the parameter is likely injectable. 
5.2 Automated Validation & Enumeration with sqlmap 
Capture Baseline Login Request 
1. Configure browser to use Burp Proxy (127.0.0.1:8080) 
2. Navigate to the test login page (https://issauga.lt/login-1/). 
3. Submit a benign login (e.g., user=ali, password=ali). 
4. In Burp HTTP history, right-click the POST request → Send to Repeater and Save item 
 
 
Prepare request file: In Burp, Save item of the vulnerable POST request (with benign values) to 
packet.txt. 
 
 
Detection (conservative): Using sqlmap to check if the parameters are injectable. 
sqlmap -r packet.txt -p username 
● Confirms injection point and lists databases. 
Sqlmap detection (identified injectable parameters) 
Dump Users Table (authorized + sanitize before storing): 
sqlmap -r packet.txt -p passwd –dump 
5.4 Post-Exploitation Evidence 
Create a sanitized CSV users_sanitized.csv  
6. Conclusion 
The login endpoint of the training instance is intentionally vulnerable to SQL Injection, enabling auth 
bypass and data access. The report demonstrates the attack chain and emphasizes actionable 
remediations aligned with OWASP guidance
