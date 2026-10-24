# Security Investigation - Introduction to Phishing

## Source
TryHackMe — SOC simulator

## Scenario

The SOC alert queue generated an alert named **Inbound Email Containing Suspicious External Link**. The alert was categorized as **Phishing** with **Medium** severity. The investigation required reviewing the email evidence and searching the available Splunk events to decide whether the alert represented a true or false positive. 

## Tasks
- Decide whether the alert is a true positive or false positive based on available evidence.

## Tools & Technologies

- **TryHackMe SOC simulation** - alert queue, alert assignment, and case reporting
- **Splunk** - search and review events
- **ICANN Lookup** - used for email domain analysis
- **Virustotal** - used for URL analysis

---

# Investigation

## 1. Initial Alert / Detection

The alert queue showed the following alert:

| Field | Value |
|---|---|
| Alert ID | `8814` |
| Alert rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Type | Phishing |
| Queue date | October 20, 2026, 13:48 (as displayed by the lab) |
| Status at discovery | Awaiting action |

The alert description said an inbound email contained one or more external links with potentially suspicious characteristics. It recommended checking firewall or proxy logs to determine whether endpoints attempted to access the URLs and whether those connections were allowed or blocked.

### Email details

| Field | Observed value |
|---|---|
| Datasource | `email` |
| Event timestamp | `10/20/2026 13:46:23.522` (as shown in the alert details) |
| Direction | `inbound` |
| Sender | `onboarding@hrconnex.thm` |
| Recipient | `j.garcia@thetrydaily.thm` |
| Subject | `Action Required: Finalize Your Onboarding Profile` |
| Attachment | None |
| Host shown in Splunk | `10.10.160.225:8989` |
| Link host | `hrconnex.thm` |
| Link path | `/onboarding/1540065406/j.garcia` |

The message claimed the recipient needed to complete their onboarding profile and included a link labeled **Set Up My Profile**. The sender and link use a similar-looking domain, which may make the message appear plausible, but visual similarity alone does not establish legitimacy.

**Screenshot — initial alert queue**

