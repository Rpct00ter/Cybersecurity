
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
After checking UserAgent there can be clearly seen devices diffrent from Macintosh. As for a loggin event I suspected mainly 2 agents that used Chrome: Windows and Linux. After checking Linux, I didn't find any correlation to Marcus, but there was a bunch of logging alerts for diffrent user: David Chen". After checking Windows I found nice logs. In Operation field I selected "UserLoggedIn"
<img width="595" height="635" alt="image" src="https://github.com/user-attachments/assets/814849a5-063e-418f-ac6b-5ce6a317fcab" />
After some clicking my query looked like this:
index=vantage  sourcetype="o365:management:activity" UserAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" Operation=UserLoggedIn ResultStatus=Success user_id="marcus.webb@vantagedynamics.onmicrosoft.com"
<img width="464" height="865" alt="image" src="https://github.com/user-attachments/assets/9cd587b6-1a42-40bf-9d39-f3ddddf46009" />


* Just after logging in, the attacker accessed a large number of emails within the first few minutes of the session. How many emails were accessed during this window?
After I determined succesfull sign in of the attacker, I searched query "index=vantage Operation=MailItemsAccessed earliest="08/27/2026:12:28:00" UserId="marcus.webb@vantagedynamics.onmicrosoft.com""  and counted OperationCount Value to determin how many MailItems have been accessed:
<img width="1008" height="739" alt="image" src="https://github.com/user-attachments/assets/921f1727-08b4-448f-8ccc-9efa0ca4f410" />

* What subject line does the attacker's inbox rule match on?
After searching "index=vantage earliest="08/27/2026:12:28:00" UserId="marcus.webb@vantagedynamics.onmicrosoft.com" Operation="New-InboxRule" we can see only one record and after analysis it shows value 'invoice'
<img width="1025" height="640" alt="image" src="https://github.com/user-attachments/assets/ff2a2611-4fb0-4bd2-aecf-38cd17c3c54c" />

* How much time elapsed between the malicious inbox rule being created and a matching reply email arriving in the mailbox?
To see incoming messages my solution was to change the sourcetype in my query: index=vantage earliest="08/27/2026:12:28:00" sourcetype="o365:reporting:messagetrace"
<img width="550" height="138" alt="image" src="https://github.com/user-attachments/assets/fbaa51de-0a88-4634-8ac0-c37d6a827589" />
<img width="998" height="881" alt="image" src="https://github.com/user-attachments/assets/b285858d-3cfd-4a03-8383-1194f0dcb366" />

* Marcus Webb's account logged a Teams message that had a link attached to it. What file did this link reference?
The MessageCreatedHasALink occured only once which can be seen in a log visualisation. To check it I put it in a query:"index=vantage earliest="08/27/2026:12:28:00" Operation="MessageCreatedHasLink""
<img width="998" height="881" alt="image" src="https://github.com/user-attachments/assets/a79067b4-6e29-4e67-9f50-a4edacb027af" />

* What operating system was used in the suspicious sign-in to David Chen's account?
* What IP address was used in this same sign-in?
After filtering the previously noticed Linux device that is not matchin company policy, setting appropriate timestamp and setting Operation to "UserLoggedIn" I was able to determine ClientIp.
<img width="418" height="547" alt="image" src="https://github.com/user-attachments/assets/75803f6a-472d-4d85-ba90-ecdc13cb723c" />

* How much time passed between this message being created and the suspicious sign-in to David Chen's account?
* The file the attacker downloaded from David Chen's account was legitimately modified earlier that day. At what time?
First I need to find the file downloaded by David Chen. To find the name I used the query below:
"index=vantage Operation=FileDownloaded
|  table _time ClientIP action Operation UserId ObjectId"

There was a file downloaded indeed named "Q4_Board_Financial_Summary.xls". I changed the Operation to file modified and there it was:
<img width="1377" height="427" alt="image" src="https://github.com/user-attachments/assets/9f321066-597a-48bf-ae6e-9a355238b5e5" />


* What file stored on the Leadership site had a sensitivity label applied to it by the organization's data governance, prior to the incident?
By applying filter to show which files had sensitivity applied, Splunk shows only 2 logs showint this operation, but only one refers the file on Leadership site. The query used:
index=vantage Operation=FileSensitivityLabelApplied
|  table _time ClientIP action Operation UserId ObjectId
<img width="1379" height="489" alt="image" src="https://github.com/user-attachments/assets/6d43de3e-54d9-4bbc-a92e-e2b09d57ae5b" />

* What file did the attacker create an external sharing link for?
First I ran: 
"index=vantage
| stats count by Operation" to see if there were any operations related to creating external sharing link.

It showed:
SharingInheritanceBroken	3
SharingInvitationCreated	1
SharingLinkUpdated	1
SharingLinkUsed	2
SharingSet
Logically to create an extrenal sharing link,first an invitation should be created. There is simply only one situation logged of that instance.
<img width="1379" height="489" alt="image" src="https://github.com/user-attachments/assets/6f2cbb37-f829-4295-a624-70c0b0ff5d54" />

* What external email address was granted access to this file?
For this one I used query: "index=vantage Operation=SharingSet UserId="david.chen@vantagedynamics.onmicrosoft.com"" to find any changes in Sharing Permissions made by compromised account. There indeed was Permission to Contibute granted and it was granted for someone with original mail d.reynolds88@protonmail.com.

<img width="1096" height="626" alt="image" src="https://github.com/user-attachments/assets/360d7256-bca7-425e-8ec3-49ac19494886" />
Not only that but the user was also added:
<img width="722" height="563" alt="image" src="https://github.com/user-attachments/assets/83b97746-f59b-4453-8387-87d05fb2a091" />

* According to the emails, what is the subject line of the notification sent to this external address by the sharing application?
After filtering logs by sourcetype, specifically message trace, I filtered recipients to show only suspicious email.
<img width="668" height="273" alt="image" src="https://github.com/user-attachments/assets/9f730e97-7b3a-4d2e-82cd-278ddc50f065" />
Final record with a message look like this:
<img width="1002" height="454" alt="image" src="https://github.com/user-attachments/assets/f4bba8e9-1e30-4ca5-8d1a-34dcd06b2651" />



After solving challenges and investigating the logs there is a clear correlation of attacks and this is how the attack was occured step by step:
1
2
3...

Who was affected:
1
2
3

What to do:
1
2
3


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
