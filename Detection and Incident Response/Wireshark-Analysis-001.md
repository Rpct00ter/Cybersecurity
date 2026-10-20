# Security Investigation - [Investigation Name]

## Source
Let's Defend - VoIP room

## Scenario
Your close friend James recently received a suspicious phone call from someone claiming to be his bank. The caller asked for sensitive information, making James uneasy. Suspecting a potential Vishing (Voice Phishing) attack, you decide to investigate by capturing and analyzing the VoIP traffic.

## Tasks
* How many RTP packets were in the traffic?
* When did the fake call with James start?
* What is the Jame's phone number?
* How long was the call with the bank?
* What is the phone number of the bank that James received a call from?
* What is the name of the bank calling?
* What is James's Social Number?

## Tools & Technologies
* Wireshark


     


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
