# Microsoft Defender XDR: Introduction and Defense Evasion Investigation

## 1. What Is Microsoft Defender XDR?

Microsoft Defender XDR is an Extended Detection and Response (XDR) platform that collects and correlates security signals across multiple areas of an organization's IT environment.

Unlike traditional Endpoint Detection and Response (EDR), which focuses primarily on individual endpoints, XDR combines information from endpoints, identities, email, and cloud applications to provide a broader view of potential attacks.

Microsoft Defender XDR integrates with security products such as:

- **Microsoft Defender for Endpoint:** Endpoint protection, threat detection, and investigation.
- **Microsoft Defender for Office 365:** Protection against malicious emails, attachments, and links.
- **Microsoft Defender for Identity:** Detection of suspicious activity involving on-premises identities.
- **Microsoft Defender for Cloud Apps:** Monitoring and protection of cloud application activity.

By correlating signals from these sources, Defender XDR helps security analysts investigate incidents, identify affected assets, hunt for threats, and coordinate response actions through a centralized portal.

<img width="1917" height="953" alt="image" src="https://github.com/user-attachments/assets/c2a489c1-d854-4f5c-9d69-02f9f14a4e24" />

*The Defender portal provides a centralized view of security incidents, alerts, and available investigation tools.*

## 2. Understanding Defense Evasion

Defense Evasion is a tactic in the MITRE ATT&CK framework describing techniques attackers use to avoid detection or bypass security controls during their operations.

Attackers may attempt to hide malicious activity, disable security tools, manipulate logs, or disguise malicious files and processes as legitimate components.

Common Defense Evasion techniques include:

- **Impair Defenses:** Attempting to disable antivirus, endpoint protection, or logging mechanisms.
- **Masquerading:** Making malicious files, processes, or services appear legitimate. F.ex: Faking/Double file extensions, Altering file or processes names or Modifying code signatures.
- **Obfuscated Files or Information:** Concealing malicious scripts or code to make analysis and detection more difficult.
- **Timestomping:** Manipulating file timestamps to make suspicious activity harder to identify.
- **Signed Binary Proxy Execution:** Abusing trusted, signed system binaries to execute malicious content.
<img width="1202" height="673" alt="image" src="https://github.com/user-attachments/assets/c8057e6f-a6a6-4554-86b0-679c13922ef0" />

Understanding these techniques helps security analysts recognize suspicious behavior and identify potential attempts to weaken an organization's defenses.

## 3. Exploring the Microsoft Defender Portal

Microsoft Defender XDR unified role-based access control (RBAC) is a centralized permissions management system that streamlines access management across the Defender security ecosystem. It provides a single interface for administrators to manage who can access what across the Microsoft Defender XDR. It unifies access control across endpoints, identities, email, cloud apps, and data based on assigned roles and responsibilities. It can be activated in the "workload settings".
<img width="1917" height="838" alt="image" src="https://github.com/user-attachments/assets/1d86db42-95e1-435a-b6ba-ed661f0a86f0" />
<img width="635" height="779" alt="image" src="https://github.com/user-attachments/assets/f76dfd91-e78e-4a11-a8a1-2da20e741854" />



Incidents are a collection of alerts related to a single threat or attack. They are confirmed security breaches or threats that have compromised an organization's assets, data, or systems.
<img width="1920" height="952" alt="image" src="https://github.com/user-attachments/assets/a042c2f1-101c-4874-b419-f9330fda5e56" />
<img width="1920" height="950" alt="image" src="https://github.com/user-attachments/assets/e71c355f-484c-4a84-9d03-6bce858a2667" />

Hunting is a feature in Microsoft Defender XDR that uses a query-based threat-hunting interface to allow security teams proactively hunt for and detect advanced threats within their organization's environment. 
<img width="917" height="713" alt="image" src="https://github.com/user-attachments/assets/6f56f339-8913-4ac6-b96a-2a1879b5c536" />
*Key areas of the portal used for incident investigation, threat hunting, and security configuration.*

## 4. Investigating a Defense Evasion Incident

In the TryHackMe lab, I investigated an incident named **Attempt to turn off Microsoft Defender Antivirus protection**.

The incident represents a potential attempt to impair defenses by modifying security-related configuration.

