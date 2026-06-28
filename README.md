# AI-Powered Cybersecurity Incident Response System

A smart cybersecurity monitoring and automated incident response platform that combines ServiceNow SecOps workflows, Gemini AI threat intelligence, and VirusTotal IP reputation analysis to detect, classify, and respond to security threats automatically — eliminating manual SOC triage for common attack patterns.

---

## Problem Statement

Security Operations Center analysts are flooded with hundreds of alerts daily. Manual triage is slow, inconsistent, and leaves critical threats unaddressed for too long. This platform solves that by automating the entire threat classification and assignment pipeline — from alert ingestion to SOC team routing — using AI and real threat intelligence data.

---

## Project Overview

When a security alert enters the system:

1. Alert is logged as a Security Incident in the custom ServiceNow table
2. Business Rule fires automatically after record creation
3. Threat data is sent to the FastAPI AI backend
4. Gemini AI performs deep threat analysis
5. VirusTotal API checks source IP against global threat databases
6. Combined intelligence updates the Security Incident record automatically
7. Critical threats auto-escalate to Investigating status
8. SOC team receives immediate email notification
9. Security dashboard reflects live threat posture

---

## Architecture

```
Security Alert Ingested
        |
ServiceNow Custom Table (u_security_incident)
        |
After-Insert Business Rule triggers
        |
REST API Call --> FastAPI Server (localhost / deployed)
        |
        |-- Gemini 2.5 Flash API
        |       Analyzes threat description
        |       Returns: type, severity, team, analysis, response steps
        |
        |-- VirusTotal API
                Checks source IP reputation
                Returns: malicious reports count, suspicious count
        |
JSON response merged and returned
        |
ServiceNow updates Security Incident fields:
- Threat Type
- Severity
- Assigned Group
- AI Analysis
- Recommended Action
- VirusTotal Result
        |
Critical Threat Escalation Rule fires
- Status auto-changed to Investigating
        |
Email Notification sent to SOC Team
        |
SOC Security Dashboard updated
```

---

## Technology Stack

| Technology | Purpose |
|---|---|
| ServiceNow PDI | Custom table, workflows, business rules, dashboard |
| FastAPI | Python REST API backend |
| Gemini 2.5 Flash | AI-powered threat classification and analysis |
| VirusTotal API | Real-time IP reputation and threat intelligence |
| Python 3.14 | Backend development language |
| google-genai | Official Gemini SDK |
| python-dotenv | Secure environment variable management |
| requests | VirusTotal HTTP calls |
| ngrok | Secure tunnel for local development and testing |
| GitHub | Version control and project documentation |

---

## Custom Table Design

Table name: `u_security_incident`

| Field Label | Field Name | Type | Purpose |
|---|---|---|---|
| Threat ID | u_threat_id | String | Unique alert identifier (THR-001) |
| Threat Type | u_threat_type | String | Attack category |
| Source IP | u_source_ip | String | Attacker IP address |
| Severity | u_severity | Choice | Critical / High / Medium / Low |
| AI Analysis | u_ai_analysis | String | Gemini technical threat analysis |
| Recommended Action | u_recommended_action | String | AI-generated response playbook |
| Assigned Group | u_assigned_group | Reference | SOC team assignment |
| Status | u_status | Choice | Open / Investigating / Contained / Resolved |
| Threat Description | u_threat_description | String | Full alert description |
| VirusTotal Result | u_virustotal_result | String | IP reputation data |

---

## Features

### AI Threat Classification
Gemini 2.5 Flash analyzes the threat description and returns a structured classification covering attack type, severity level, responsible SOC team, detailed technical analysis, and specific incident response steps tailored to the threat.

### VirusTotal IP Reputation
Every security incident triggers a real-time check of the source IP against VirusTotal's database of 70+ security vendor reports. The result — including malicious and suspicious report counts — is stored directly on the incident record, giving analysts immediate threat context.

### Auto-Escalation Workflow
A dedicated Business Rule monitors severity levels. When a Critical threat is detected, the status automatically advances from Open to Investigating, ensuring no critical alert sits unacknowledged in the queue.

### SOC Team Auto-Assignment
Based on AI analysis, incidents are automatically routed to the most appropriate team:

| Threat Type | Assigned Team |
|---|---|
| Brute Force, Phishing, Unknown | SOC Team |
| Ransomware, Data Breach, Insider Threat | Threat Response Team |
| Malware, Endpoint compromise | Endpoint Security |
| DDoS, Network intrusion | Network Security |

### Email Notifications
Critical severity incidents trigger immediate email alerts to the assigned SOC team with full threat details, AI analysis, and recommended response steps embedded in the notification.

### SOC Security Dashboard
Three live reports visible on the SOC dashboard:
- Threats by Severity — pie chart showing risk distribution
- Threats by Type — bar chart of attack categories
- Threats by Assignment Group — SOC team workload visibility

---

## Project Structure

```
ai-cybersecurity-project/
|
|-- backend/
|   |-- main.py                   # FastAPI server with /analyze-threat endpoint
|   |-- threat_classifier.py      # Gemini AI + VirusTotal integration logic
|   |-- requirements.txt          # Python dependencies
|   `-- .env                      # API keys (not committed to GitHub)
|
|-- screenshots/                  # ServiceNow UI screenshots
|
`-- README.md
```

---

## Setup Guide

