>**Note**: TODO: Try Hack ME MS Sentinel: Deploy, Ingest Data, Detect
# Deployment: 
>**Note**: To enable Microsoft Sentinel, Microsoft Sentinel Contributor permissions are required at the resource group level where the Microsoft Sentinel workspace resides.



[Room was created thanks to the resources and guidance provided by TryHackMe MS Sentinel modules describing deployment, data ingestion and detection]

<img width="1916" height="962" alt="image" src="https://github.com/user-attachments/assets/d9215406-233c-4bab-a84a-ced22be9fb9c" />


1.) Creating Log Analytics Workspace (LAW):
<img width="1912" height="897" alt="image" src="https://github.com/user-attachments/assets/01a891f2-53d9-4bd0-a96b-4cc7c274ba42" />
<img width="1912" height="897" alt="image" src="https://github.com/user-attachments/assets/05039201-37b3-43c8-b293-6d25360510f2" />

<img width="844" height="961" alt="image" src="https://github.com/user-attachments/assets/1cf49f09-62cf-4e40-89c5-f2d8ac69514b" />





2.) Add Microsoft Sentinel to the Workspace:
law:
<img width="730" height="363" alt="image" src="https://github.com/user-attachments/assets/f384a33c-a28c-4b4a-8405-2360a7f54596" />

<img width="585" height="536" alt="image" src="https://github.com/user-attachments/assets/b3c1ad3e-ec74-4d6b-b584-d1704195aea6" />
<img width="1917" height="903" alt="image" src="https://github.com/user-attachments/assets/e60070d1-815c-4240-8675-d52301ff2702" />
3.) Check settings
<img width="1917" height="794" alt="image" src="https://github.com/user-attachments/assets/a4bd2b7b-48b9-4c2a-b29e-9add366744bb" />

3.5.) How to specify from which devices logs are sent to this specific created LAW? How does it work that after LAW, and then after installing content hub logs suddenly occure?


4.) Installing content hub solutions:
<img width="1634" height="706" alt="image" src="https://github.com/user-attachments/assets/cc631388-158b-439c-8a67-befc556a1ba8" />

Microsoft Defender:
<img width="1920" height="950" alt="image" src="https://github.com/user-attachments/assets/4da784d1-6005-4f7e-82a2-94c41478f49a" />
<img width="1920" height="950" alt="image" src="https://github.com/user-attachments/assets/aa1f42a8-b81d-4363-b7a7-77555eae6854" />

Data Connector:
<img width="1917" height="957" alt="image" src="https://github.com/user-attachments/assets/251637f2-f97a-42fe-ab26-cea216a8666a" />
<img width="857" height="436" alt="image" src="https://github.com/user-attachments/assets/368c8b51-63d1-4612-a727-d757ea85bc12" />



Or hub Solutions may be in azure portal:
<img width="1911" height="887" alt="image" src="https://github.com/user-attachments/assets/1589b473-94c2-4a79-8876-1bdce56916c9" />
<img width="1920" height="920" alt="image" src="https://github.com/user-attachments/assets/eb293a8f-8cb5-406f-a97d-c581e37dcd69" />

<img width="1466" height="621" alt="image" src="https://github.com/user-attachments/assets/0a059587-50ef-4642-b94d-01af414e8b51" />
<img width="1501" height="735" alt="image" src="https://github.com/user-attachments/assets/bc4f3799-46a5-4cc7-9d2d-5c514cac8daa" />




<img width="452" height="703" alt="image" src="https://github.com/user-attachments/assets/400ed461-6577-4c87-97a8-47f54956f7c9" />
<img width="1880" height="797" alt="image" src="https://github.com/user-attachments/assets/0269b685-d875-49a8-b488-e5ccea2d2f55" />

<img width="3321" height="1424" alt="image" src="https://github.com/user-attachments/assets/47b84606-5049-44d9-8b73-9205701be68a" />
<img width="745" height="932" alt="image" src="https://github.com/user-attachments/assets/2456eca7-bb05-4435-a011-f9c89865ab1d" />

<img width="745" height="932" alt="image" src="https://github.com/user-attachments/assets/7a25152a-48b8-46d2-bc95-130cee157184" />

