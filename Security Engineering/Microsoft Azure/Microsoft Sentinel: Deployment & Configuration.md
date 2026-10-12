# Deploying and Preparing Microsoft Sentinel for Security Monitoring

## What is Microsoft Sentinel  
Microsoft Sentinel is a cloud-native Security Information and Event Management (SIEM) and Security Orchestration, Automation, and Response (SOAR) platform. It collects and analyzes security data from connected sources to help security teams detect threats, investigate incidents, and automate responses.  

## How Device Logs Reach Microsoft Sentinel  
Creating a Log Analytics Workspace (LAW) does not automatically collect logs from every device. Each data source must be configured to send its logs to the intended workspace using an appropriate data connector or log collection mechanism.  

## Configuring Log Collection with a Data Collection Rule (DCR)

A **Data Collection Rule (DCR)** defines which data should be collected, how it should be collected, and where it should be sent. For supported Windows and Linux endpoints, Azure Monitor Agent (AMA) works with DCRs to collect selected events and forward them to a specified Log Analytics Workspace. F.ex.: To collect Windows security events from a particular endpoint, I would install and configure AMA, create a DCR that selects the required Windows Security events, specify the destination LAW, and associate the rule with the endpoint. This allows administrators to control which devices participate in log collection and which event types are ingested, rather than collecting everything from every device. 

## Example: Detecting a Failed Windows Login

When a user enters an incorrect password on a Windows endpoint, Windows records a failed logon event in its Security event log, typically with **Event ID 4625**.

The event reaches Microsoft Sentinel through the following process:

1. **Windows endpoint:** Windows generates Event ID 4625 and records it in the local Security event log.
2. **Azure Monitor Agent:** AMA collects the event according to the configured collection rules.
3. **Data Collection Rule:** The DCR defines the events to collect and routes them to the selected Log Analytics Workspace.
4. **Log Analytics Workspace:** The event is ingested into the appropriate table, commonly `SecurityEvent` when using the relevant Windows Security Events via AMA configuration.
5. **Microsoft Sentinel:** Sentinel can query the ingested event using Kusto Query Language (KQL). An analytics rule can detect the failed login and generate an alert or incident when its configured conditions are met.

>**Note:** Ingesting a log does not automatically create a Sentinel incident. A suitable analytics rule must be configured and enabled to detect the event, and the event must satisfy that rule's conditions.



# Microsoft Sentinel Deployment & Configuration guide
## 1. Creating a Log Analytics Workspace (LAW)

A Log Analytics workspace is the central repository where Azure Monitor collects and stores log data. Microsoft Sentinel uses this workspace to query, correlate, and analyze data from connected sources. Before Sentinel can analyze security events, an appropriate workspace must be created and configured.



The workspace's region, access controls, retention requirements, and ingestion costs should be considered during deployment.

<img width="1912" height="897" alt="image" src="https://github.com/user-attachments/assets/01a891f2-53d9-4bd0-a96b-4cc7c274ba42" />
<img width="1912" height="897" alt="image" src="https://github.com/user-attachments/assets/05039201-37b3-43c8-b293-6d25360510f2" />
<img width="844" height="961" alt="image" src="https://github.com/user-attachments/assets/1cf49f09-62cf-4e40-89c5-f2d8ac69514b" />

## 2. Enabling Microsoft Sentinel

After creating the workspace, the next step is to enable Microsoft Sentinel. Sentinel uses the Log Analytics workspace as its data foundation. Enabling Sentinel adds the security monitoring capabilities needed to work with connected log sources, analytics rules, incidents, and investigation tools.

<img width="730" height="363" alt="image" src="https://github.com/user-attachments/assets/f384a33c-a28c-4b4a-8405-2360a7f54596" />
<img width="585" height="536" alt="image" src="https://github.com/user-attachments/assets/b3c1ad3e-ec74-4d6b-b584-d1704195aea6" />
<img width="1917" height="903" alt="image" src="https://github.com/user-attachments/assets/e60070d1-815c-4240-8675-d52301ff2702" />
<img width="1917" height="794" alt="image" src="https://github.com/user-attachments/assets/a4bd2b7b-48b9-4c2a-b29e-9add366744bb" />

## 3. Installing Solutions from the Content Hub

The Content hub provides a centralized location for discovering and installing Microsoft Sentinel solutions and standalone security content. Solutions can include data connectors, analytics rule templates, workbooks, hunting queries, and playbooks. These components help administrators prepare Sentinel for specific monitoring scenarios.

<img width="1634" height="706" alt="image" src="https://github.com/user-attachments/assets/cc631388-158b-439c-8a67-befc556a1ba8" />

>**Note**: Sometimes Content Hub may not load properly because of the future migration to Defender. It is possible to try to bypass it by refreshing or it can be accessed by opening Defender portal (like on screenshots below).
><img width="1920" height="950" alt="image" src="https://github.com/user-attachments/assets/4da784d1-6005-4f7e-82a2-94c41478f49a" />
<img width="1920" height="950" alt="image" src="https://github.com/user-attachments/assets/aa1f42a8-b81d-4363-b7a7-77555eae6854" />

I also explored the Threat Intelligence solution as part of the lab.

### Why is the Content hub important?

Installing relevant solutions makes prebuilt integrations and security content available without requiring every component to be created from scratch. This can accelerate deployment and provide a starting point for threat detection and investigation. However, installing a solution does not necessarily activate all its components or start collecting logs. Data connectors and other configurable content may still require additional setup.


