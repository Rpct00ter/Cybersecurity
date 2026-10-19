# Security Investigation - [MicrosoftSentinel-Investigation-001]

## THE ROOM IS INCOMPLETED, BECAUSE OF DEPLOYMENT ISSUES ON AZURE PORTAL. WRITEUP WILL BE COMPLETED AFTER FIX.
## Source
Try Hack Me - "MS Sentinel: Investigate" room

## Scenario
In your company's Microsoft Sentinel environment, previously enabled Analytics rules started to generate incidents. You need to start triaging and investigating the incidents. First, review open incidents and take ownership. Then, investigate the incident. Finally, escalate the incident to SOC LVL 2 for in-depth investigation and threat hunting. You decided to take ownership of the Solorigate incident. You are logged in as: Microsoft Sentinel Contributor, Log Analytics Contributor. 

## Tasks
* What is the MITRE Tactics classification for Solorigate incidents?
* Investigate the incident "Sign-ins from IPs that attempt sign-ins to disabled accounts". What is the IP entity involved in this incident?
* Check out this IP's geolocation. What is the country?
* Which disabled account was targeted by attackers?
* How many login attempts were there for this disabled account?

## Tools & Technologies
* Microsoft Sentinel

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
