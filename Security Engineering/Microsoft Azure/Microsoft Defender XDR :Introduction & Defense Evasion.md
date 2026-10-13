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

**<Screenshot of Microsoft Defender XDR home dashboard>**

*The Defender portal provides a centralized view of security incidents, alerts, and available investigation tools.*

## 2. Understanding Defense Evasion

Defense Evasion is a tactic in the MITRE ATT&CK framework describing techniques attackers use to avoid detection or bypass security controls during their operations.

Attackers may attempt to hide malicious activity, disable security tools, manipulate logs, or disguise malicious files and processes as legitimate components.

Common Defense Evasion techniques include:

- **Impair Defenses:** Attempting to disable antivirus, endpoint protection, or logging mechanisms.
- **Masquerading:** Making malicious files, processes, or services appear legitimate.
- **Obfuscated Files or Information:** Concealing malicious scripts or code to make analysis and detection more difficult.
- **Timestomping:** Manipulating file timestamps to make suspicious activity harder to identify.
- **Signed Binary Proxy Execution:** Abusing trusted, signed system binaries to execute malicious content.

Understanding these techniques helps security analysts recognize suspicious behavior and identify potential attempts to weaken an organization's defenses.

## 3. Exploring the Microsoft Defender Portal

1. Sign in to the Microsoft Defender portal using the authorized lab account.
2. Review the main navigation pane and locate **Incidents & alerts**, **Hunting**, and the product-specific sections.
3. Open **Settings → Microsoft Defender XDR** to explore available configuration options.
4. Review **Permissions and roles** to understand how security teams control access to incidents, investigations, and administrative settings.
5. Check the available features and permissions in the lab tenant, as some capabilities depend on licensing and assigned roles.

**<Screenshot of Defender portal navigation pane>**

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

**<Screenshot of the “Attempt to turn off Microsoft Defender Antivirus protection” incident>**

*The incident overview identifies the affected device, user, severity, and related security alerts.*

**<Screenshot of the alert timeline and initiating process details>**

*The alert timeline provides process and command-line context that helps reconstruct the suspicious activity.*

## 5. Hunting for Potential Defense Evasion

In addition to reviewing generated alerts, Microsoft Defender XDR provides **Advanced Hunting**, which allows analysts to search security telemetry using Kusto Query Language (KQL).

This is useful when an alert has not already been generated or when an analyst wants to investigate whether similar behavior occurred elsewhere.

### Example: Detecting a suspicious `svchost.exe` location

`svchost.exe` is a legitimate Windows process. However, an executable with this name running from an unexpected directory may warrant investigation.

1. Navigate to **Hunting → Advanced hunting**.
2. Open the **Queries** section.
3. Browse the community query templates and locate queries related to Defense Evasion.
4. Run the following example query to identify `svchost.exe` process events recorded outside the standard Windows System32 directory.

```kusto
DeviceProcessEvents
| where FileName =~ "svchost.exe"
| where FolderPath !startswith "C:\\Windows\\System32"
```

The query uses the `DeviceProcessEvents` table to search process creation telemetry.

An unexpected file path is not proof of malicious activity. Review the process command line, parent process, device, user, file signature, and surrounding events before drawing conclusions.

**<Screenshot of Advanced Hunting with the KQL query and results>**

*Advanced Hunting can help identify potentially suspicious process execution for further investigation.*

## 6. Mitigating Defense Evasion

Microsoft Defender provides several controls that can help prevent or detect attempts to weaken security defenses.

- **Tamper Protection:** Helps prevent unauthorized changes to important Microsoft Defender security settings.
- **Microsoft Defender Antivirus:** Provides real-time protection, cloud-delivered protection, and behavioral monitoring.
- **Attack Surface Reduction (ASR) rules:** Help restrict risky behaviors, such as certain obfuscated scripts and suspicious process creation.
- **Least privilege:** Limits the ability of unauthorized users or processes to modify security configurations.
- **Automated Investigation and Remediation:** Can investigate and remediate supported threats when the required configuration and licensing are available.

For a practical exercise, I reviewed the Tamper Protection setting and the available endpoint security policies in the Defender portal.

**<Screenshot of the Tamper Protection setting>**

*Tamper Protection is an important safeguard against attempts to modify or disable security controls.*

Where permissions allow, suspicious incidents can also be investigated using additional evidence collection, automated investigation, Live Response, or device isolation. Response actions should be selected according to the incident's evidence and organizational procedures.



> **Note**:*This write-up documents a guided TryHackMe learning exercise. Screenshots and investigation findings should reflect the actual lab environment and actions performed.*
