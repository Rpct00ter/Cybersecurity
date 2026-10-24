# Security Investigation - Malicious Link
## Source
TryHackMe - SOC Simulator

## Scenario
Alert `8816` reported an outbound request to a URL listed by the organization's blacklist or threat-intelligence controls. The firewall blocked the request. The investigation checked the URL's reputation and searched for related activity from the destination and source IPs.

## Tools & Technologies
- SOC Simulator
- Splunk
- VirusTotal
---

# Investigation

### 1. Initial Alert / Detection

| Field | Value |
|---|---|
| Alert ID | `8816` |
| Rule | Access to Blacklisted External URL Blocked by Firewall |
| Severity | High |
| Timestamp | `2026-10-20 13:50:50.522` |
| Action | `blocked` |
| Source IP | `10.20.2.17` |
| Destination IP | `67.199.248.11` |
| Destination port | `80` |
| URL | `http://bit.ly/3sHkX3da12340` |
| Application / protocol | `web-browsing` / TCP |

<img src="https://github.com/user-attachments/assets/0ed70562-119c-4661-b3a8-dc15e94c4ba9" width="650" alt="SOC Simulator alert 8816">

### 2. Triage

I checked the URL using VirusTotal. The supplied screenshot shows **2 of 93 security vendors** flagging it as phishing, with another vendor marking it suspicious. The URL redirects from the `bit.ly` short link to an HTTPS version of the same short link.

<img src="https://github.com/user-attachments/assets/4a203e60-b991-4cb8-834c-e77fa00fcf8e" width="650" alt="VirusTotal URL detection results">

These results support treating the URL as suspicious, but vendor results are not a complete determination of the final destination's behavior.

### 3. Evidence Analysis

I checked whether the destination IP appeared elsewhere in the available Splunk data. The notes indicate no other matching event was found.

I then searched for activity from the source IP:

```spl
index=* SourceIP="10.20.2.17"
```

The results show the blacklisted-URL request was **blocked** at `13:50:50.522`. Another event from the same source IP at `13:48:28.522` records an allowed HTTPS request to a Google search URL. The shown events do not, by themselves, establish compromise.

<img src="https://github.com/user-attachments/assets/9c39a4ed-355f-45be-a903-df57917795f5" width="650" alt="Splunk events for source IP 10.20.2.17">

### 4. Correlation

- **13:48:28.522:** `10.20.2.17` made an allowed HTTPS request to a Google search URL.
- **13:50:50.522:** a request to `http://bit.ly/3sHkX3da12340` was blocked by the firewall.

The firewall prevented the recorded connection to the listed URL. The evidence does not confirm whether the short link was reached through an email, browser redirect, or another source.

### 5. Threat Intelligence

VirusTotal showed a small number of detections for the short URL (2/93 vendors flagged it as phishing, with another marking it suspicious in the screenshot). This supports further investigation and continued blocking. Because the link is shortened, its ultimate destination should be resolved and assessed in a controlled environment before drawing conclusions about the final landing page.

### 6. MITRE ATT&CK

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Link *(possible delivery vector; not confirmed)* | T1566.002 | A suspicious shortened URL was requested, but the evidence does not identify how the user encountered it. |

No successful connection to the flagged URL, credential submission, malware execution, or account compromise is demonstrated by the supplied evidence.

## Findings

The alert is supported by a firewall event showing a request to a suspicious shortened URL, and the firewall action was `blocked`. The source-IP search did not reveal clear additional signs of compromise in the events shown. Escalation is appropriate to establish the origin of the request and check endpoint/user context.

**Verdict:** True Positive - suspicious URL access attempt; no confirmed compromise

**Severity:** High (original alert severity)

**Affected entity:** Source IP `10.20.2.17`; destination IP `67.199.248.11`; URL `http://bit.ly/3sHkX3da12340`

<img src="https://github.com/user-attachments/assets/e519d2f5-312e-4f0d-bce8-ef2fa0a3ed09" width="650" alt="Submitted case report for alert 8816">

## Response & Recommendations

- Keep the URL/domain blocked and search email, proxy, DNS, and browser telemetry to identify how the request was initiated.
- Identify the device and user associated with `10.20.2.17`; review endpoint and authentication events around the alert time.
- Check for related requests, redirects, downloads, or credential submission; do not infer compromise from the blocked request alone.
- Review the short URL's resolved destination using approved, safe analysis procedures and update block lists if additional indicators are confirmed.