### Prerequisites
- Python 3.10 or above
- ServiceNow Personal Developer Instance (PDI)
- Gemini API key — free from aistudio.google.com
- VirusTotal API key — free from virustotal.com
- ngrok — free from ngrok.com (for connecting local backend to ServiceNow)

### 1. Clone the Repository

```bash
git clone "repo link"
cd AI-Cybersecurity-Incident-Response/backend
```

### 2. Create Virtual Environment

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# Linux / Mac
source venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

Create a `.env` file inside the backend folder:

```
GEMINI_API_KEY=your_gemini_api_key_here
VIRUSTOTAL_API_KEY=your_virustotal_api_key_here
```

### 5. Run the Backend Server

```bash
uvicorn main:app --reload
```

Server starts at: `http://127.0.0.1:8000`

Interactive API docs: `http://127.0.0.1:8000/docs`

### 6. Expose Local Server to ServiceNow via ngrok

ServiceNow is a cloud platform and cannot reach localhost directly. Use ngrok to create a secure public tunnel:

```bash
ngrok http 8000
```

Copy the generated HTTPS URL (example: `https://abc123.ngrok-free.app`) and update it in the ServiceNow REST Message endpoint as:

```
https://abc123.ngrok-free.app/analyze-threat
```

---

## API Reference

### GET /

Health check endpoint.

**Response:**
```json
{
  "status": "AI Cybersecurity backend is running"
}
```

---

### POST /analyze-threat

Analyzes a security threat using Gemini AI and VirusTotal.

**Request Body:**
```json
{
  "threat_description": "Multiple failed SSH login attempts on production server. Over 500 attempts in 10 minutes targeting root account with systematic password guessing.",
  "source_ip": "45.141.84.120",
  "threat_type": "Brute Force"
}
```

**Response:**
```json
{
  "threat_type": "Brute Force",
  "severity": "Critical",
  "assignment_group": "SOC Team",
  "ai_analysis": "This is a high-volume SSH brute force attack targeting the root account of a production server. The systematic nature of the attempts suggests an automated credential stuffing tool. Immediate containment is required to prevent unauthorized access.",
  "recommended_action": "Block the source IP at the firewall immediately. Disable root SSH login and enforce key-based authentication. Review auth logs for any successful login attempts from this IP and audit all recent root account activity.",
  "virustotal_result": "Malicious reports: 23, Suspicious: 4"
}
```

---

## ServiceNow Configuration

### Business Rules

**AI Threat Analyzer**
- Table: Security Incident (u_security_incident)
- When: After Insert
- Purpose: Calls FastAPI backend, receives AI + VirusTotal analysis, updates all fields on the security incident record using `current.getValue()` for reliable field reading

**Critical Threat Escalation**
- Table: Security Incident (u_security_incident)
- When: After Insert and Update
- Purpose: Detects Critical or High severity threats and automatically advances status from Open to Investigating

### Key Technical Note
Field values are read using `current.getValue('field_name')` instead of `current.field_name.toString()`. This ensures reliable data retrieval regardless of deployment environment, as getValue() reads directly from the database rather than the GlideElement object in memory.

---

## Real Threat Test Results

These are actual results from testing with real-world malicious IPs:

| Source IP | Known Threat | VT Malicious | AI Severity |
|---|---|---|---|
| 185.220.101.45 | Tor exit node | 17 | Critical |
| 45.141.84.120 | Brute force origin | 23 | Critical |
| 203.45.67.89 | Phishing server | 8 | Medium |

---

## AI Classification Examples

| Threat Description | AI Type | Severity | Team |
|---|---|---|---|
| 500 failed SSH logins in 10 minutes | Brute Force | Critical | SOC Team |
| Ransomware encrypting files across network shares | Ransomware | Critical | Threat Response Team |
| Employee clicked phishing link, credentials possibly stolen | Phishing | High | SOC Team |
| Server CPU 100%, traffic spike from multiple IPs | DDoS | High | Network Security |
| Malware binary detected on finance workstation | Malware | High | Endpoint Security |
| Admin login from unrecognized country at 3AM | Suspicious Login | High | SOC Team |

---

## Future Improvements

- GeoIP integration to display attacker country and city on each incident
- Threat heatmap showing global attack origins in real time
- SIEM integration with Wazuh or Splunk for automated alert ingestion
- AI chatbot SOC assistant via ServiceNow Virtual Agent
- Automated malware file hash scanning via VirusTotal Files API
- AbuseIPDB integration for additional IP reputation scoring
- Predictive threat analytics using historical incident patterns
- SOAR playbook integration for fully automated response execution

---

## What I Learned

Building this project gave me hands-on experience with:

- Designing custom tables and schemas in ServiceNow for domain-specific workflows
- Writing server-side JavaScript Business Rules with proper GlideRecord API usage
- Integrating multiple external APIs (Gemini + VirusTotal) in a single backend call
- Understanding the difference between GlideElement object access and getValue() in ServiceNow
- Prompt engineering for cybersecurity-specific AI analysis
- Building SOC-grade dashboards and escalation workflows in an enterprise platform

---

## Resume Description

Developed an AI-powered cybersecurity incident response platform using ServiceNow SecOps, FastAPI, and Gemini 2.5 Flash API for automated threat classification, intelligent SOC team routing, VirusTotal IP reputation enrichment, critical threat escalation workflows, and real-time SOC dashboard monitoring.

---