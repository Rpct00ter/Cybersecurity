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



























Deploy:
Role: You are logged in with the following Azure job function role assignments:

    Log Analytics Contributor
    Microsoft Sentinel Contributor

Lab scenario: A product needs to be utilized as part of organizational security policies. Your initial assignment is to onboard Microsoft Sentinel for the organization. You will:

    First, create a Log Analytics workspace.
    Then, enable/onboard Microsoft Sentinel to that workspace.

Access your lab by clicking the Cloud Details button below in conjunction with the lab instructions from Task 4:

Step 1: Create a Log Analytics Workspace (LAW)
Walkthrough (Click to see)

    Log into the Azure portal using your lab credentials (make sure you first logged out of any previous lab account).
    Using the top search bar, search for "Sentinel" and select "Microsoft Sentinel" in the dropdown.

    Shows a search result for "sentinel" in the Microsoft Azure portal. The search results are categorized into "All," "Services," and "Resources." Under "Services," there is one result: "Microsoft Sentinel."
    Click Create Microsoft Sentinel.
    On the Add Microsoft Sentinel to a workspace page, click Create a new workspace.
    Create a Log Analytics workspace (LAW) instance with the following values (your values might differ from the below screenshot):
        Subscription: Az-Subs-B2C-1
        Resource Group: rg-AZURE_LAB_ID
        Name: law-AZURE_LAB_ID
        Region: East US

        A Microsoft Azure interface for creating a Log Analytics workspace. The interface is under the "Basics" tab and includes fields for project details and instance details.
    Click Review + Create followed by Create again.
    Wait for the LAW deployment.
    Confirm that the new LAW is listed on the Add Microsoft Sentinel to a workspace page (you might need to refresh).

Step 2: Add Microsoft Sentinel to the workspace
Walkthrough (Click to see)

    Select recently created LAW, and click Add.

    Microsoft Azure portal showing the "Add Microsoft Sentinel to a workspace" page. The page includes options to create a new workspace and refresh the list.
    Wait for Microsoft Sentinel to be added to the LAW.
    Click OK after the Microsoft Sentinel free trial activated message.
    Browse through the Get Started page to get an idea of what's to come next.

    Microsoft Sentinel interface within Microsoft Azure. The interface is on the "News & guides" section under the "Get started" tab.

    Go to General -> Overview on the left hand side to see the tiles - Incidents, Data, Automation and Analytics.

    Microsoft Sentinel interface within Microsoft Azure. The interface is on the "Overview" section showing zero incidents.
    Confirm that no data connectors or incidents have been found yet, as this is a brand-new deployment.


Roles:
Before we dive deep into Microsoft Sentinel's specific features, it might be worthwhile to understand some Sentinel-specific roles and the scopes under which team members work.

    Microsoft Sentinel Reader
        View data, incidents, workbooks, and other Microsoft Sentinel resources
    Microsoft Sentinel Responder  (in addition to the above Sentinel Reader)
        Manage incidents
    Microsoft Sentinel Contributor (in addition to the above Sentinel Responder)
        Install/Update solutions from the Content hub.
        Create and edit workbooks, analytics rules, and other Microsoft Sentinel resources
    Microsoft Sentinel Playbook Operator
        List, view, and manually run playbooks
    Microsoft Sentinel Automation Contributor (not for user accounts)
        Allows Microsoft Sentinel to add playbooks to automation rules

Looking from the Microsoft SOC team and analysts' perspective, team members will be assigned either or a combination of the above roles (except the last one!) to perform their daily tasks. Below could be a sample distribution of Microsoft Sentinel-specific roles to SOC team members:

Stakeholders, SOC managers, and more.
	

Microsoft Sentinel Reader

Security analysts, incident responders (usually SOC Level-1 Analysts)
	

Microsoft Sentinel Responder

Security engineers, Fusion Analytics team members (usually SOC Level-2 Analysts)

Install and manage Solutions using Content Hub

Create and delete workbooks
	

Microsoft Sentinel Contributor
Automation team members (usually SOC Level-1 Analysts)

Automate responses to threats with playbooks
	

Microsoft Sentinel Playbook Operator

As a best practice, try to assign these roles to the Resource Group where Microsoft Sentinel workspace resides, as there might be other resources in that resource group which support the functionality of Microsoft Sentinel.

Given that Microsoft Sentinel is an Azure service add-on to Log Analytics workspace, while working with Sentinel, there will be two sets of settings that you need to maintain.

    Microsoft Sentinel settings
    Log Analytics workspace settings

