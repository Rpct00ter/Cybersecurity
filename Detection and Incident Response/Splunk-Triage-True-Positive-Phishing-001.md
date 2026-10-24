# Security Investigation - Phishing Alert

## Source
TryHackMe - SOC Simulator

## Scenario
Alert `8817` flagged an inbound email claiming unusual Microsoft account sign-in activity. The investigation confirmed a suspicious link and found a firewall event showing that an internal host connected to the destination.

## Tools & Technologies
- SOC Simulator
- Splunk
- Firewall event logs

---

# Investigation

### 1. Initial Alert / Detection

| Field | Value |
|---|---|
| Alert ID | `8817` |
| Rule | Inbound Email Containing Suspicious External Link |
| Severity | Medium |
| Sender | `no-reply@m1crosoftsupport.co` |
| Recipient | `c.allen@thetrydaily.thm` |
| Subject | `Unusual Sign-In Activity on Your Microsoft Account` |
| Email timestamp | `2026-10-20 12:51:54.522` |
| URL | `https://m1crosoftsupport.co/login` |

<img src="https://github.com/user-attachments/assets/f0000307-9df9-4a07-b1fe-2cef182f7c09" width="650" alt="SOC Simulator alert 8817">

### 2. Triage

I searched Splunk for the recipient:

```spl
index=*
| search recipient="j.garcia@thetrydaily.thm"
```

The supplied email event instead identifies the recipient as `c.allen@thetrydaily.thm`. The query shown in the screenshot searches for `j.garcia@thetrydaily.thm`, so the query and the displayed event do not match. The sender search below returned the email event:

```spl
index=*
| search sender="no-reply@m1crosoftsupport.co"
```

<img src="https://github.com/user-attachments/assets/cf157542-afb9-487c-8119-ead1f9ebaa41" width="650" alt="Splunk email event">

The message impersonates Microsoft account security and uses `m1crosoftsupport.co`, substituting the numeral `1` for the letter `i`. This is a strong phishing indicator.

### 3. Evidence Analysis

A broader search for the domain found a firewall event:

```spl
index=* m1crosoftsupport.co
```

The firewall event recorded an allowed HTTPS connection to `https://m1crosoftsupport.co/login`.

| Field | Value |
|---|---|
| Time | `2026-10-20 12:53:03.522` |
| Source IP | `10.20.2.25` |
| Destination IP | `45.148.10.131` |
| Destination port | `443` |
| Action | `allowed` |
| URL | `https://m1crosoftsupport.co/login` |

<img src="https://github.com/user-attachments/assets/bffa6218-5202-439b-8614-6762f944345b" width="520" alt="Firewall event for the suspicious URL">

The connection occurred about 1 minute and 9 seconds after the email event. This confirms that a connection to the suspicious URL was allowed from the internal source IP. It does **not** establish that credentials were entered or that the account was compromised.

I also searched for the destination IP:

```spl
index=* DestinationIP="45.148.10.131"
```

The investigation notes indicate this was the only matching outbound connection found in the available data.

### 4. Correlation

- **12:51:54.522:** suspicious email received by `c.allen@thetrydaily.thm`.
- **12:53:03.522:** firewall logged an allowed HTTPS connection from `10.20.2.25` to the phishing URL's destination IP.

The close timing is consistent with a recipient or user visiting the link, but the logs shown do not directly link the mailbox to source IP `10.20.2.25`.

### 5. Threat Intelligence

The supplied evidence identifies the lookalike domain and destination IP, but does not include a confirmed reputation result from an external threat-intelligence service. The phishing assessment is based on the message content, domain typo, and observed connection.

### 6. MITRE ATT&CK

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | Email attempts to direct the recipient to a lookalike Microsoft sign-in page. |

No evidence provided confirms credential theft, account compromise, or subsequent attacker activity.

## Findings

The email is a **true-positive phishing alert**. Splunk also shows an allowed HTTPS connection to the suspicious URL from `10.20.2.25`. This raises concern that the link was visited, but compromise remains unconfirmed.

**Verdict:** True Positive

**Severity:** Medium (original alert severity); escalate for investigation of the observed connection.

**Affected entities:** `c.allen@thetrydaily.thm`; source IP `10.20.2.25`; destination `45.148.10.131`; domain `m1crosoftsupport.co`.

<img src="https://github.com/user-attachments/assets/30b363be-85bc-4e87-bb94-4ccb897f8407" width="650" alt="Submitted case report for alert 8817">

## Response & Recommendations

- Block `m1crosoftsupport.co` and the phishing URL through appropriate DNS, proxy, and web-filter controls.
- Identify the host and user associated with `10.20.2.25`; correlate DNS, proxy, endpoint, and authentication logs around `12:53:03`.
- Contact the user to establish whether the page was opened and credentials were submitted.
- If credentials may have been exposed, reset them, revoke active sessions, and review sign-in activity.
- Search for other messages containing the sender, domain, URL, or destination IP and remove/quarantine related emails where possible.
- Keep the incident escalated until the connection's context and potential credential exposure have been assessed.
