Wazuh + Shuffle SOAR: Automated Failed-Login Alerting
A small SOC automation pipeline that catches Windows failed login attempts with Wazuh and pushes a formatted alert email out through Shuffle SOAR and a self-hosted Postfix relay — no manual triage needed for this alert type.
Overview
A Windows endpoint runs the Wazuh Agent, which forwards security events to a Wazuh Manager. When a failed logon is detected, Wazuh fires a webhook to Shuffle SOAR, which filters on the specific rule ID and triggers an email action. That email is relayed through a containerized Postfix instance to the SOC inbox.
Right now the pipeline is scoped to one detection: Windows Event ID 4625 (failed logon) via Wazuh Rule 60122. It's meant as a working proof of concept for the pattern — swap in a different rule ID and email template, and the same skeleton handles a new alert type.
Architecture
text
Windows Endpoint
       │  Event ID 4625
       ▼
Wazuh Agent
       │
       ▼
Wazuh Manager  ──  Rule 60122
       │
       │  Webhook
       ▼
Shuffle SOAR  ──  Rule ID filter
       │
       ▼
Email Action
       │
       ▼
Postfix SMTP Relay
       │
       ▼
Inbox
Stack
Wazuh (Agent + Manager)
Shuffle SOAR
Docker / Docker Swarm
Postfix SMTP relay (containerized)
Gmail SMTP as the upstream relay
Windows Security Event Logs
Detection
Parameter	Value
Windows Event ID	4625
Wazuh Rule ID	60122
Rule Description	Logon Failure - Unknown user or bad password
Wazuh Rule Level	5
Event Type	Authentication Failure
Workflow
A failed login on the Windows endpoint generates Event ID 4625.
The Wazuh Agent ships the event to the Manager.
Rule 60122 matches it as a logon failure.
Wazuh posts the alert to a Shuffle webhook.
Shuffle filters on the rule ID and passes matches to the Email action.
The Email action builds the notification and hands it to the Postfix container.
Postfix relays it out via Gmail SMTP to the SOC inbox.
What's in the alert email
Alert title, rule ID, severity, timestamp, alert ID
Agent ID, name, and IP
Windows Event ID, computer name, security channel
Target username, logon type, failure reason
Source IP, process name
Screenshots
Wazuh Agent
Windows endpoint connected and reporting to the Wazuh Manager. Show Image
Failed Login Detection
Rule 60122 firing on a failed logon attempt. Show Image
Alert Details
Full authentication context, including Event ID 4625. Show Image
Shuffle Workflow
Webhook in, rule filter, Email action out. Show Image
Delivered Alert
The final notification landing in the inbox. Show Image
Security notes
Gmail App Passwords are stored as Docker Secrets, not in the workflow.
SMTP credentials never appear in source.
No API keys are committed to this repo.
Private IPs and personal email addresses are redacted from screenshots before publishing.
Roadmap
 Cover more Windows security event types
 Threat-intel enrichment (IP reputation lookups)
 Slack / Microsoft Teams notification channel
 Automated response actions (e.g. auto-block suspicious source IPs)
 Additional SOAR workflows for other rule sets
 Basic dashboarding/reporting layer
 Detect suspicious PowerShell activity
 Detect privilege escalation attempts
Author
Raghava Reddy — Cybersecurity / SOC / Security Automation
Linkdin - https://www.linkedin.com/in/raghava-reddy-795b8b310/