That's why some functionality changes in Microsoft Sentinel require you to modify underlying Log Analytics workspace settings.
Data Retention

Although Microsoft Sentinel has many individual features and specific settings, let's mention here one of the most important ones: Data Retention. It is also important to note that Microsoft Sentinel is billed based on the volume of data ingested for analysis.

These two tie together, and there is a correlation between them, i.e., the more data you have—or keep in the Log Analytics workspace—the more relevant and accurate threat analysis you can perform.

Microsoft Sentinel settings can be found at Microsoft Sentinel > Select the workspace > Configuration > Settings.

Microsoft Sentinel interface within Microsoft Azure. The interface is on the "settings" section.

To modify Data Retention, we need to go to Workspace settings since data is stored in the underlying Log Analytics workspace for Microsoft Sentinel.

A settings menu with the "Workspace settings" tab highlighted, featuring options like "Entity behavior analytics," "Anomalies," "Workspace manager configuration," "Playbook permissions," "How do we use your data?," "Auditing and health monitoring," and "Remove Microsoft Sentinel."
Data Retention settings are tucked into Usage and estimated costs under Settings.

"Usage and estimated costs" section within a Log Analytics workspace in Microsoft Azure.

Default data retention is 31 days, and due to organizational policies, data governance, and other requirements, you might need to increase it. Note that longer retention will incur additional charges as the Microsoft Sentinel billing model is based on log data storage. Depending on how many data connectors you have (we will get to Data Connectors soon) and the amount of log data they are pouring into LAW, sometimes these logs can grow exponentially, which, in return, will be reflected in the costs. Hence, it is important to monitor the organization's log usage details regularly.

Data Sources and Data Connectors

Data connectors are used to connect log data to various data sources. As of this writing, several hundred data sources could send data to Microsoft Sentinel, and this number keeps growing each day. Hence, there are several hundreds of associating data connectors since these connectors are specific to the data sources, data types and data formats of the ingested log data.

Displays four categories with corresponding numbers: Solutions: 342 (with a cloud icon), Standalone contents: 273 (with a grid icon), Installed: 0 (with a green checkmark icon), Updates: 0 (with a blue upward arrow icon).

Microsoft Sentinel includes various data connectors for both Microsoft and third-party products to assist in onboarding your data. These data connectors are essential for integrating and ingesting data from various sources, such as Microsoft Defender, , Cisco, etc., into Microsoft Sentinel, providing real-time threat detection and analysis.

Data connectors are available in various ways:

    Out-of-the-box (OOTB)
    Packaged with Content hub solutions

Microsoft Sentinel Data Connectors page in Microsoft Azure. The left sidebar contains navigation options, including Overview, Logs, News & guides, Search, Incidents, Workbooks, Hunting, Notebooks, Entity behavior, Threat intelligence, MITRE ATT&CK, Content hub, Repositories, Community, Workspace manager, Data connectors, Analytics, and Watchlist. The main section displays "0 Connectors" and "0 Connected."

In the earlier years of Microsoft Sentinel, the only option to ingest data was to use data connectors. However, as the number of data connectors increased, the need to organize them based on their type and package them into a solution-like entity emerged. Out of this need, the Content hub concept was introduced. The Content hub provides a packaged way of installing and enabling data connectors for various data sources.

Microsoft Sentinel Content hub is a place to centrally discover, install, enable, and manage out-of-the-box content and solutions for Microsoft Sentinel. It is usually the starting point for any Microsoft Sentinel administrator after the initial deployment.

Content hub provides:

    Centralized in-product search for solutions
    Single-step deployment
    Installation of out-of-the-box solutions and content in Microsoft Sentinel

Content hub can also be referred to as a Security Information and Event Management () library that enables you to ingest data, monitor, alert, hunt, investigate, respond, and connect with different products, platforms, and services in Microsoft Sentinel.

A "Content hub" interface in a software platform displays 342 solutions and 273 standalone contents, none installed or updated. A table lists contents with columns for Title, Status, Source, Provider, Support, Category, and Type.
Content Source: Standalone vs. Solutions

It is important to note that content sources can be standalone content or solutions.

A user interface for selecting content sources. There is a search bar labeled "Search values" at the top. Below the search bar, there are three options with checkboxes: "All," "Solution," and "Standalone."

Microsoft Sentinel Solutions are packages of content, such as data connectors, workbooks, analytic rules, playbooks, etc., that fulfil an end-to-end product, domain, or industry vertical scenario in Microsoft Sentinel. We will explore the details of these content types in some upcoming rooms, but for now, we need to get a good grasp of how they are organized in Microsoft Sentinel.

