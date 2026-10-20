---

> ### Detection & Incident Response
> * This directory contains hands-on security investigations covering different security incidents, alerts, and defensive scenarios.
> * Files inside this directory follow the naming rule below (Name of Incident or SIEM used in detection):
>
> ```text
> └── Investigations/
>     ├── Phishing-001/
>     ├── Brute-Force-001/
>     ├── Suspicious-PowerShell-001/
>     ├── Splunk-Analysis-001/
>     └── Network-Intrusion-001/
> ```
> * Each and every investigation in this directory should follows the structure similar to the one that is shown below:
<h2 align="center">
🦊 🦊 🦊
</h2>

<br>


# Security Investigation - [Investigation Name]

## Source
(Source of the challenge.)

## Scenario
(Description of the initial situation.)

## Tasks
* Task 1
* Task 2
* Task 3

## Tools & Technologies
* [SIEM / Monitoring Tool]
* [EDR / Security Tool]
* [Network Analysis Tool]
* [Threat Intelligence]
* [Other relevant tools]
----


     


# Investigation

### 1. Initial Alert / Detection

(Describe the initial alert and the available evidence. + some screenshots)

### 2. Triage

(Explain the first steps taken to determine whether the alert is a true or false positive. + some screenshots)

### 3. Evidence Analysis

(Document relevant logs, processes, network connections, files, users, IPs, domains, hashes, etc. + some screenshots)

### 4. Correlation

(Build the incident timeline. + some screenshots)

### 5. Threat Intelligence

(Analyze relevant IOCs using available threat intelligence sources. + some screenshots)

### 6. MITRE ATT&CK

| Technique   | ID      | Evidence   |
| ----------- | ------- | ---------- |
| [Technique] | [TXXXX] | [Evidence] |

## Findings

(Quick summary)

**Verdict:** True Positive / False Positive / Inconclusive

**Severity:** Informational / Low / Medium / High / Critical

**Affected Assets:** [Systems/users/assets]

## Response & Recommendations

* [Action 1]
* [Action 2]
* [Action 3]
