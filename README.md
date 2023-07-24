# Attack Monitoring with Microsoft Sentinel (SIEM)
The purpose of this lab is to utilize Microsoft Sentinel to keep track of unsuccessful Remote Desktop Protocol (RDP) login attempts made by attackers worldwide on an exposed Windows 10 virtual machine set up in Microsoft Azure. The failed login events and geolocation data are extracted using a PowerShell script on the virtual machine, processed using the Log Analytics Workspace on Microsoft Azure, and visualized in Microsoft Sentinel on Microsoft Azure.

## Summary
- Microsoft Sentinel was used to monitor failed RDP login attempts from global attackers on an exposed Windows 10 virtual machine configured in Microsoft Azure.
- A custom log file (`failed_rdp.log`) was generated using a PowerShell script that extracts failed login events from Security Log on Event Viewer and forwards them to a third-party API to get geolocation data.
- A custom table (`FAILED_RDP_WITH_GEO_CL`) was created in Log Analytics Workspace on Microsoft Azure using the generated log file (`failed_rdp.log`). Custom fields were extracted from the table using a Kusto Query Language (KQL) query.
- A workbook was created in Microsoft Sentinel using KQL to query data from the `FAILED_RDP_WITH_GEO_CL` table to display global attackers (RDP login failure) on the world map according to physical location and magnitude (attack count).

## Procedure
The procedures to build this lab can be found [here](https://github.com/robsann/AzureSentinelSIEMAttackMap/blob/main/procedure.md), and it was adapted from [Josh Madakor](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=1544s&ab_channel=JoshMadakor-Tech%2CEducation%2CCareer).

## Diagram
<img src="images/diagram.png" title="Lab Diagram"/>

# Highlighs

## 1 - The Virtual Machine on Microsoft Azure

### 1.1 - The Virtual Machine Configuration
Here is the information about the virtual machine created on Microsoft Azure. The virtual machine was named `honeypot-vm` and runs the Windows 10 operating system. Information about the virtual machine can be found on this screen, such as the computer name, operating system, public and private IP addresses, and the hardware used. Furthermore, the virtual machine can be accessed via RDP.

<img src="images/1-honeypot-vm.png" title="honepot-vm"/>

### 1.2 - The Windows Defender Firewall Disabled
The Windows Defender Firewall of the Windows 10 virtual machine was turned off for the Domain, Private, and Public profiles allowing connections from outside.

<img src="images/2-windows-firewall.png" title="Windows Defender Firewall"/>

## 2 - The PowerShell Script

### 2.1 - The PowerShell Script Main Sections
Here are some chunks of the PowerShell script utilized to retrieve unsuccessful login events from the Security Log in Event Viewer. The retrieved IP address is then sent to a third-party API to get geolocation information that will be combined with the event details to create a custom log file.

#### XML Filter
The XML filter used on the PowerShell script to filter failed login events (ID = 4625) on Event Viewer.

<img src="images/3a-xml-filter.png" title="XML filter"/>

#### Event Viewer Log Gathering
This section of the code extracts fields from the events filtered from the Security Log in Event Viewer by the XML filter.

<img src="images/3b-event-viwer-log-gather.png" title="Event Viwer log gather"/>

#### API Data Gathering
The IP address extracted from Security Log in Event Viewer was used to make web requests to the geolocation API.

<img src="images/3c-api-data-gather.png" title="API data gather"/>


### 2.2 - The PowerShell Script Extracting Failed Login Attempts
Below is the PowerShell ISE, with the PowerShell script used to extract failed login attempts from Security Log in Event Viewer running and outputting to the terminal the detected failed login attempts. At the right is an example of a failed login attempt event on Event Viewer. Some relevant fields are `Account Name`, the used username, `Source Network Address`, the attacker IP address, `Event ID`, `Logged`, and `Computer`.

<img src="images/3-powershell-script.png" title="PowerShell script + Event Viwer"/>

### 2.3 - The Custom Log File Generated by the PowerShell Script
This text file is the custom log file (`failed_rdp.log`) generated by the PowerShell script. It has information about the failed RDP login attempt, such as the username attempted, the source IP address, and the geolocation data gathered using the IP address. It is the file that is ingested by Log Analytics Workspace on Microsoft Azure.

<img src="images/3d-failed_rdp.log.png" title="Failed RDP Log"/>

## 3 - The Log Analytics Workspace Custom Table on Microsoft Azure
The `failed_rdp.log` was imported in real-time from the virtual machine, and its content was stored in the `RawData` field of the created `FAILED_RDP_WITH_GEO_CL` table. A KQL query was used to extract the data from the `RawData` field and create new fields on the `FAILED_RDP_WITH_GEO_CL` table, named accordingly to the extracted data.

<img src="images/4-law-honeypot1-logs.png" title="law-honeypot1-logs"/>

## 4 - The Microsoft Sentinal Map on Microsoft Azure
The visualization of failed RDP login attempts on the world map was created on Microsoft Sentinel. The attackers' IP address and geolocation (latitude, longitude, and country) were obtained from the `FAILED_RDP_WITH_GEO_CL` table using a KQL query.

### 4.1 - After 1 Hour of Exposure.
During the first hour, two failed RDP login attempts were registered from me and three from the United Kingdom.

<img src="images/5a-sentinel-map-1h.png" title="Sentinel map 1h"/>

### 4.2 - After 24 Hours of Exposure
After 24 hours of exposure, the virtual machine had 844 failed RDP login attempts from India, 447 from Palestine, 338 from Japan, and 533 from other countries.

<img src="images/5b-sentinel-map-24h.png" title="Sentinel map 24h"/>

### 4.3 - After 48 Hours of Exposure
Over a 48-hour period, there were a significant number of failed RDP login attempts on the virtual machine, originating from various countries. There were 1.75k failed attempts from Netherlands, 1.05k from Pakistan, 844 from India, and 3.09k from other countries.

<img src="images/5c-sentinel-map-48h.png" title="Sentinel map 48h"/>

### 4.4 - The Top 10 Usernames Used by Attackers
This bar plot displays the top 10 usernames most frequently used by attackers, obtained from the `FAILED_RDP_WITH_GEO_CL` table through a KQL query. The most commonly used username is `ADMINISTRATOR`, followed by `ADMIN`, `administrator`, `Administrator`, and `admin`.

<img src="images/6-TopUsernames.png" title="Top 10 Usernames"/>