<img width="1888" height="932" alt="image" src="https://github.com/user-attachments/assets/c6ed6f36-d6de-4ce3-b98f-72dee23c66e8" />








# Deploying and Preparing Microsoft Sentinel for Security Monitoring

**Project type:** Hands-on SIEM deployment lab  
**Platform:** Microsoft Azure, Microsoft Sentinel, Log Analytics  
**Learning environment:** TryHackMe  
**Objective:** Deploy Microsoft Sentinel, configure log ingestion, install security content, and enable an analytics rule to detect potentially suspicious activity.

## How Device Logs Reach Microsoft Sentinel

Creating a Log Analytics Workspace (LAW) does not automatically collect logs from every device. Each data source must be configured to send its logs to the intended workspace using an appropriate data connector or log collection mechanism.

### 1. Configuring Log Collection with a Data Collection Rule (DCR)

A **Data Collection Rule (DCR)** defines which data should be collected, how it should be collected, and where it should be sent. For supported Windows and Linux endpoints, Azure Monitor Agent (AMA) works with DCRs to collect selected events and forward them to a specified Log Analytics Workspace.

For example, to collect Windows security events from a particular endpoint, I would install and configure AMA, create a DCR that selects the required Windows Security events, specify the destination LAW, and associate the rule with the endpoint.

This allows administrators to control which devices participate in log collection and which event types are ingested, rather than collecting everything from every device.

### 2. Example: Detecting a Failed Windows Login

When a user enters an incorrect password on a Windows endpoint, Windows records a failed logon event in its Security event log, typically with **Event ID 4625**.

The event reaches Microsoft Sentinel through the following process:

1. **Windows endpoint:** Windows generates Event ID 4625 and records it in the local Security event log.
2. **Azure Monitor Agent:** AMA collects the event according to the configured collection rules.
3. **Data Collection Rule:** The DCR defines the events to collect and routes them to the selected Log Analytics Workspace.
4. **Log Analytics Workspace:** The event is ingested into the appropriate table, commonly `SecurityEvent` when using the relevant Windows Security Events via AMA configuration.
5. **Microsoft Sentinel:** Sentinel can query the ingested event using Kusto Query Language (KQL). An analytics rule can detect the failed login and generate an alert or incident when its configured conditions are met.

**Important:** Ingesting a log does not automatically create a Sentinel incident. A suitable analytics rule must be configured and enabled to detect the event, and the event must satisfy that rule's conditions.



## 1. Project Overview

Microsoft Sentinel is a cloud-native Security Information and Event Management (SIEM) and Security Orchestration, Automation, and Response (SOAR) platform. It collects and analyzes security data from connected sources to help security teams detect threats, investigate incidents, and automate responses.

In this lab, I explored the initial deployment and configuration of Microsoft Sentinel, including the creation of a Log Analytics workspace, installation of Content hub solutions, configuration of a data connector, and activation of an analytics rule.

The goal was to understand the basic workflow required to prepare a SIEM environment for security monitoring.

## 2. Creating a Log Analytics Workspace (LAW)

A Log Analytics workspace is the central repository where Azure Monitor collects and stores log data. Microsoft Sentinel uses this workspace to query, correlate, and analyze data from connected sources.

Before Sentinel can analyze security events, an appropriate workspace must be created and configured.

### Deployment steps

1. Sign in to the Microsoft Azure portal.
2. Search for **Log Analytics workspaces**.
3. Select **Create**.
4. Choose the appropriate subscription and resource group.
5. Specify a workspace name and Azure region.
6. Review the configuration and select **Create**.
7. Open the deployed workspace and verify that it is available.

### Why is the workspace important?

- **Centralized logging:** Stores logs from multiple connected data sources.
- **Querying and investigation:** Supports Kusto Query Language (KQL) for searching and analyzing events.
- **Sentinel integration:** Provides the workspace that Microsoft Sentinel uses for security analytics.
- **Data management:** Allows administrators to configure retention, access permissions, and other workspace settings.

The workspace's region, access controls, retention requirements, and ingestion costs should be considered during deployment.

**Screenshots to include:**

[SCREENSHOT 1 — Log Analytics workspace overview]

*Caption: The deployed Log Analytics workspace in Azure, showing its name and overview.*