![Initial alert queue showing alert 8814](https://github.com/user-attachments/assets/ac5a56a3-8f50-467c-b967-cb63ce4db716)

**Screenshot — expanded alert details and the second queued alert**

![Expanded alert 8814 and alert 8815 in the queue](https://github.com/user-attachments/assets/f8dedca9-cfa2-4221-b682-9e6eb754e9f1)

## 2. Triage

### Step 1: Find the triggering email event

The first step was to search Splunk for events with the sender address shown in the alert.

```spl
index = *
| search sender = "onboarding@hrconnex.thm"
```

This returned the email event associated with the alert. The event contained the sender, recipient, subject, message body, timestamp, and link. The available event data showed no attachment.

**Screenshot — Splunk search by sender**

![Splunk results for the sender address](https://github.com/user-attachments/assets/b5137ee1-f5a2-42c3-bbf8-309de78a2f2a)

### Initial observations

- The sender address, `onboarding@hrconnex.thm`, appeared consistent with the onboarding theme of the message.
- The URL used the host `hrconnex.thm`, which resembles the sender's domain.
- The message asked the recipient to complete an onboarding profile.
- No attachment was listed.

These observations explain why the email could look legitimate at first glance. They are **not sufficient on their own to rule out phishing**: display names and sender addresses can be spoofed, and a domain that resembles a legitimate service can still be attacker-controlled.

### Step 2: Review other messages for the recipient

The next search looked for other email events involving the recipient:

```spl
index = *
| search recipient = "j.garcia@thetrydaily.thm"
```

The supplied Splunk screenshot showed multiple messages to the same recipient. The visible examples included:

- **Employee Feedback Survey** — content asking the recipient to provide feedback.
- **Hiring Update - Interview Schedule** — content referring to interviews and checking calendars.
- **Action Required: Finalize Your Onboarding Profile** — the message that triggered alert 8814.
- **New Product Launch - All Hands on Deck** — an internal-sounding announcement.
- Other messages were also present in the results.

The surrounding messages gave context that the recipient might be involved in a hiring or onboarding process. However, the screenshot alone does not verify that the sender or linked site was authorized.

**Screenshot — Splunk search by recipient**

![Splunk events for recipient j.garcia@thetrydaily.thm](https://github.com/user-attachments/assets/2d7a551a-04cc-490b-85f3-3d0643d9c334)

### Triage assessment

The recipient's other messages made the onboarding explanation plausible, but the evidence provided did not show URL reputation, domain ownership, email authentication results, or whether the link was accessed. These checks would help distinguish a legitimate onboarding message from a well-crafted phishing email.

## 3. Evidence Analysis

### Email evidence

| Evidence | Observation | Assessment |
|---|---|---|
| Sender | `onboarding@hrconnex.thm` | Plausible for onboarding, but not proof of authenticity. |
| Recipient | `j.garcia@thetrydaily.thm` | Matches the user investigated in the recipient search. |
| Subject | `Action Required: Finalize Your Onboarding Profile` | Uses urgency and an account/profile action; common in both legitimate onboarding and phishing. |
| Message body | Welcomes the recipient and asks them to set up a profile. | Context is plausible, but should be independently verified. |
| URL host | `hrconnex.thm` | Similar to the sender domain; reputation and ownership were not established in the supplied evidence. |
| Attachment | None | No attachment was recorded; the link remains the primary concern. |
| Other messages | Hiring/interview and workplace-related messages were visible for the same recipient. | Provides context, not proof that this email is safe. |
| Splunk host field | `10.10.160.225:8989` | Internal/lab event metadata shown in Splunk; its exact role was not established. |

Additionally I checked domain reputation to check for correlation between sender and receiver e-mail, but supposedly because of inner character of SOC Simulation there was no registry for this domain. Additionaly URL shown in the message doesn't raise any suspicions in Virustotal, because it can't find the URL (its possible that the URL was generated for SOC Simulation):
<img width="1050" height="229" alt="image" src="https://github.com/user-attachments/assets/2fa3b884-ed84-4500-95d8-4d136321ea01" />

### Missing evidence / recommended validation

The supplied notes do not include results for the following checks. They should therefore be treated as **not verified**, rather than as clean results:

- Email authentication and header review (for example, SPF, DKIM, and DMARC results where available).
- Domain registration, ownership, and reputation for `hrconnex.thm`.
- URL analysis in an approved, isolated environment.
- Proxy, DNS, firewall, or endpoint telemetry showing whether the URL was accessed.
- Whether a connection was allowed or blocked.
- Whether the destination requested credentials or collected sensitive information.
- Independent confirmation from the organization's HR/onboarding team.

Do not visit a suspicious URL directly from a normal workstation. If further validation is required, use the organization's approved security tooling and safe-analysis procedures.

## 4. Correlation

### Available timeline

The screenshots show more than one timestamp for the same event. The table below preserves the values as displayed and does not assume that the discrepancy has been resolved.

| Time / date shown | Event | Source |
|---|---|---|
| October 20, 2026, `12:46:23.522 PM` | Email event displayed in the expanded Splunk event screenshot. | Splunk screenshot |
| October 20, 2026, `13:46:23.522` | Timestamp field shown in the alert details. | Alert details |
| October 20, 2026, around `12:48` | Alert 8814 detection time shown in the case-report screen. | Case report |
| October 20, 2026, `13:48` | Alert 8814 date/time shown in the alert queue. | Alert queue |
| During the investigation | Alert 8815 appeared in the queue with the same rule and Medium severity. | Alert queue |
| After review | Alert 8814 was classified as a false positive in the lab and the alert was closed. | Closure and case-report screenshots |

**Timestamp note:** The queue, case report, and event fields appear to use different displayed times. Confirm the lab's timezone and timestamp semantics before constructing a definitive chronology. The screenshots are not enough to establish whether this is a timezone conversion, a display inconsistency, or a difference between event and detection times.

### Correlation result

The sender search identified the email event matching the alert's sender, and the recipient search returned other messages addressed to the same recipient. No supplied screenshot demonstrates a related URL connection, endpoint execution, credential submission, or other post-delivery activity.

## 5. Threat Intelligence

No threat-intelligence lookups or URL-reputation results were included in the supplied notes.

The following indicators were identified from the event and could be investigated using approved threat-intelligence sources:

| Indicator | Type | Status |
|---|---|---|
| `hrconnex.thm` | Domain | Reputation/ownership not checked in the supplied evidence. |
| `https://hrconnex.thm/onboarding/1540065406/j.garcia` | URL | Not shown as detonated or reputation-checked. |
| `onboarding@hrconnex.thm` | Sender address | Authenticity not independently verified. |
| `10.10.160.225:8989` | Host/port shown in Splunk metadata | Role not established; do not assume it is the destination URL's server. |

The `.thm` names and private-address context appear consistent with a lab environment. Do not infer that these are public, real-world indicators or that a reputation lookup would be meaningful outside the simulation.

## 6. MITRE ATT&CK

The alert concerns a potentially suspicious inbound email, but the available evidence does not confirm malicious activity. The following mapping is therefore **provisional**.

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Link | [T1566.002](https://attack.mitre.org/techniques/T1566/002/) | The email contained an external onboarding link and triggered a suspicious-link alert. Malicious intent was not independently confirmed. |

This technique should be retained only if it accurately represents the lab's intended classification. If the message is ultimately verified as legitimate, the alert alone should not be treated as proof that the technique occurred.

## Findings

The alert was generated because an inbound email contained an external link. Searching by sender located the relevant event, and searching by recipient revealed other messages with hiring- and workplace-related context. Those details made the onboarding message plausible, but the supplied evidence did not independently validate the sender, domain, URL, or any subsequent network activity.

**Verdict:** False Positive *(the lab's recorded decision; not independently confirmed by the supplied evidence)*

**Severity:** Medium *(as assigned by the alert rule)*

**Affected Assets:**
- Recipient: `j.garcia@thetrydaily.thm`
- Email event associated with alert `8814`
- Splunk metadata host: `10.10.160.225:8989` (role not established)

### Closure rationale recorded in the lab

The submitted case report classified the alert as a false positive because the link appeared to match the sender's domain, the message seemed consistent with onboarding, and other messages suggested the recipient could be in a hiring/onboarding process.

This reflects the analyst's lab decision. Because sender/domain similarity and contextual messages do not prove authenticity, a production investigation should validate the URL and email authentication before closing the alert as benign.

**Screenshot — alert closure decision**

![Alert 8814 closure dialog with False positive selected](https://github.com/user-attachments/assets/2b1b62d1-2d7f-4bfd-9762-7184cc633d0c)

**Screenshot — submitted case report**

![Case report for alert 8814](https://github.com/user-attachments/assets/d77fc6b0-95ce-4faa-9564-6ed017eb7f50)

## Response & Recommendations

### Actions completed in the lab

- Assigned alert `8814`.
- Searched Splunk for the sender and recipient.
- Reviewed the email details and related recipient messages.
- Selected **False Positive** in the alert closure workflow.
- Submitted the case report and closed the alert.

### Recommended follow-up in a real SOC

1. **Validate the sender and domain.** Review full email headers and authentication results; verify the onboarding domain through a trusted, independent source.
2. **Inspect the URL safely.** Use approved URL-reputation and sandbox tooling rather than opening it on a standard workstation.
3. **Check network telemetry.** Search DNS, proxy, firewall, and endpoint logs for requests to the destination and determine whether they were allowed or blocked.
4. **Check for user impact.** If the link was visited, investigate redirects, credential entry, downloads, and subsequent endpoint or identity activity.
5. **Preserve evidence.** Retain the original email, headers, relevant event records, timestamps, and query results according to the organization's evidence-handling process.
6. **Document uncertainty.** If authenticity cannot be established, keep the case inconclusive or escalate it instead of treating plausible wording as sufficient proof of a false positive.
7. **Review alert 8815 separately.** It appeared while alert 8814 was being investigated. Do not assume it has the same cause or verdict merely because it shares the same alert rule.

### Splunk queries used

**Find email events from the sender:**

```spl
index = *
| search sender = "onboarding@hrconnex.thm"
```

**Find email events addressed to the recipient:**

```spl
index = *
| search recipient = "j.garcia@thetrydaily.thm"
```
