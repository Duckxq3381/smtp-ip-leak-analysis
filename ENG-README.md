---

# Backend IP Leak Analysis via SMTP Configuration Misconfiguration

This case study analyzes how a backend server's real IP address (Origin) can be exposed (Information Disclosure) due to an email server (SMTP) misconfiguration, despite having a CDN/WAF (e.g., Cloudflare) protection layer in front of the web application.

## 1. Discovery and Leak Scenario
When examining the raw header logs (Email Headers) of an automated email (such as password reset, account activation, etc.) sent from the web application to a user, it was observed that the intermediary SMTP service recorded the IP address of the initial station from which it received the email.

Example leaked header line (Anonymized):
`Received: from server_name [46.4.X.X] (Authenticated sender: no-reply@targetsite.com) by smtp-service.com (Exim);`

This leak allows potential attackers to entirely bypass the CDN protection layer, directly target the backend server's IP address (Origin), perform port scanning, or launch direct DDoS attacks.

## 2. Verification
When the leaked IP address is requested directly through a web browser, the underlying web server (e.g., OpenResty/Nginx) responds directly and throws a `403 Forbidden` error. This confirms that the IP address belongs directly to an exposed backend server rather than a protected CDN edge.

## 3. Mitigation and Fix
To prevent this type of information disclosure, the following steps should be implemented:
1. **API-Based Email Delivery:** Emails should be sent via the service provider's Web APIs (HTTP POST) instead of standard SMTP Relay. This prevents the backend IP from being injected into the header history.
2. **IP Whitelisting:** The origin server should be configured to accept incoming traffic exclusively from trusted CDN IP ranges, dropping all other direct external connections at the firewall level.