[SCREENSHOT 2 — Create Log Analytics workspace wizard]

*Caption: Workspace creation configuration, including subscription, resource group, name, and region.*

## 3. Enabling Microsoft Sentinel

After creating the workspace, the next step is to enable Microsoft Sentinel.

Sentinel uses the Log Analytics workspace as its data foundation. Enabling Sentinel adds the security monitoring capabilities needed to work with connected log sources, analytics rules, incidents, and investigation tools.

### Deployment steps

1. Search for **Microsoft Sentinel** in the Azure portal.
2. Select **Create** or **Add**, depending on the portal interface.
3. Select the Log Analytics workspace created previously.
4. Enable Microsoft Sentinel for that workspace.
5. Open the Sentinel workspace and verify that its overview is accessible.

**Screenshot to include:**

[SCREENSHOT 3 — Microsoft Sentinel workspace overview]

*Caption: Microsoft Sentinel enabled and associated with the Log Analytics workspace.*

## 4. Installing Solutions from the Content Hub

The Content hub provides a centralized location for discovering and installing Microsoft Sentinel solutions and standalone security content.

Solutions can include data connectors, analytics rule templates, workbooks, hunting queries, and playbooks. These components help administrators prepare Sentinel for specific monitoring scenarios.

### Deployment steps

1. Open the Microsoft Sentinel workspace.
2. Navigate to **Content management → Content hub**.
3. Search for the **Microsoft Entra ID** solution.
4. Open the solution and select **Install**.
5. Wait for installation to finish.
6. Review the installed content and its configuration requirements.

I also explored the Threat Intelligence solution as part of the lab.

### Why is the Content hub important?

Installing relevant solutions makes prebuilt integrations and security content available without requiring every component to be created from scratch. This can accelerate deployment and provide a starting point for threat detection and investigation.

However, installing a solution does not necessarily activate all its components or start collecting logs. Data connectors and other configurable content may still require additional setup.

**Screenshots to include:**

[SCREENSHOT 4 — Content hub solution catalogue]

*Caption: Available solutions and standalone content in Microsoft Sentinel's Content hub.*

[SCREENSHOT 5 — Microsoft Entra ID solution details]

*Caption: Solution details showing the types of content provided by the Microsoft Entra ID package.*

[SCREENSHOT 6 — Installed solution status]

*Caption: Successful installation of the solution and review of its available content.*

## 5. Configuring a Data Connector and Ingesting Logs

A data connector establishes the integration between Microsoft Sentinel and a supported data source. It defines how relevant events or security data are collected and made available for analysis.

Installing a connector through the Content hub does not automatically mean that it is connected.

### Deployment steps

1. Open the installed solution and select **Manage** where available.
2. Locate the relevant data connector.
3. Open its configuration page.
4. Review the prerequisites and required permissions.
5. Follow the connector-specific configuration instructions.
6. Select **Connect** or complete the required setup.
7. Check the connection status and verify whether data is being received.

In the lab, I configured the Microsoft Defender Threat Intelligence connector to import threat intelligence indicators into Sentinel.

### Understanding the log ingestion process

The relationship between the workspace, Content hub, and data connector is important:

1. **Log Analytics workspace:** Provides the destination for supported log data.
2. **Content hub solution:** Makes the relevant connector and other security content available.
3. **Data connector:** Establishes and configures the integration with the source.
4. **Data ingestion:** Events or indicators are delivered to the workspace when the integration is configured and the source provides data.
5. **Microsoft Sentinel:** Uses the ingested data for analytics, hunting, investigation, and incident generation.

Logs do not appear simply because Sentinel is enabled or a Content hub solution is installed. The data source must be configured to send data, and the selected connector must support that source.

For example, endpoint logs might require an agent or endpoint integration, while Azure resource logs may require diagnostic settings to route supported logs to the workspace.

### How to verify ingestion

The connector's status and last-log-received information can provide useful indicators, but the most important verification is to confirm that data is actually present in the expected Log Analytics table.

For the Threat Intelligence connector explored in this lab, the relevant table was `ThreatIntelligenceIndicator`.

A successful connection may not immediately produce data. The result depends on the connector, source activity, configuration, and ingestion delay.

**Screenshots to include:**

