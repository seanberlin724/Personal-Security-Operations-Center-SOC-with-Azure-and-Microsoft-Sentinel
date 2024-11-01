# Personal Security Operations Center (SOC) with Azure and Microsoft Sentinel

## Objective
To develop and enhance a cloud-based Security Operations Center (SOC) using Azure Virtual Machines, Microsoft Sentinel, and integrated threat intelligence tools, simulating real-world security monitoring and advanced threat detection processes. This project includes configuring a SIEM system, monitoring RDP access, implementing custom detection rules, integrating external threat intelligence feeds (MISP), and exploring automation to create alerts for high-confidence threats.


### Skills Learned
- Configuring and managing cloud-based virtual machines (Azure VM)
- Setting up and operating a SIEM system with Microsoft Sentinel for proactive threat monitoring
- Integrating threat intelligence feeds with Microsoft Sentinel via API (MISP2Sentinel)
- Developing custom detection rules and queries for specific threat indicators
- Deploying Docker containers and managing APIs to automate threat ingestion
- Using Log Analytics Workspace for centralized data ingestion and in-depth threat analysis
- Configuring automation for real-time responses to security alerts

### Tools Used
- **Azure** (Virtual Machines, Log Analytics Workspace, Azure CLI, Key Vault)
- **Microsoft Sentinel** (SIEM system, Analytics, Incidents, Content Hub)
- **RDP** (Remote Desktop Protocol) for access simulation and security testing
- **MISP2Sentinel Data Connector** for threat intelligence integration
- **Docker** for MISP deployment on Azure VM
- **Log Analytics** for centralized data collection and analysis
- **Azure Function Apps** for automated data processing and alert response
- **Visual Studio Code** for code modifications and deployment of custom applications

## Steps
### 1. Set up Azure Virtual Machine
- Created a Windows-based Virtual Machine in Azure, using the free trial credits for deployment.
- Exposed RDP (Remote Desktop Protocol) access to simulate potential attacks and generate security events for monitoring.
<img src="Images/1.png">

*Ref 1: Azure Virtual Machine Overview Screenshot*


### 2. Configure Microsoft Sentinel
- Deployed Microsoft Sentinel on Azure and linked it to the Log Analytics Workspace to begin ingesting security event logs from the VM.
- Connected the VM’s security logs to Sentinel using Data Connectors, enabling log collection for analysis.
<img src="Images/2.png">

*Ref 2: Microsoft Sentinel Dashboard Screenshot*

### 3. Create Custom Detection Rules for RDP Sign-ins
- Configured custom detection rules in Microsoft Sentinel to monitor and alert on successful RDP sign-ins.
- The rules triggered real-time alerts, allowing for immediate detection of suspicious login attempts.
<img src="Images/3a.png">

*Ref 3: Custom Detection Rule Setup Screenshot 1/2*

<img src="Images/3b.png">

*Ref 3: Custom Detection Rule Setup Screenshot 2/2*

### 4. Real-Time Alerts and Analytics
- Demonstrated real-time incident generation in Microsoft Sentinel by attempting RDP logins. The successful attempts were flagged and alerted by the custom detection rules set up in the previous step.
- Incidents were generated for both successful logins via RDP, simulating a real-world security scenario (i.e. brute force attack).
<img src="Images/4.png">

*Ref 4: Microsoft Sentinel Analytics Screenshot*

### 5. Log Analytics Workspace Configuration
- Linked the Azure Virtual Machine to the Log Analytics Workspace to centralize data collection.
- Optimized data ingestion and monitoring workflows, ensuring that security events from the VM were available for analysis in real-time.
<img src="Images/5a.png">

*Ref 5:  Microsoft Sentinel Incidents Screenshot*

<img src="Images/5b.png">

*Ref 6: Incident Details Screenshot*

### 6. Integrate threat intelligence indicators through API calls for more comprehensive security monitoring
- The next series of events will entail setting up a threat intelligence feed and sending it into the SIEM utilizing Microsoft's prepackaged data connectors.
- To start, I installed the MISP2Sentinel Data Connector from the Content Hub.
- MISP is an open source threat intelligence platform. They utilize various threat indicators pulled from various partnered companies. The default (free) feeds are described in a JSON format. This will come into play when importing the data.
- 
<img src="Images/Install MISP2Sentinel Data Connector.png">

*Ref 7: Install MISP2Sentinel Data Connector*

- To continue this process, I created a new VM to host a Docker container to import the data from MISP.
- Below is the overview of the VM for documentation purposes.

<img src="Images/VM2 Overview.png">

*Ref 8: VM2 Overview*


- I then set up SSH using Azure CLI to connect to the VM.

<img src="Images/Connect to VM2 through SSH.png">

*Ref 9: Connect to VM2 through SSH*

- I then installed Docker using the following commands from the Docker website

<img src="Images/Docker Install Commands.png">

*Ref 10: Docker Install Commands*

- The following screenshot verifies that the Docker Engine installation is successful by running the hello-world image.
<img src="Images/Verify Docker Installation.png">

*Ref 11: Verify Docker Installation*



- Next, I cloned the MISP Docker Portfolio using the command "$ git clone https://github.com/MISP/misp-docker"
- I then copied the template.env to .env and modified .env to set the value of the "BASE_URL" variable to the IP of the VM.
- I then performed the listed set of commands below from the GitHub portfolio

<img src="Images/Docker Run Commands.png">

*Ref 12: Docker Run Commands*



- Under the Networking section of the VM (Ubuntu VM), I opened up port 443 to allow the connection to be established from anywhere.
  