1. Navigate to **Investigation & response → Incidents & alerts → Incidents**.
2. Adjust the time range to **6 months**, as instructed in the lab.
3. Search for `Attempt to turn off Microsoft Defender Antivirus protection`.
4. Open the matching incident and review its overview.
5. Examine the affected device, user account, timestamps, and associated alerts.
6. Open the **Alert timeline** to investigate the sequence of events.
7. Review the initiating process, command-line information, and relevant registry changes.
8. Use the incident's related entities and process details to understand how the activity was initiated.

The lab material describes an example involving `reg.exe`, initiated by `cmd.exe`, attempting to modify a registry setting related to Microsoft Defender Antivirus.

This type of behavior is suspicious because attempts to weaken endpoint protection can create opportunities for further malicious activity. However, the alert should be investigated in context before concluding that the activity is malicious.

<img width="1920" height="954" alt="image" src="https://github.com/user-attachments/assets/bfe7d492-9089-4eff-83b8-2a3018626183" />
<img width="1522" height="890" alt="image" src="https://github.com/user-attachments/assets/94a0fc27-a272-458b-8c91-e477bb73c6da" />
After investigation, if this alert is considered a malicious event, the following can be done on the device to mitigate the threat and prevent further escalation.

    Collect Investigation Package for further analysis.
    Start Microsoft Defender XDR Automated Investigation.
    Initiate Live Response Session to see the registry changes.
    Isolate the device to prevent lateral movement or the attack from getting access to other devices.
<img width="898" height="712" alt="image" src="https://github.com/user-attachments/assets/10de6e1c-e14f-4ca1-b0be-9d6588955319" />


## 5. Hunting for Potential Defense Evasion

In addition to reviewing generated alerts, Microsoft Defender XDR provides **Advanced Hunting**, which allows analysts to search security telemetry using Kusto Query Language (KQL). This is useful when an alert has not already been generated or when an analyst wants to investigate whether similar behavior occurred elsewhere.

### Example: Detecting a suspicious `svchost.exe` location
`svchost.exe` is a legitimate Windows process. However, an executable with this name running from an unexpected directory may warrant investigation.

```kusto
DeviceProcessEvents
| where FileName =~ "svchost.exe"
| where FolderPath !startswith "C:\\Windows\\System32"
```

The query uses the `DeviceProcessEvents` table to search process creation telemetry. An unexpected file path is not proof of malicious activity. Review the process command line, parent process, device, user, file signature, and surrounding events before drawing conclusions.

<img width="3051" height="927" alt="image" src="https://github.com/user-attachments/assets/a3e8a88d-ddd0-4dd6-8f80-1282f5f4d8c0" />

*Advanced Hunting can help identify potentially suspicious process execution for further investigation.*

## 6. Mitigating Defense Evasion & Prevention

Microsoft Defender provides several controls that can help prevent or detect attempts to weaken security defenses.

- **Tamper Protection:** Helps prevent unauthorized changes to important Microsoft Defender security settings.
- **Microsoft Defender Antivirus:** Provides real-time protection, cloud-delivered protection, and behavioral monitoring.
- **Attack Surface Reduction (ASR) rules:** Help restrict risky behaviors, such as certain obfuscated scripts and suspicious process creation.
- **Least privilege:** Limits the ability of unauthorized users or processes to modify security configurations.
- **Automated Investigation and Remediation:** Can investigate and remediate supported threats when the required configuration and licensing are available.

    Ensure the following Microsoft Defender for endpoint Attack Surface Reduction (ASR) Rules are in block mode to restrict Defense evasion activities.

> **Note**: ASR rules are only supported on Windows operating systems.

        Block Office applications from injecting code into other processes: To prevent common Defense Evasion techniques where malicious Office macros are injected into trusted processes.
        Block execution of potentially obfuscated scripts: To prevent execution of highly obfuscated scripts from running.
        Block untrusted and unsigned processes that run from USB: To prevent malware execution from removable drives, which is a common way to bypass security tools.
        Block all Office applications from creating child processes: This will prevent malware from launching scripts or binaries through Office apps like Word or Excel files.
        Block executable content from email and webmail: This will prevent users from running malicious attachments from campaigns that may lead to Defense Evasion.
<img width="1526" height="539" alt="image" src="https://github.com/user-attachments/assets/1e5f6a84-ae59-4208-a433-863ddeb591cd" />





> **Note**:*This write-up documents a guided TryHackMe learning exercise. Screenshots and investigation findings should reflect the actual lab environment and actions performed.*
