# Security Audit Report: Payment Service

**Auditor:** Ralesy  
**Audit Date:** April 21, 2026  
**Objective:** Evaluation of the payment-service security. The analysis covers source code, configurations, and dependencies to identify potential vulnerabilities.  
**Methodology:** Static Analysis Security Testing (SAST), configuration review (Dockerfile, Pipfile), dependency analysis, and vulnerability scanning using Bandit.  
**Note:** This report focuses on the payment-service as a standalone component. In the full project architecture (including gateway and other microservices), some risks may be mitigated (e.g., authentication at the gateway level). It is recommended to consider this during integration.

## Key Vulnerabilities

| Risk Level | Vulnerability | Description | Consequences | Recommendations |
| :--- | :--- | :--- | :--- | :--- |
| **Critical** | Missing Webhook Signature Verification | The `/webhook/yoomoney/confirm_payment` endpoint in `app.py` processes data without validating the digital signature from YooMoney. | Payment forgery, unauthorized data modification. | Implement HMAC verification according to YooMoney documentation. |
| **High** | Sensitive Data Logging | Personal data such as `user_id`, `amount`, and `metadata` are logged in plain text within `app.py`. | PII (Personally Identifiable Information) leak. | Remove or mask sensitive data in logs. |
| **High** | Missing Authentication (IDOR) | Endpoints rely on the `Header('User')` without proper verification or token validation. | Unauthorized access to user data and services. | Implement JWT validation. |
| **Medium** | Permissive CORS Policy | `app.py` defines `allow_origins = ['*']`. | Potential CSRF and Cross-Origin attacks. | Restrict allowed origins to specific domains. |
| **Medium** | Unpinned Dependency Versions | `Pipfile` uses wildcard versions (`*`). | Introduction of vulnerable updates (Supply Chain risk). | Pin specific versions and scan for CVEs. |
| **Medium** | Binding to All Interfaces (Bandit B104) | `main.py` uses `host = '0.0.0.0'`, binding the server to all network interfaces. | Unauthorized access in the absence of a firewall. | Change host to `127.0.0.1` or use a reverse proxy/firewall. |
| **Low** | Absence of Rate Limiting | No protection against automated requests or DDoS. | Service exhaustion or abuse. | Implement a rate limiter. |

## Compensating Controls from Related Services

In the full project architecture, some risks may be mitigated by other components:
* **API Gateway:** Can handle JWT verification for internal requests and inject the `User` header. This partially compensates for the lack of auth in `/initiate-payment` and `/subscriptions/valid`. *Recommendation: Ensure strict gateway-level protection.*
* **General Infrastructure:** Supabase provides data encryption at rest; other infrastructure layers can provide rate limiting or monitoring.
* **External Services:** YooMoney webhooks are sent directly to the service—mitigation is impossible without code-level changes.

## Conclusion

In isolation, this service requires security hardening. When integrated with other services, overall risks are reduced. Critical vulnerabilities must be addressed before production deployment.

---
*Note: A secure version with fixed vulnerabilities will be released soon. In the meantime, please consider the findings above.*
