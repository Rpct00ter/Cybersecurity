# Security Investigation - Splunk-Triage-False-Positive

## Source
TryHackMe — SOC Simulator

## Scenario
Alert `8814`, **Inbound Email Containing Suspicious External Link**, flagged an inbound onboarding email as Medium-severity phishing. I reviewed the email and related Splunk events, then checked the domain and URL in threat-intelligence services.

## Tools & Technologies
- Splunk
- ICANN Lookup
- VirusTotal

---

# Investigation

### 1. Initial Alert / Detection

| Field | Value |
|---|---|
| Alert ID | `8814` |
| Rule / severity | Inbound Email Containing Suspicious External Link / Medium |
| Sender | `onboarding@hrconnex.thm` |
| Recipient | `j.garcia@thetrydaily.thm` |
| Subject | `Action Required: Finalize Your Onboarding Profile` |
| URL | `hrconnex.thm/onboarding/1540065406/j.garcia` |
| Attachment | None |

The email asked the recipient to complete an onboarding profile through an external link.

<img src="https://github.com/user-attachments/assets/ac5a56a3-8f50-467c-b967-cb63ce4db716" width="650" alt="Alert queue showing alert 8814">

### 2. Triage

I searched Splunk for the sender and recipient to locate the triggering email and review surrounding messages.

```spl
index=*
| search sender="onboarding@hrconnex.thm"
```

```spl
index=*
| search recipient="j.garcia@thetrydaily.thm"
```

Other messages referenced hiring and workplace activities, which made the onboarding message plausible but did not prove it was legitimate.

<img src="https://github.com/user-attachments/assets/b5137ee1-f5a2-42c3-bbf8-309de78a2f2a" width="650" alt="Splunk results for sender">
<img src="https://github.com/user-attachments/assets/2d7a551a-04cc-490b-85f3-3d0643d9c334" width="650" alt="Splunk results for recipient">

### 3. Evidence Analysis

- The sender and link used the same `hrconnex.thm` domain.
- The message had no attachment.
- Related messages suggested a possible hiring/onboarding context.
- Domain and URL checks did **not** return usable reputation or registration results. ICANN Lookup reported: **“No registry RDAP server was identified for this domain.”** VirusTotal could not find the URL.

These lookup results are inconclusive: they do not establish that the domain or URL is safe or malicious. To resolve this I assume that this domain and URL were not public and were created for the sake of the SOC Simulator exercise. With this assumption I categorize this as a legit traffic. 

<img src="https://github.com/user-attachments/assets/2fa3b884-ed84-4500-95d8-4d136321ea01" width="650" alt="ICANN Lookup result">

### 4. Correlation
The sender search matched the alert email; the recipient search showed other workplace-related messages. The available evidence did not show whether the link was visited or whether credentials were submitted.

### 5. Threat Intelligence

| Source | Result |
|---|---|
| ICANN Lookup | No registry RDAP server identified for the domain |
| VirusTotal | URL not found / no usable result |

No reputation verdict was available from these checks.

### 6. MITRE ATT&CK

No technique is confidently mapped because the available evidence does not confirm malicious activity.

## Findings

The lab recorded this alert as a **False Positive**. The onboarding context appeared plausible, but the domain and URL lookups were inconclusive. Similarity between sender and link domains is not proof of legitimacy, so the false-positive decision reflects the lab outcome rather than independent confirmation that the link was safe.

**Verdict:** False Positive 

**Severity:** Medium *(original alert severity)*

**Affected Assets:** `j.garcia@thetrydaily.thm`; email alert `8814`

<img src="https://github.com/user-attachments/assets/2b1b62d1-2d7f-4bfd-9762-7184cc633d0c" width="650" alt="False-positive closure decision">
<img src="https://github.com/user-attachments/assets/d77fc6b0-95ce-4faa-9564-6ed017eb7f50" width="650" alt="Submitted case report">

## Response & Recommendations

- Reviewed the alert and searched Splunk by sender and recipient.
- Checked the domain in ICANN Lookup and the URL in VirusTotal; neither returned a usable result.
- Recorded the lab’s false-positive decision. In a real SOC, verify email authentication and proxy/DNS activity before closing a similar alert.
