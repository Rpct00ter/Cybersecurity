# Security Investigation - Splunk-Triage-True-Positive-Phishing-002

## Source
TryHackMe SOC Simulator – Alert 1005: **Suspicious Attachment Found in Email**

## Scenario
An inbound email containing a suspicious ZIP attachment was flagged for investigation.

## Tasks
- Investigate the sender, email content, and attachment context.
- Correlate email-related events with endpoint activity in Splunk.
- Identify attacker actions, assess impact, and document escalation recommendations.

## Tools & Technologies
- Splunk Enterprise
- TryHackMe SOC Simulator
- Cisco Talos
- Windows endpoint telemetry / PowerShell event analysis

---

# Investigation

### 1. Initial Alert / Detection

I assigned myself alert **1005 – Suspicious Attachment found in email**, categorized as a low-severity phishing alert.

| Field | Value |
|---|---|
| Sender | `john@hatmakereurope.xyz` |
| Recipient | `michael.ascot@tryhatme.com` |
| Subject | `FINAL NOTICE: Overdue Payment - Account Suspension Imminent` |
| Attachment | `ImportantInvoice-February.zip` |
| Direction | Inbound |
| Alert timestamp | `2026-10-21 10:48:34.908` (alert display) |

![Alert details](https://github.com/user-attachments/assets/ce49c92d-b4f7-4749-b9eb-b99af0f789fd)

### 2. Triage

#### Sender activity

I searched Splunk for messages from the sender to determine whether the address had sent additional messages:

```spl
index=*
| search sender="john@hatmakereurope.xyz"
```

![Sender search](https://github.com/user-attachments/assets/c6d73cad-57b3-4c10-974a-c358b81a31d6)

The search returned one matching message in the available data. This provided no additional sender activity to correlate, but does not establish that the sender is benign.

#### Domain reputation

I checked `hatmakereurope.xyz` using Cisco Talos. The lookup returned no established reputation or categorization information.

![Talos domain lookup](https://github.com/user-attachments/assets/7ab5d4f1-5d94-42d5-b0b3-310eacc1b76e)

A reputation result of unknown should not be treated as proof that a domain is safe.

#### Email content and attachment

The message used pressure tactics, including **“FINAL NOTICE,” “URGENT,” “immediate payment,”** and threats of legal action. It urged the recipient to open the attached invoice immediately. These are common social-engineering indicators.

![Email details](https://github.com/user-attachments/assets/34479cc0-59ca-4b08-91dd-b5ade23f2131)

The attachment was not available for direct analysis in the simulator. I therefore could not calculate its hash, scan it with VirusTotal, or detonate it in a sandbox. Given the suspicious email content and subsequent endpoint evidence, I escalated the incident rather than treating the attachment as safe.

### 3. Evidence Analysis

#### Attachment-related events

I searched for the attachment filename across the available Splunk indexes:

```spl
index=*
| search "ImportantInvoice-February.zip"
```

![Attachment search](https://github.com/user-attachments/assets/7e0f5343-9a9b-4282-ad71-d300a1e70cbf)

The search returned six events associated with the filename. Three file-creation events occurred at approximately `2026-10-21 10:09:55.090`, associated with `OUTLOOK.EXE`, PID `8668`, and a path within Outlook's local cache.

![Outlook file creation events](https://github.com/user-attachments/assets/91a4131c-df65-4d77-992f-ab83c32a5e87)

The events indicate that the attachment was written to a local Outlook-related cache location. File creation alone does not prove execution; the later endpoint activity provided stronger evidence of compromise.

#### Expanded host investigation

I broadened the search to host `win-3450`:

```spl
index=*
| search host.name="win-3450"
```

![Host events](https://github.com/user-attachments/assets/b98d6586-68d5-4aa0-a7f1-cc45c8bb9cb2)

The search returned 95 events in the available dataset. I filtered through the events, distinguishing routine activity (including printer-related activity) from events relevant to the suspicious sequence.

![Routine events example](https://github.com/user-attachments/assets/9c256332-4dbd-4483-9320-04caa2cc7539)

![Routine events example 2](https://github.com/user-attachments/assets/45143903-a82e-4743-9819-78a5e0710480)

> **Timeline caveat:** The SOC simulator session was restarted during the investigation. The later endpoint events were captured in the restarted session and show timestamps on `2026-11-21`. These are documented as a separate observed sequence, rather than being presented as a continuous clock sequence with the original email timestamps.

#### Suspicious download and command execution

Endpoint events showed a DNS query for `raw.githubusercontent.com` from `powershell.exe`, followed by activity consistent with downloading and running PowerCat from GitHub.

![DNS query](https://github.com/user-attachments/assets/7c75cc89-2e50-43e6-ba02-eef85fbe75c4)

A file-stream/hash event was recorded for an `invoice.pdf.lnk` file in a temporary location:

![LNK file event](https://github.com/user-attachments/assets/06099c8c-a67b-4c53-b693-c55e26a1758a)

The process telemetry showed PowerShell using `System.Net.WebClient` to retrieve `powercat.ps1` from GitHub, followed by a PowerCat command connecting to `2.tcp.ngrok.io` on port `19282` and using PowerShell as the command shell. This is consistent with reverse-shell behavior.

![PowerCat execution](https://github.com/user-attachments/assets/874aeb3f-81ba-4411-b85e-7152dbb5a70f)

PowerShell pipeline execution telemetry further recorded activity associated with the command:

![PowerShell pipeline](https://github.com/user-attachments/assets/d60e4b43-f93f-4636-923d-608252da032a)

#### Host discovery and execution-policy bypass

At approximately `2026-11-21 20:22:00.666`, process telemetry recorded discovery commands such as `net.exe user` and `whoami.exe /priv`, launched by PowerShell. This is consistent with initial host and privilege-context discovery after remote command execution.

![Discovery commands](https://github.com/user-attachments/assets/44e48e46-c240-418a-8fc2-65a51967a627)

At approximately `20:23:08.666`, PowerShell telemetry recorded an execution-policy bypass command.

![Execution policy bypass](https://github.com/user-attachments/assets/2667580f-9f1f-4f43-9a4b-8009046e5e5d)

#### Active Directory reconnaissance

Additional PowerShell activity and script-block telemetry appeared consistent with Active Directory / domain reconnaissance. The activity included PowerView-related tooling and commands to enumerate domain and computer information.

![AD reconnaissance](https://github.com/user-attachments/assets/ea3ba8f0-c190-4872-9c9e-16d8c249374c)

![AD reconnaissance continued](https://github.com/user-attachments/assets/9a5cfc94-8246-45b2-a5bd-95205ea7a0eb)

#### Data collection

The attacker created an `exfiltration` directory under the user's Downloads path. Telemetry subsequently showed files appearing in that directory, including:
- `InvestorPresentation2023.pptx`
- `ClientPortfolioSummary.xlsx`

![Exfiltration directory](https://github.com/user-attachments/assets/321fe8b3-3de6-441b-9bfb-768f8f91e72c)

![Collected files](https://github.com/user-attachments/assets/1a97d2df-ff20-40f2-af41-89d05fbfd39e)

![File activity](https://github.com/user-attachments/assets/c04acd0a-5293-4822-a6ab-63aa64187e0c)

#### Further domain enumeration and DNS-based exfiltration

At approximately `2026-11-21 20:26:36.666`, PowerShell activity was consistent with enumerating domain computers and checking their connectivity.

![Domain computer enumeration](https://github.com/user-attachments/assets/03521e9d-f80f-4ab1-9286-beaede539f76)

![Enumeration results](https://github.com/user-attachments/assets/4f5e9347-5baf-45a0-96a2-2b6029860bc3)

Later PowerShell telemetry showed the contents of `BitcoinWalletPasscodes.txt` being read, Base64-encoded, split into chunks, and sent through DNS queries to `haz4rdw4re.io`. This is consistent with DNS-based data exfiltration.

![DNS exfiltration command](https://github.com/user-attachments/assets/4dcbc930-f32e-4d04-b2e4-7d1fa0751fa1)

The last observed PowerShell script activity in this sequence was around `2026-11-21 20:27:00.666`.

### 4. Correlation

Because the simulator session was restarted, the original email/attachment timestamps and the later endpoint sequence are presented separately, rather than as one uninterrupted timeline.

| Observed time | Event | Interpretation |
|---|---|---|
| `2026-10-21` – alert time | Inbound email with `ImportantInvoice-February.zip` | Suspicious phishing email detected |
| `2026-10-21` – attachment-related events | `OUTLOOK.EXE` file-creation events in Outlook cache | Attachment appears to have been written to local cache |
| `2026-11-21 20:21:35.666` (approx.) | DNS query for `raw.githubusercontent.com` | Possible retrieval of remote tooling |
| `2026-11-21 20:21:43.666` | File-stream/hash event for `invoice.pdf.lnk` | Suspicious shortcut-related artifact |
| `2026-11-21 20:21:44.666` (approx.) | PowerShell downloads PowerCat and connects to ngrok endpoint | Reverse-shell behavior |
| `2026-11-21 20:22:00.666` | `net.exe user`, `whoami.exe /priv` | Host/user/privilege discovery |
| `2026-11-21 20:23:08.666` | PowerShell execution-policy bypass | Bypass of execution restrictions |
| Later in sequence | PowerView-related / domain enumeration activity | Domain reconnaissance |
| Later in sequence | `exfiltration` directory and files created | Data collection staging |
| `2026-11-21 20:26:36.666` (approx.) | Domain computer enumeration / connectivity checks | Network and domain discovery |
| Later in sequence | Base64 encoding and DNS queries to `haz4rdw4re.io` | Likely DNS-based exfiltration |

### 5. Threat Intelligence

| Indicator | Context | Assessment |
|---|---|---|
| `hatmakereurope.xyz` | Email sender domain; checked in Cisco Talos | No established reputation result; not proof of benignness |
| `raw.githubusercontent.com` | PowerShell retrieval of PowerCat | Legitimate hosting service used to retrieve tooling; context is suspicious |
| `2.tcp.ngrok.io:19282` | PowerCat connection command | Tunnel endpoint used in reverse-shell command |
| `haz4rdw4re.io` | DNS queries carrying encoded/split content | Strongly suspicious in the observed context |

The context and behavior of these indicators are more meaningful than the reputation of any single domain. The available evidence did not establish the sender-domain reputation as malicious, but endpoint behavior strongly supports a compromise.

### 6. MITRE ATT&CK

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Attachment | T1566.001 | Suspicious inbound email with ZIP attachment and urgency-based social engineering |
| Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell used to download and execute PowerCat, run commands, and perform reconnaissance |
| Ingress Tool Transfer | T1105 | PowerShell downloaded PowerCat from GitHub |
| Remote Access Tool | T1219 (provisional) | PowerCat used to establish a reverse shell via an ngrok endpoint |
| System Information Discovery | T1082 | Discovery commands executed after remote shell access |
| Account Discovery | T1087 (provisional) | `net.exe user` activity |
| Domain Discovery | T1482 (provisional) | PowerShell/PowerView-related domain reconnaissance |
| File and Directory Discovery | T1083 (provisional) | Activity involving file discovery/collection and file paths |
| Data Staged | T1074.001 | Files gathered in a directory named `exfiltration` |
| Exfiltration Over Alternative Protocol | T1048 (provisional) | Encoded data transmitted through DNS queries |
| Obfuscated Files or Information | T1027 | Base64 encoding and splitting of file contents before transmission |

Technique mappings are behavior-based and indicative; several are provisional and should be refined against the exact commands and event fields.

## Findings

The investigation identified a phishing email containing a suspicious ZIP attachment, followed by endpoint activity consistent with successful compromise of `win-3450`.

Evidence included PowerShell retrieving and running PowerCat, reverse-shell behavior, host and domain reconnaissance, data staging, and apparent DNS-based exfiltration. Files observed in the staging directory included `InvestorPresentation2023.pptx` and `ClientPortfolioSummary.xlsx`. Telemetry also indicated access to `BitcoinWalletPasscodes.txt` and transmission of encoded content through DNS queries to `haz4rdw4re.io`.

**Verdict:** True Positive – compromise supported by endpoint evidence

**Severity:** Critical

**Affected Assets:** `win-3450` / user context `michael.ascot`

**Scope:** Evidence indicates significant compromise of the investigated endpoint and potential data exfiltration. No indicators of compromise were identified on other hosts during the limited checks performed; this does not conclusively rule out wider compromise.

## Response & Recommendations

- Immediately isolate `win-3450` from the network while preserving evidence.
- Escalate to Incident Response / L2-L3 for containment, forensic acquisition, and scope determination.
- Reset the affected user's credentials and revoke active sessions/tokens; investigate possible credential exposure.
- Block or monitor `haz4rdw4re.io` and the ngrok endpoint as appropriate, and review DNS logs for related queries.
- Investigate the downloaded PowerCat script, shortcut (`.lnk`) artifact, PowerShell history/script-block logs, and related process tree.
- Identify and preserve staged files and determine whether additional sensitive data was accessed or exfiltrated.
- Hunt across other endpoints and accounts for PowerShell download-and-execute patterns, PowerCat, ngrok, PowerView, suspicious DNS payloads, and the same filenames/commands.
- Review and strengthen controls for email attachments, PowerShell logging, DNS monitoring, and endpoint protection.
