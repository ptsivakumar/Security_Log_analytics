### Security Incident Report

**Incident Title:** Unauthorized Data Access and Exposure
**Date of Incident:** August 10, 2025
**Incident Status:** Contained, Under Investigation
**Date of Report:** August 10, 2025
**Reported to MAS:** August 10, 2025

---

### **1. Executive Summary**

On August 10, 2025, a security incident was identified where a user, "User A," retrieved sensitive data belonging to "User B." The incident was caused by an **authorization logic flaw** in a microservice. The **Web Application Firewall (WAF)** and **API Gateway** performed their intended functions, but the flaw in the application's business logic allowed the unauthorized query to proceed. A hotfix has been applied, containing the breach. The affected data includes personal and financial information.

---

### **2. Timeline of Events**

All timestamps are in SGT (GMT+8). The timeline was constructed by correlating logs from the API Gateway, WAF, Application Microservice, and Database Audit logs.

| Timestamp | Component | Log Entry | Correlation |
| :--- | :--- | :--- | :--- |
| **10:00:15 AM** | **API Gateway** | `INFO: Request ID: 67890. User ID: User A. IP: 203.0.113.10. Endpoint: /api/accounts/UserB/details.` | User A successfully authenticates and sends a request for User B's data. The API Gateway forwards the request. |
| **10:00:15 AM** | **WAF (Cloud)** | `INFO: Request ID: 67890. User ID: User A. Rule: SQLi/XSS check. Status: PASS.` | The WAF inspects the request payload for known threats and allows it to pass, as no malicious code was detected. |
| **10:00:16 AM** | **Application Microservice** | `DEBUG: Request ID: 67890. Authenticated user: User A. Attempting to retrieve data for account ID: User B.` | The application receives the request and confirms User A's identity via their session token. |
| **10:00:16 AM** | **Application Microservice** | `ERROR: Request ID: 67890. Authorization check failed. User A does not have access to User B's account. Fallback to default logic.` | **This log entry is the first indicator of the flaw.** A specific authorization check failed, but the application's faulty logic allowed the request to proceed. |
| **10:00:17 AM** | **Database Audit Log** | `QUERY: Request ID: 67890. User: app_service_user. SELECT * FROM accounts WHERE account_id = 'UserB'.` | The application executes a query on the database to retrieve User B's data, confirming the request was not blocked. |
| **10:00:18 AM** | **Application Microservice** | `INFO: Request ID: 67890. Data retrieved for account ID: User B. Sending 1 record to User A.` | The microservice logs the successful retrieval and transmission of User B's data to User A. This is the **data exposure event**. |
| **10:05:00 AM** | **User Activity Log** | User B reports via a secure channel that their account details are visible to another user. | The incident is formally reported, triggering the investigation. |
| **10:15:00 AM** | **CI/CD Pipeline** | `ACTION: A security engineer deployed a hotfix to microservice-accounts. Fix applied to production.` | The engineering team pushed a hotfix to correct the authorization logic flaw. |
| **10:15:30 AM** | **Application Microservice** | `INFO: Request ID: 99999. User ID: User A. Attempting to retrieve data for User B. Authorization check successful: DENIED.` | A test request confirms the hotfix is working as intended. The request is now correctly denied. |

---

### **3. Root Cause Analysis**

The root cause was an **authorization logic flaw** within the `microservice-accounts` component. A specific authorization rule intended to deny cross-user data retrieval failed due to a conditional logic error. Instead of returning an "unauthorized" response, the code defaulted to a "success" path, allowing the data to be retrieved. This flaw was not caught during code review or automated security testing. The **WAF and API Gateway performed their intended functions** by checking for payload-based attacks and authenticating the user, respectively, but they are not designed to enforce granular business-level authorization rules.

---

### **4. Impact Assessment**

- **Affected Parties:** User B.
- **Data Exposed:** Personal identifiable information (PII) and financial details associated with User B's account.
- **Breach Status:** The incident was a one-time occurrence with a single user's data. The risk of widespread data exposure has been mitigated by the hotfix.

---

### **5. Immediate Actions Taken**

1. A **hotfix** was deployed to the vulnerable microservice to correct the authorization logic.
2. The **CI/CD pipeline** was updated with new security checks to prevent similar flaws from being deployed in the future.
3. The affected user (User B) has been notified of the incident and provided with guidance on securing their account.
4. All relevant logs have been preserved for forensic analysis.

---

### **6. Future Remediation**

1. **Code Review Enhancement:** Implement stricter code review policies, specifically for authorization and authentication logic.
2. **Automated Testing:** Implement more robust automated security testing in the CI/CD pipeline, including integration tests that specifically target authorization checks.
3. **Threat Modeling:** Conduct a formal threat modeling exercise on all critical microservices to identify and mitigate similar vulnerabilities.
4. **SIEM Alerting:** Create a new SIEM correlation rule to alert the security team whenever an "authorization check failed" log entry is immediately followed by a "data retrieved" log entry, which would have flagged this incident in real-time.
