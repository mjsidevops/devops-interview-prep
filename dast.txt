1. What is OWASP ZAP?

OWASP ZAP (Zed Attack Proxy) is an open-source DAST tool used to identify security vulnerabilities in a running web application or API. Unlike SAST tools such as SonarQube, which analyze source code, ZAP tests the application from an external/attacker perspective.

2. How does ZAP work?

We provide ZAP with the application's URL, for example https://staging.myapp.com. ZAP crawls the application using its spider/crawler to discover available URLs and endpoints. It then analyzes the HTTP requests and responses and looks for vulnerabilities such as XSS, insecure headers, cookies, authentication issues, and other web security vulnerabilities. For authenticated applications, ZAP can be configured with authentication so it can scan protected URLs as well.

For APIs, we can provide an OpenAPI/Swagger specification so ZAP knows the API endpoints.

3. How is it integrated into CI/CD?

In the CI/CD pipeline, we first build and deploy the application to a DEV or staging environment. Then we run OWASP ZAP as a Docker container from the GitLab CI runner. We provide the staging application's URL to ZAP and run a baseline or full scan. ZAP generates a security report, and we configure a security threshold—for example, critical or high vulnerabilities can fail the pipeline. If the scan passes, the application can proceed toward production.

The flow is:

GitLab
   │
   ▼
Build & Test
   │
   ▼
SAST / SCA
   │
   ▼
Deploy to Staging
   │
   ▼
OWASP ZAP
   │
   ├── Crawl Application
   ├── Security Scan
   └── Generate Report
           │
           ▼
      Security Gate
        /       \
     PASS       FAIL
      │           │
      ▼           ▼
 Production    Stop Pipeline
One-line answer to remember

"OWASP ZAP is a DAST tool that scans a running application for security vulnerabilities. In CI/CD, we deploy the application to a staging environment, run ZAP as a Docker container against the staging URL, generate a security report, and use the scan result as a security gate before production.

Passive Scan: ZAP observes and analyzes normal HTTP requests/responses without sending attack payloads.
Example: detecting missing security headers, insecure cookies, information disclosure.
Active Scan: ZAP actively sends attack-like requests/payloads to try to find exploitable vulnerabilities.
Example: testing for SQL Injection, XSS, path traversal, etc.
Baseline Scan vs Full Scan
Baseline Scan (zap-baseline.py): A quick, lightweight scan mainly involving crawling + passive analysis. Suitable for frequent CI/CD pipeline runs.
Full Scan (zap-full-scan.py): A deeper scan that includes crawling + passive scanning + active scanning. Takes longer and should generally run against a controlled DEV/QA/staging environment.

Easy way to remember:

Passive = Observe
Active  = Attack


Baseline = Quick / Passive-focused
Full     = Deep / Active + Passive
