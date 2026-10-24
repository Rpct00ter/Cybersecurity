# Security Investigation — Suspicious Parent-Child Relationship

## Source
TryHackMe — SOC Simulator

## Scenario
Low-severity alert `1001` detected an unusual process parent-child relationship on host `win-3459`. The investigation checked the process event in Splunk and searched for child processes associated with the alerted process.

## Tools & Technologies
- SOC Simulator alert queue
- Splunk

---

# Investigation

### 1. Initial Alert / Detection

| Field | Value |
|---|---|
| Alert ID | `1001` |
| Rule | Suspicious Parent Child Relationship |
| Severity | Low |
| Host | `win-3459` |
| Process | `TrustedInstaller.exe` |
| Process ID | `3577` |
| Parent process | `services.exe` |
| Parent PID | `3506` |
| Timestamp | `2026-10-21 10:40:50.908` |
| Command line | `C:\Windows\servicing\TrustedInstaller.exe` |
| Working directory | `C:\Windows\system32\` |

<img src="https://github.com/user-attachments/assets/3b34889b-d3fe-41e2-a2a3-355422021235" width="650" alt="SOC alert 1001 details">

### 2. Triage

I searched Splunk for events from the affected host:

```spl
index=*
| search host.name="win-3459"
```

The results included a process-creation event for `TrustedInstaller.exe` with `services.exe` as its parent.

<img src="https://github.com/user-attachments/assets/bc75b163-fae8-4d6f-8917-a9bf1b073568" width="650" alt="Splunk process event on win-3459">

### 3. Evidence Analysis

`TrustedInstaller.exe` is associated with Windows servicing. In this event, its command line points to the expected servicing directory:

`C:\Windows\servicing\TrustedInstaller.exe`

I then searched for events where the process was the parent:

```spl
index=*
| search host.name="win-3459" process.parent.pid="3577"
```

No matching events were returned. This means no further child-process events were found by this query in the available data; it does not by itself prove the process was benign.

### 4. Correlation

The available evidence shows one process-creation event:

- **10:40:50.908, 21 October 2026:** `services.exe` launched `TrustedInstaller.exe` on `win-3459`.
- **Follow-up search:** no child-process events matching parent PID `3577` were found.

The event is consistent with Windows servicing and analysis show no following processes created by the suspicious process.

### 5. Threat Intelligence

No external threat-intelligence lookup was recorded or needed for the current assessment. The available evidence is local process telemetry and does not include a file hash, signature-verification result, or reputation result.

### 6. MITRE ATT&CK

No ATT&CK technique is confidently mapped from the available evidence. An unusual parent-child relationship alone does not establish malicious behavior.

## Findings

The alert flagged `TrustedInstaller.exe` being launched by `services.exe`. The executable path matches the expected Windows servicing location, and the follow-up query returned no child-process events for PID `3577`. Based on the evidence available in this lab, I classified the event as benign.

**Verdict:** False Positive *(benign activity)*

**Severity:** Low *(original alert severity)*

**Affected Assets:** `win-3459`; process `TrustedInstaller.exe` (PID `3577`)

## Response & Recommendations

- Close the alert as a false positive in the lab and document the supporting process path and query results.
- In a production investigation, verify the executable's digital signature and hash, and check Windows Update / servicing logs for matching activity.
- If the path, signature, or servicing context is unexpected, investigate the process tree and nearby events before treating it as benign.
