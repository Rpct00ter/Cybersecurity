---

> ### Security Investigations
> This directory contains hands-on security investigations covering different security incidents, alerts, and defensive scenarios. Each and every investigation in this directory follows the structure below:
<h2 align="center">
🦊 🦊 🦊
</h2>

<br>

----

# Security Investigation - [Investigation Name]

## Source
(Source of the challenge.)

## Description
(Brief description of the security incident.)

## Scenario
(Description of the initial situation and how the investigation was triggered.)

## Objectives
* Identify the cause of the alert/incident
* Determine whether the activity is malicious
* Identify affected users, systems, and assets
* Determine the scope and impact
* Recommend appropriate response actions

## Tools & Technologies
* [SIEM / Monitoring Tool]
* [EDR / Security Tool]
* [Network Analysis Tool]
* [Threat Intelligence]
* [Other relevant tools]

## Investigation

### 1. Initial Alert / Detection

(Describe the initial alert and the available evidence. + some screenshots)

### 2. Triage

(Explain the first steps taken to determine whether the alert is a true or false positive. + some screenshots)

### 3. Evidence Analysis

(Document relevant logs, processes, network connections, files, users, IPs, domains, hashes, etc. + some screenshots)

### 4. Correlation

(Correlate evidence from different sources and build the incident timeline. + some screenshots)

### 5. Threat Intelligence

(Analyze relevant IOCs using available threat intelligence sources. + some screenshots)

### 6. MITRE ATT&CK

| Technique   | ID      | Evidence   |
| ----------- | ------- | ---------- |
| [Technique] | [TXXXX] | [Evidence] |

## Findings

(Summarize what was discovered during the investigation.)

**Verdict:** True Positive / False Positive / Inconclusive

**Severity:** Informational / Low / Medium / High / Critical

**Affected Assets:** [Systems/users/assets]

## Response & Recommendations
(Describe the recommended containment, remediation, and follow-up actions.)

* [Action 1]
* [Action 2]
* [Action 3]

## Lessons Learned
(Briefly describe what the investigation demonstrated and what could be improved or monitored in the future.)

----
> ### Files inside this directory follow the naming rule below (Name of Incident or SIEM used in detection):
>
> ```text
> └── Investigations/
>     ├── Phishing-001/
>     ├── Brute-Force-001/
>     ├── Suspicious-PowerShell-001/
>     ├── Splunk-Investigation-001/
>     └── Network-Intrusion-001/
> ```
---- 