<img src="Images/Open Port 443 on VM2.png">

*Ref 13: Open Port 443 on VM2*


- Next, I logged into the MISP portal via a normal web browser on my host machine. Before any other steps, I first changed the default password used to login.
- The next step was to enable the feeds on the website.
- I started by copying the JSON file from the MISP GitHub of "https://github.com/MISP/MISP/blob/2.4/app/files/feed-metadata/defaults.json"
- I then went to my personal MISP portal and enabled the threat indicators by pasting the previously copied JSON content into the Feeds section and selecting enabled on all the pages available.
- This enables the ability to select "fetch and store all feed data".

<img src="Images/Paste JSON into Import Feeds.png">

*Ref 14: Paste JSON into Import Feeds*



- Next, I set up a data connector into the Microsoft Sentinel to now be pulling these threat indicators in.
- To start, I installed MISP into Microsoft Sentinel by using the Upload Indicators API.
- I registered a new application the Azure tenant to service the MISP to Microsoft Sentinel connection.

<img src="Images/Register MISP to Sentinel.png">

*Ref 14: Register MISP to Sentinel*



I then created a new client secret for MISP.
Next, I added the Microsoft Sentinel contributor role to the Azure app that was just created. This allows the app to access the workspace.
To do this, I navigated to the Log Analytics Workspace where the Microsoft Sentinel instance was created and granted a role that allows the app to access Microsoft Sentinel.

<img src="Images/Grant Role to App.png">

*Ref 15: Grant Role to App*



I then created a Key Vault to store the MISP API key.
Then, to get the MISP API Key, which is needed to allow the function app to connect to the MISP instance and allow it to pull the indicators to send it over to Microsoft Sentinel, I navigated to Administration in the MISP web portal in List Auth Keys. I then added an authentication key and copied it down as the entire ID will not be able to be retrieved beyond this point.

<img src="Images/Authentication Key Index.png">

*Ref 16: Authentication Key Index*


Next, I created a function app rather than needing to create a function on the MISP instance itself on the Ubuntu server. In other words, the function app will be used to pull from MISP directly into Microsoft Sentinel. 
Note: The function app uses "App Service" as the hosting option and is not part of the Microsoft Azure Free Subscription. For example, I upgraded to the Basic Subscription 

<img src="Images/Set Up Function App.png">

*Ref 17: Set Up Function App*


Next, I created a function, but I created it on my desktop rather than utilizing the Azure portal. I downloaded and opened the following GitHub repository into Visual Studio Code: "https://github.com/cudeso/misp2sentinel". Note: Be sure to have the Python and Python Debugger extensions installed.
I then signed into my Azure account using VS Code.
The two files that I changed were "config.py" and the "_init_.py".
The reasoning for the modifications is because the instructions (in the function app) do not match the python code.

Below is the modification made to the "_init_.py". This file is the function that runs to pull things from the MISP instance using the API key and then sending it into the Microsoft Sentinel data connector. The deletion of the following lines of code prevents an error as this is a Single-tenant and not a Multi-tenant. To be clear, delete the code from line 105 to 110.

<img src="Images/_init_py Change.png">

*Ref 18: _init_py Change*

Below is the modification made to the "config.py" file on line 40.

<img src="Images/config Change.png">

*Ref 19: config Change*



I then saved the changes and deployed the function app

<img src="Images/Successful Function App Deployment from VS Code.png">

*Ref 20: Successful Function App Deployment from VS Code*



The next step involved adjusting the environmental variables and setting everything that is going to be called from that function.
The reason for this is that in the "config.py" file, the ms_auth function has API settings that need to be set as highlighted below.

<img src="Images/config Environmental Variables.png">

*Ref 20: config Environmental Variables*


I then added the below Environmental Variables in the Function App as well as additional ones such as "AzureFunctionsJobHost_functionTimeout" and "timeTriggerSchedule".
These were done due to instructions given on the GitHub portfolio.

<img src="Images/Environmental Variables 1 of 2.png">

*Ref 21: Environmental Variables 1 of 2.png*

<img src="Images/Environmental Variables 2 of 2.png">

*Ref 22:  Environmental Variables 2 of 2*


Next, I tested the Function App live to make sure it is properly running rather than waiting for the Azure results as they are frequently delayed. 
To do this, I viewed the Function App Logs to check for any errors. Fortunately, no errors were found as the proper environmental variables were being pulled.

<img src="Images/Function App Test Logs.png">

*Ref 23:  Function App Test Logs*

The MISP2Sentinel data connector in Microsoft Sentinel should now have the Status of Connected if everything is running properly.

<img src="Images/MISP2Sentinel Connected.png">

*Ref 24:  MISP2Sentinel Connected*

Threat Intelligence should now be correctly working and be viewable in Microsoft Sentinel as well.

<img src="Images/Microsoft Sentinel Threat Intelligence.png">

*Ref 25:  Microsoft Sentinel Threat Intelligence*


Furthermore, Threat Intelligence Indicators as a table that can be used to create alerts

<img src="Images/Threat Intelligence Indicator Table.png">

*Ref 25:  Threat Intelligence Indicator Table*

For example, if you wanted to look for a Threat Intelligence Indicator with a one hundred percent confidence rate (likely to be a real threat). 
Additionally, network IP addresses can be used to identify security events containing an IP address from a predetermined list. 
In this case, I created a security event that will be triggered when it encounters a network IP with a 100 ConfidenceScore. This query can then be put into an Analytics Rule to create custom detections.

<img src="Images/Threat Intelligence Indicator Query.png">

*Ref 26:  Threat Intelligence Indicator Query.png*