Content type selection menu with a search bar labeled "Search values" at the top. Options include: "All," "Analytics rule (1387)," "Data connector (316)," "Hunting query (834)," "Parser (299)," "Playbook (411)," "Watchlist (27)," and "Workbook (301)." All checkboxes are selected.

As an example, let's take a peek at the Microsoft Entra ID solution.

Microsoft Entra ID solution for Microsoft Sentinel including description.

As seen above, the solution description shows what this solution is made of in terms of content types, i.e., what the package contains:

    1 Data connector - for ingesting log data
    63 Analytics rules - for analyzing the threats and creating incidents
    11 Playbooks - for automation

This also means that, when installed, this solution will deploy the above-listed contents into your Microsoft Sentinel environment. Currently, with no Content hub solution installed, you won't see any data connectors available to connect.

"Microsoft Sentinel | Data connectors" interface. The selected workspace is 'law-20240213-123'. The interface includes a search bar, options for "Refresh" and "Guides & Feedback," and a notification that "Azure Active Directory (Azure AD) is now Microsoft Entra ID."

With this out of the way, we can go ahead and install our first Content hub solution in upcoming tasks.

Context: Your company recently deployed Microsoft Sentinel. However, no Content hub solutions have been installed yet.

Role: You are logged in as:

    Microsoft Sentinel Contributor
    Log Analytics Contributor

The Azure portal shows the "Job function roles" tab, filtered for roles related to "Microsoft Sentinel," including "Microsoft Sentinel Automation Contributor," "Microsoft Sentinel Contributor," "Microsoft Sentinel Playbook Operator," "Microsoft Sentinel Reader," and "Microsoft Sentinel Responder."

Lab scenario: Following the initial deployment of Microsoft Sentinel, you are tasked with installing a Content hub solution.

    First, make sure Sentinel is enabled for the workspace
    Then, you will install a Content hub solution
    Finally, you will review the results

Access your lab by clicking the Cloud Details button below in conjunction with the lab instructions from Task 4:

Please make sure to have performed these actions on the Cloud Details pop-up:
Environment Tab 	Click Join Lab
Actions Tab 	Click Deploy Lab

Step 1: Install a Content hub solution
Walkthrough (Click to see)

    Log in to Azure portal using your lab credentials (make sure you first logged out of any previous lab account)
    Go to Microsoft Sentinel and open the available workspace

    Under Content management, select Content hub

Content management menu with three options: Content hub, Repositories (Preview), and Community.

    Note that currently, no standalone contents or packaged solutions are installed

Microsoft Sentinel Content Hub. The statistics at the top indicate 342 Solutions and 273 Standalone contents available, with 0 Installed and 0 Updates.

    Search for solution: Microsoft Entra ID, select it and click Install

User interface for selecting and installing software in Microsoft Sentinel. "Microsoft Entra ID" is highlighted as "FEATURED." The description includes provider, support, version, and a note to read release notes before installation.

    Wait for the content hub installation to complete

A status bar with the following: checkbox icon, "Microsoft Entra ID" text with diamond logo, pink "FEATURED" label, blue circular "In progress" icon with arrows.A status bar with the following: checkbox icon, "Microsoft Entra ID" text with diamond logo, pink "FEATURED" label, green circular "Installed" icon checked.

Step 2: Review the results
Walkthrough (Click to see)

    Select the recently installed Content hub solution

    Review the Content hub solution details on the right pane

Microsoft Entra ID solution for Microsoft Sentinel.

    Note that various content types are deployed by this Content hub solution:
        63 Analytics Rules
        1 Data Connector
        11 Playbooks
        2 Workbooks (dashboards)

    Click Manage to see the package details for the Content hub solution

A Microsoft Sentinel Content hub interface displaying content items for Microsoft Entra ID. It shows 77 installed content items and 75 configuration-needed.

    Under Content name, click on Microsoft Entra ID

    Review the data connector status

A "Data connectors" interface from a software application. It displays a summary with 1 connector available and 0 connected.

    Note that the data connector is installed. However, it is still not connected yet. We will do this later on
    After completing this lab, now you should be able to:
        Install a Content hub solution
        Review an installed Content hub solution and it's connector status
Now that we have a Content hub solution installed, which comes with a data connector, the next step is to connect and configure it so that log data starts flowing into the Log Analytics workspace for Microsoft Sentinel to analyze and correlate.

To connect a data connector, the concept is to go to its connector page and follow the specific instructions for that connector.

