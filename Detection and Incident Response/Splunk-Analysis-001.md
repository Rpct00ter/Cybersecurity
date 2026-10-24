# Security Investigation - The O365 Intrusion

## Source
TryHackMe: [The O365 Intrusion Challenge](https://tryhackme.com/room/theo365intrusion)  
## Scenario
Microsoft 365 (O365) intrusions commonly begin with stolen credentials-often through phishing, fake sign-in pages, password reuse, or session-token theft. Once inside, an attacker may read email, create mailbox rules to hide or target messages, abuse collaboration tools, access cloud files, and share sensitive data externally. Consequences can include business-email compromise, fraud, exposure of confidential documents, and persistent unauthorized access.

Vantage Dynamics escalated suspicious activity involving Marcus Webb and David Chen on 27 August 2026. The investigation uses Splunk logs from `index=vantage` to identify the entry point, mailbox persistence, movement to a second identity, and possible data exposure.

## Tasks
- Establish how Marcus Webb’s account was initially compromised.
- Identify the attacker’s immediate actions and mailbox persistence.
- Trace the move from Marcus Webb to David Chen.
- Determine which sensitive files were accessed and whether external sharing occurred.

## Tools & Technologies
- Splunk / SPL
- Microsoft 365 Unified Audit Log, Exchange message trace, Teams, SharePoint and Entra ID sign-in events
- MITRE ATT&CK

---

# Investigation

### 1. Initial Alert / Detection

The initial report described an unfamiliar document being associated with Marcus Webb and unusual activity in David Chen’s account. The organization issues managed MacBooks to employees, so successful sign-ins using Windows or Linux were suspicious and required correlation with audit events.

Set Splunk’s time picker to **All time** and search `index=vantage`. Review the `UserAgent` field to identify devices outside the expected macOS baseline.

<img width="520" alt="UserAgent top values" src="https://github.com/user-attachments/assets/814849a5-063e-418f-ac6b-5ce6a317fcab" />

The UserAgent distribution includes Windows Chrome and Linux Chrome. Filtering for successful `UserLoggedIn` events tied to Marcus isolated the Windows sign-in. The available evidence identifies the account takeover, but does not establish the exact credential-theft method; phishing or token theft should not be stated as confirmed without further evidence.

### 2. Triage

A focused Splunk query for Marcus’s successful Windows Chrome sign-in:

```spl
index=vantage sourcetype="o365:management:activity"
UserAgent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36"
Operation=UserLoggedIn ResultStatus=Success
UserId="marcus.webb@vantagedynamics.onmicrosoft.com"
```

The event shows `UserLoggedIn`, `ResultStatus=Success`, Windows 10, and source IP `185.220.101.47`. The event detail screenshot displays a timestamp of `2026-08-27 12:28:17`; the walkthrough gives the challenge answer as `2026-08-27 12:28:31`. Treat this small timestamp discrepancy as a validation point when reproducing the query and selecting the first qualifying event.

<img width="520" alt="Marcus Webb successful non-macOS sign-in query" src="https://github.com/user-attachments/assets/9cd587b6-1a42-40bf-9d39-f3ddddf46009" />

### 3. Evidence Analysis

**Mailbox access and persistence**

After the suspicious sign-in, `MailItemsAccessed` events appeared for Marcus. The challenge walkthrough reports **22 emails accessed** in the initial window. One displayed event has `OperationCount=6`; this is an event-level count, not the total across the window.

```spl
index=vantage Operation=MailItemsAccessed
earliest="08/27/2026:12:28:00"
UserId="marcus.webb@vantagedynamics.onmicrosoft.com"
```

<img width="520" alt="MailItemsAccessed events for Marcus Webb" src="https://github.com/user-attachments/assets/921f1727-08b4-448f-8ccc-9efa0ca4f410" />

A `New-InboxRule` event was created at `12:30:10`. Its `SubjectContainsWords` parameter is `invoice`, indicating a rule matching invoice-related messages. This is consistent with an attempt to target or manipulate business-payment correspondence. The provided event excerpt does not fully establish every rule action, so avoid claiming that it definitely deleted or forwarded messages.

```spl
index=vantage earliest="08/27/2026:12:28:00"
UserId="marcus.webb@vantagedynamics.onmicrosoft.com"
Operation="New-InboxRule"
```

<img width="520" alt="New inbox rule matching invoice" src="https://github.com/user-attachments/assets/ff2a2611-4fb0-4bd2-aecf-38cd17c3c54c" />

A message trace shows a matching reply delivered to Marcus at `12:41:09`, with subject `RE: Meridian invoice batch - please confirm`. From rule creation at `12:30:10`, the elapsed time was **10 minutes 59 seconds**.

<img width="520" alt="Matching reply message trace" src="https://github.com/user-attachments/assets/fbaa51de-0a88-4634-8ac0-c37d6a827589" />

<img width="520" alt="Message trace details" src="https://github.com/user-attachments/assets/b285858d-3cfd-4a03-8383-1194f0dcb366" />

**Teams link and second account**

At `13:03:38`, a Teams `MessageCreatedHasLink` event from Marcus contained a link to `Meridian_Invoice_Batch_Aug2026.pdf`. The logged URL shown in the screenshot appears to repeat the `.pdf` suffix in one field; the challenge answer uses the filename with a single `.pdf` suffix.

<img width="520" alt="Teams message containing the invoice batch link" src="https://github.com/user-attachments/assets/a79067b4-6e29-4e67-9f50-a4edacb027af" />

At `13:27:04`, David Chen’s account recorded a successful sign-in from a Linux device at `91.219.237.88`. This is inconsistent with the managed macOS baseline and occurred **23 minutes 26 seconds** after the Teams message. The matching timeline supports a related intrusion, though the logs alone do not prove exactly how David’s credentials or session were obtained.

<img width="520" alt="David Chen suspicious Linux sign-in" src="https://github.com/user-attachments/assets/75803f6a-472d-4d85-ba90-ecdc13cb723c" />

**Sensitive files and external sharing**

A `FileModified` event shows `Q4_Board_Financial_Summary.xlsx` was modified at `2026-08-27 08:27:03`, earlier that day. A `FileSensitivityLabelApplied` event also identifies this workbook on the Leadership SharePoint site, indicating that it had a sensitivity label applied. The screenshot shows a label event at `08:27:41` for David’s workbook and another at `09:10:32` for Marcus’s `Contract_Review_Draft.docx`; the Leadership-site file requested by the challenge is the workbook.

```spl
index=vantage Operation=FileModified
| table _time ClientIP action Operation UserId ObjectId
```

<img width="520" alt="File modification event for the Leadership workbook" src="https://github.com/user-attachments/assets/9f321066-597a-48bf-ae6e-9a355238b5e5" />

```spl
index=vantage Operation=FileSensitivityLabelApplied
| table _time ClientIP action Operation UserId ObjectId
```

<img width="520" alt="Sensitivity label events" src="https://github.com/user-attachments/assets/6d43de3e-54d9-4bbc-a92e-e2b09d57ae5b" />

A sharing event for `Vendor_Onboarding_Notes.docx` shows access granted to `d.reynolds88@protonmail.com` with **Contribute** permission. The event timestamp is `14:52:44`; an Azure AD `Add user` event for the external identity appears at `14:52:43`. This is evidence of external sharing/access being configured, but it does **not** by itself prove the recipient downloaded the file.

<img width="520" alt="External sharing event for Vendor Onboarding Notes" src="https://github.com/user-attachments/assets/6f2cbb37-f829-4295-a624-70c0b0ff5d54" />

<img width="520" alt="External sharing permissions" src="https://github.com/user-attachments/assets/360d7256-bca7-425e-8ec3-49ac19494886" />

<img width="520" alt="External user added to the tenant" src="https://github.com/user-attachments/assets/83b97746-f59b-4453-8387-87d05fb2a091" />

The sharing notification email was delivered to the external address with subject: `David Chen shared "Vendor_Onboarding_Notes.docx" with you`.

<img width="520" alt="Message trace filtered for external recipient" src="https://github.com/user-attachments/assets/9f730e97-7b3a-4d2e-82cd-278ddc50f065" />

<img width="520" alt="External sharing notification message trace" src="https://github.com/user-attachments/assets/f4bba8e9-1e30-4ca5-8d1a-34dcd06b2651" />

### 4. Correlation

Key indicators and events connect the two identities:

- Marcus: successful Windows 10 Chrome sign-in from `185.220.101.47`, followed by mailbox access and an invoice-matching inbox rule.
- Marcus: Teams link to the invoice-batch PDF at `13:03:38`.
- David: successful Linux sign-in from `91.219.237.88` at `13:27:04`.
- David: access activity involving the Leadership workbook and external sharing of `Vendor_Onboarding_Notes.docx` to a Proton Mail address.

The UserAgent screenshot also shows common macOS Chrome traffic, plus Windows and Linux agents. UserAgent alone is not proof of compromise; the conclusion relies on the mismatch with company policy and the correlated audit events.

<img width="520" alt="UserAgent distribution in Splunk" src="https://github.com/user-attachments/assets/90f5a569-4f97-46d1-9a61-479ed6e247ff" />

### 5. Threat Intelligence

- `185.220.101.47` - source IP for Marcus’s suspicious Windows sign-in.
- `91.219.237.88` - source IP for David’s suspicious Linux sign-in.
- `d.reynolds88@protonmail.com` - external identity granted access to the shared document.
- `Meridian_Invoice_Batch_Aug2026.pdf` - file linked in the Teams message.
- `Q4_Board_Financial_Summary.xlsx` - sensitive Leadership-site workbook.
- `Vendor_Onboarding_Notes.docx` - document shared externally.

These are incident indicators derived from the lab logs. This report does not assert a verified reputation or attribution for either IP address. Check them against current threat-intelligence sources and tenant sign-in context before operational blocking.

### 6. MITRE ATT&CK

| Technique | ID | Evidence |
| --- | --- | --- |
| Valid Accounts | [T1078](https://attack.mitre.org/techniques/T1078/) | Successful sign-ins using Marcus Webb’s and David Chen’s identities from unexpected operating systems/IPs. |
| Email Collection | [T1114](https://attack.mitre.org/techniques/T1114/) | `MailItemsAccessed` events after Marcus’s suspicious sign-in. |
| Email Forwarding Rule / Inbox Rule abuse | [T1114.003](https://attack.mitre.org/techniques/T1114/003/) | A new mailbox rule matching `invoice`; the observed rule’s full behavior should be confirmed from all parameters. |
| Phishing (possible initial method) | [T1566](https://attack.mitre.org/techniques/T1566/) | A plausible account-compromise route, but not directly established by the supplied logs. Treat as a hypothesis, not a confirmed technique. |
| Exfiltration to Cloud Storage / Web Service (potential exposure) | [T1567](https://attack.mitre.org/techniques/T1567/) | External sharing of a SharePoint document is observed; successful download or exfiltration is not confirmed by this evidence alone. |

## Findings

The evidence supports a **true-positive Microsoft 365 account intrusion** involving Marcus Webb and David Chen. The attacker used an unexpected Windows sign-in for Marcus, accessed mailbox content, created an invoice-matching inbox rule, and later activity was observed from an unexpected Linux sign-in to David’s account. The investigation also found access to a sensitive Leadership workbook and external sharing of a second document.

**Verdict:** True Positive  
**Severity:** High - sensitive business documents and two identities are involved; confirm data access/exfiltration scope during response.  
**Affected assets:**  
- `marcus.webb@vantagedynamics.onmicrosoft.com`
- `david.chen@vantagedynamics.onmicrosoft.com`
- Microsoft 365 mailbox and Teams activity
- Leadership SharePoint document `Q4_Board_Financial_Summary.xlsx`
- Shared document `Vendor_Onboarding_Notes.docx`
- External identity `d.reynolds88@protonmail.com`

**Roadmap**:
1. **Initial access observed:** Marcus Webb’s account successfully signs in from Windows 10 Chrome, contrary to the organization’s managed-MacBook baseline. The event detail shows `12:28:17`; the external walkthrough’s challenge answer is `12:28:31`, so verify the exact event timestamp in Splunk.
2. **Mailbox reconnaissance:** `MailItemsAccessed` events follow the sign-in; the challenge reports 22 emails accessed in the initial window.
3. **Mailbox persistence/targeting:** At `12:30:10`, a new inbox rule matches the word `invoice`, potentially targeting payment-related correspondence.
4. **Business email context:** At `12:41:09`, a reply with subject `RE: Meridian invoice batch - please confirm` arrives-10 minutes 59 seconds after the rule was created.
5. **Collaboration activity:** At `13:03:38`, Marcus’s Teams account creates a message containing a link to `Meridian_Invoice_Batch_Aug2026.pdf`.
6. **Second identity appears:** At `13:27:04`, David Chen signs in successfully from Linux at `91.219.237.88`, 23 minutes 26 seconds after the Teams message.
7. **Sensitive file involved:** `Q4_Board_Financial_Summary.xlsx` on the Leadership SharePoint site was modified at `08:27:03` and had a sensitivity label applied. The available screenshots show the file’s legitimate modification/label activity, but the precise attacker download event should be confirmed in the complete audit log.
8. **External sharing:** At about `14:52:43–14:52:44`, the external identity is added and `Vendor_Onboarding_Notes.docx` is shared with `d.reynolds88@protonmail.com` using Contribute permissions.
9. **Notification sent:** A sharing notification is delivered with subject `David Chen shared "Vendor_Onboarding_Notes.docx" with you`.
10. **Impact assessment:** The observed evidence confirms suspicious account activity and external access being configured. It does not, on its own, prove that the external recipient downloaded the file or establish the initial credential-theft method. Validate download events and complete the tenant-wide scope before declaring the exact amount of data exfiltrated.


## Response & Recommendations

- Revoke active sessions and refresh tokens for both users; reset credentials and verify MFA methods and sign-in risk.
- Remove and preserve evidence of the suspicious inbox rule; review mailbox forwarding, delegates, OAuth consent, and other persistence.
- Temporarily disable or restrict external sharing for the affected SharePoint item; remove the external user/link after preserving audit evidence.
- Review file access/download logs to determine whether the workbook or shared document was actually downloaded by the attacker or external recipient.
- Hunt across the tenant for both IPs, non-macOS sign-ins, similar inbox rules, suspicious Teams links, and other external shares.
- Validate MFA, conditional-access policies, phishing-resistant authentication, alerting for unusual device/OS changes, and sensitive-file sharing controls.