## 4. Configuring a Data Connector and Ingesting Logs

A data connector establishes the integration between Microsoft Sentinel and a supported data source. It defines how relevant events or security data are collected and made available for analysis. Installing a connector through the Content hub does not automatically mean that it is connected.

<img width="1911" height="887" alt="image" src="https://github.com/user-attachments/assets/1589b473-94c2-4a79-8876-1bdce56916c9" />
<img width="1920" height="920" alt="image" src="https://github.com/user-attachments/assets/eb293a8f-8cb5-406f-a97d-c581e37dcd69" />
<img width="1466" height="621" alt="image" src="https://github.com/user-attachments/assets/0a059587-50ef-4642-b94d-01af414e8b51" />

In the lab, I configured the Microsoft Defender Threat Intelligence connector to import threat intelligence indicators into Sentinel.

### Understanding the log ingestion process

The relationship between the workspace, Content hub, and data connector is important:

1. **Log Analytics workspace:** Provides the destination for supported log data.
2. **Content hub solution:** Makes the relevant connector and other security content available.
3. **Data connector:** Establishes and configures the integration with the source.
4. **Data ingestion:** Events or indicators are delivered to the workspace when the integration is configured and the source provides data.
5. **Microsoft Sentinel:** Uses the ingested data for analytics, hunting, investigation, and incident generation.

Logs do not appear simply because Sentinel is enabled or a Content hub solution is installed. The data source must be configured to send data, and the selected connector must support that source. For example, endpoint logs might require an agent or endpoint integration, while Azure resource logs may require diagnostic settings to route supported logs to the workspace.

### How to verify ingestion

The connector's status and last-log-received information can provide useful indicators, but the most important verification is to confirm that data is actually present in the expected Log Analytics table. For the Threat Intelligence connector explored in this lab, the relevant table was `ThreatIntelligenceIndicator`. A successful connection may not immediately produce data. The result depends on the connector, source activity, configuration, and ingestion delay.
<img width="1501" height="735" alt="image" src="https://github.com/user-attachments/assets/bc4f3799-46a5-4cc7-9d2d-5c514cac8daa" />


## 5. Enabling an Analytics Rule

Analytics rules analyze ingested data to identify activity that may indicate a security threat. Depending on the rule and its configuration, detections can generate alerts and incidents for investigation. In the lab, I used a prebuilt rule template to configure detection of rare subscription-level operations in Azure.

### Configuration steps

1. Install the **Azure Activity** solution from the Content hub.
2. Navigate to **Analytics** in the Sentinel workspace.
3. Open the **Rule templates** tab.
4. Search for the rule *Rare subscription-level operations in Azure*.
5. Select the template and choose **Create rule**.
6. Review the rule description and its MITRE ATT&CK mappings.
7. Inspect the KQL query and understand which operations it identifies.
8. Configure the query schedule to run every hour and look back over the previous seven days.
9. Review the incident settings.
10. Select **Review + create** and save the rule.

The rule can be customized to suit the environment, but changes should be made only after understanding the detection logic and the expected results.

### Important configuration concepts

- **KQL query:** Defines the events and conditions the rule evaluates.
- **Run frequency:** Determines how often the query executes.
- **Lookup period:** Defines the historical window of data evaluated by the query.
- **Severity:** Indicates the configured severity of a detection.
- **MITRE ATT&CK mapping:** Associates the detection with relevant adversary tactics and techniques.
- **Incident settings:** Control how triggered alerts are grouped into incidents.

An enabled analytics rule is not proof that a threat has been detected. It must have suitable data to evaluate, and its results should be validated.


<img width="452" height="703" alt="image" src="https://github.com/user-attachments/assets/400ed461-6577-4c87-97a8-47f54956f7c9" />
<img width="1880" height="797" alt="image" src="https://github.com/user-attachments/assets/0269b685-d875-49a8-b488-e5ccea2d2f55" />
<img width="3321" height="1424" alt="image" src="https://github.com/user-attachments/assets/47b84606-5049-44d9-8b73-9205701be68a" />
<img width="745" height="932" alt="image" src="https://github.com/user-attachments/assets/2456eca7-bb05-4435-a011-f9c89865ab1d" />
<img width="745" height="932" alt="image" src="https://github.com/user-attachments/assets/7a25152a-48b8-46d2-bc95-130cee157184" />
<img width="1888" height="932" alt="image" src="https://github.com/user-attachments/assets/c6ed6f36-d6de-4ce3-b98f-72dee23c66e8" />



## 6. Permissions and Operational Considerations

Deploying Sentinel requires appropriate permissions at the relevant Azure resource and workspace scopes. During the lab, I worked with Microsoft Sentinel Contributor and Log Analytics Contributor permissions. However, the exact permissions required depend on the operation and the data connector being configured.

Before deploying Sentinel in a production environment, it is important to consider:

- **Least privilege:** Assign only the permissions required for each task.
- **Data source coverage:** Confirm that relevant systems are connected and sending the required events.
- **Ingestion and retention costs:** Monitor data volume and configure retention to meet operational and compliance needs.
- **Detection quality:** Review analytics rule logic, severity, schedules, and potential false positives.
- **Validation:** Confirm that expected log tables contain data and that detection rules behave as intended.
- **Incident response:** Define how incidents will be investigated, escalated, documented, and resolved.



>*Note: This file was created thanks to the resources and guidance provided by TryHackMe MS Sentinel modules describing deployment, data ingestion and detection*