Microsoft Sentinel Data connectors page with the "Microsoft Entra ID" connector selected.

Let's explore some sections of a connector page. There will be lots of information on connector pages; note that they are all specific to that particular connector. The main sections to note are:

    Description: What the connector is
    Data types: Log tables that will be populated in the Log Analytics workspace once the connector is connected, i.e., types of log data
    Instructions:
        Prerequisites: Permissions you need to have to connect the data
        Configuration: Specific config steps for the connector

Microsoft Sentinel Data connectors page for Microsoft Entra ID.

Note: As per prerequisites, your current assigned role may not always have the necessary permissions to connect a data connector, although you might have a Microsoft Sentinel-specific role assigned, such as Microsoft Sentinel Contributor. It's a good practice to familiarize yourself with Microsoft Sentinel roles and permissions.

Depending on a specific connector, you may need extra permissions for different scopes, which are detailed in the Prerequisites section. If you don't have these permissions, be sure to contact your cloud administrator.

In the next task, we will connect and configure a data connector.
Context: Your company recently deployed Microsoft Sentinel and a Content hub solution is installed. A data connector is populated; however, it has not been connected yet.

Role: You are logged in as:

    Microsoft Sentinel Contributor
    Log Analytics Contributor

Lab scenario: Following the installation of the Content hub solution, you are tasked with configuring data connectors.

    First, you will install another Content hub solution: Threat Intelligence
    Then, you will connect the data connector
    Finally, you will review the ingested Threat Intelligence data

First, leave the lab we started in Task 5 by pressing the Leave Lab button on the Cloud Details pop-up from Task 5. Then, access your new lab by clicking the Cloud Details button below in conjunction with the lab instructions from Task 4:

Please make sure to have performed these actions on the Cloud Details pop-up:
Environment Tab 	Click Join Lab
Actions Tab 	Click Deploy Lab

Step 1: Install the Threat Intelligence Content hub solution
Walkthrough (Click to see)

    Log in to Azure portal using your lab credentials (make sure you first logged out of any previous lab account)
    Go to Microsoft Sentinel and open the available workspace
    Under Content Management, Select Content hub
    Search for Threat Intelligence and select it
    Install the Threat Intelligence Content hub solution

Search results in a content management system for "threat intel." There is a single search result titled "Threat Intelligence," labeled as "FEATURED" and "PREVIEW."

Step 2: Connect the Threat Intelligence data connector
Walkthrough (Click to see)

    After the installation is complete, select Manage to review the deployed Threat Intelligence (TI) connectors

Threat intelligence solution for Microsoft Sentinel with description.

    Select the Microsoft Defender Threat Intelligence data connector and open the connector page

Different threat intelligence data connectors. The columns are labeled "Content name," "Created content," "Content type," and "Version."

    Notice the Prerequisites and Configuration sections:
        Prerequisites will show the required permissions to configure the data connector.

Microsoft Defender Threat Intelligence (Preview) setup, displaying "Prerequisites" with details for workspace read/write permissions.

    For this data connector, only workspace-scoped permissions are needed, which you already have
    Other data connectors might have different permission requirements on various scopes depending on the integration nature of that data connector
    Click Connect to import Indicators of Compromise (IOCs) from Microsoft Defender Threat Intelligence (MDTI) into Microsoft Sentinel

Microsoft Defender Threat Intelligence (Preview) configuration screen in Microsoft Sentinel.

    Review the data connector's status to ensure it is connected

Microsoft Defender Threat Intelligence (Preview) interface. The status is connected, with the provider noted as Microsoft and the last log received currently unavailable.

    When the log data starts to flow, the Last Log Received will be updated (this might take 10-15min, but it's not required to wait for it)

Microsoft Defender Threat Intelligence (Preview) dashboard. The status is Connected, with Microsoft as the provider, and no data received for the last log. The data type listed is ThreatIntelligenceIndicator with no data received.

    Once the connection is established, the data type icon will turn to green, indicating that the corresponding log table is populated and now contains data in the Log Analytics workspace

A line graph titled "Data received" displays data from April 3 to April 9. The y-axis ranges from 0 to 340, while the x-axis lists the dates.

    Clicking on the table name will take you to the ThreatIntelligenceIndicator log table in the Log Analytics workspace

Microsoft Sentinel Logs interface displaying a result for "ThreatIntelligenceIndicator." The result includes information about a security alert generated by Microsoft Defender Threat Intelligence

    After completing this task, you should be able to:
        Connect a data connector
        Review its connection status and confirm that logs for that connector are being ingested successfully


