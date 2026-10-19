
# Security Investigation - [Investigation Name]

## Source
TryHackMe: The O365 Intrusion Challenge
## Scenario
You have been brought on at Security Services THM Security Service (TSS). Vantage Dynamics' IT team escalated a report from their Accounts Payable department on 27 August. An employee, Marcus Webb, noticed a colleague referencing a document tied to his account that he had no memory of sending. Around the same time, a second employee's account, David Chen, began showing signs of unusual activity of its own. Vantage has not made any changes to either account and has handed over sign-in and activity logs from the day of the incident. TSS has been engaged to determine how the intrusion began, how it spread between accounts, and whether any sensitive data left the organization.  

Mail content from Lead Responder Analyst:   
"Start with Marcus Webb's sign-in history for the day. I pulled a few details from the company beforehand, and it looks like every employee is issued a company managed MacBook as standard.
Marcus is not the only one on that team who messages colleagues and shares files constantly, so do not assume every message or share you find is part of the incident. Check whether it fits the account's usual behavior first.
Once you have the entry point, I want the persistence mechanism next, then trace whether this reached anyone else.

01 Establish how Marcus Webb's account was initially compromised.  
02 Determine what the attacker did immediately after gaining access.  
03 Identify the persistence mechanism left on the mailbox.  
04 Trace how the attacker moved from Marcus Webb's account to a second identity.  
05 Confirm what sensitive data was accessed and whether it left the organization."  
>**Note**: Apply index=vantage with the All time filter and examine the evidence to answer the tasks.

## Tasks
* What is the first timestamp at which a non-macOS device signed into Marcus Webb's account?
* Just after logging in, the attacker accessed a large number of emails within the first few minutes of the session. How many emails were accessed during this window?
* What subject line does the attacker's inbox rule match on?
* How much time elapsed between the malicious inbox rule being created and a matching reply email arriving in the mailbox?
* Marcus Webb's account logged a Teams message that had a link attached to it. What file did this link reference?
* What operating system was used in the suspicious sign-in to David Chen's account?
* What IP address was used in this same sign-in?
* How much time passed between this message being created and the suspicious sign-in to David Chen's account?
* The file the attacker downloaded from David Chen's account was legitimately modified earlier that day. At what time?
* What file stored on the Leadership site had a sensitivity label applied to it by the organization's data governance, prior to the incident?
* What file did the attacker create an external sharing link for?
* What external email address was granted access to this file?
* According to the emails, what is the subject line of the notification sent to this external address by the sharing application?

## Tools & Technologies
* Splunk
----


     


# Investigation

### 1. Initial Alert / Detection

(Describe the initial alert and the available evidence. + some screenshots)

### 2. Triage

(Explain the first steps taken to determine whether the alert is a true or false positive. + some screenshots)

### 3. Evidence Analysis
<img width="1918" height="965" alt="image" src="https://github.com/user-attachments/assets/90f5a569-4f97-46d1-9a61-479ed6e247ff" />

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