[SCREENSHOT 7 — Threat Intelligence data connector configuration]

*Caption: Connector configuration and prerequisites for connecting Microsoft Defender Threat Intelligence.*

[SCREENSHOT 8 — Connected data connector]

*Caption: Connector connection status and the data types expected to be ingested.*

[SCREENSHOT 9 — Log Analytics table containing ingested data]

*Caption: Verification of the ThreatIntelligenceIndicator table in the workspace.*

## 6. Enabling an Analytics Rule

Analytics rules analyze ingested data to identify activity that may indicate a security threat. Depending on the rule and its configuration, detections can generate alerts and incidents for investigation.

In the lab, I used a prebuilt rule template to configure detection of rare subscription-level operations in Azure.

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

**Screenshots to include:**

[SCREENSHOT 10 — Analytics rule template]

*Caption: Selecting the Rare subscription-level operations in Azure detection template.*

[SCREENSHOT 11 — KQL query and scheduling settings]

*Caption: Reviewing the detection logic and configuring an hourly execution schedule with a seven-day lookback.*

[SCREENSHOT 12 — Active analytics rule]

*Caption: The saved analytics rule displayed in the Active rules list.*

## 7. Configuring an Automation Rule

Automation rules can perform actions when specified incident-related conditions are met. They help standardize incident handling and reduce repetitive manual tasks.

As an additional exercise, I configured an automation rule named **Tag Rule** to add the `subscription` tag when an incident is created.

### Configuration steps

1. Open the analytics rule's automated response settings or the relevant automation rules interface.
2. Create a new automation rule.
3. Set the trigger to **When incident is created**.
4. Add the action **Add tags**.
5. Specify the tag `subscription`.
6. Review the configuration and save the rule.

This exercise demonstrates a basic incident-management workflow. Tagging can help organize incidents, but the tag should represent a meaningful classification in a real environment.

**Screenshot to include:**

[SCREENSHOT 13 — Automation rule configuration]

*Caption: Automation rule configured to add a tag to newly created incidents.*

## 8. Permissions and Operational Considerations

Deploying Sentinel requires appropriate permissions at the relevant Azure resource and workspace scopes.

During the lab, I worked with Microsoft Sentinel Contributor and Log Analytics Contributor permissions. However, the exact permissions required depend on the operation and the data connector being configured.

Before deploying Sentinel in a production environment, it is important to consider:

- **Least privilege:** Assign only the permissions required for each task.
- **Data source coverage:** Confirm that relevant systems are connected and sending the required events.
- **Ingestion and retention costs:** Monitor data volume and configure retention to meet operational and compliance needs.
- **Detection quality:** Review analytics rule logic, severity, schedules, and potential false positives.
- **Validation:** Confirm that expected log tables contain data and that detection rules behave as intended.
- **Incident response:** Define how incidents will be investigated, escalated, documented, and resolved.

These considerations help move a Sentinel deployment from initial configuration toward a maintainable security monitoring environment.

## 9. Skills Demonstrated

Through this hands-on lab, I practised the following skills relevant to a Junior Security Engineer role:

- Deploying a Log Analytics workspace and enabling Microsoft Sentinel.
- Exploring and installing Microsoft Sentinel Content hub solutions.
- Understanding the purpose and configuration of data connectors.
- Verifying data ingestion into Log Analytics tables.
- Reviewing KQL-based analytics rule logic.
- Configuring detection schedules and incident settings.
- Mapping detections to the MITRE ATT&CK framework.
- Creating a basic automation rule for incident tagging.
- Understanding the importance of RBAC, least privilege, and data retention.

## 10. Conclusion

This lab provided practical experience with the foundational components of Microsoft Sentinel, from workspace preparation and solution installation to data ingestion and analytics rule configuration.

It also reinforced an important operational principle: a SIEM is useful only when relevant data is successfully ingested, detections are configured appropriately, and resulting alerts and incidents can be investigated.

**Next steps:** Extend the lab by connecting an additional log source, writing a custom KQL detection, validating a detection with sample events, and documenting the investigation process.

*Portfolio note: This project documents a guided TryHackMe lab. Any screenshots, configurations, and results presented should reflect the steps actually completed in the lab environment.*



