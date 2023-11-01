# Attack Monitoring with Microsoft Sentinel (SIEM)
<div align="justify">
The objective of this lab is to employ Microsoft Sentinel for monitoring unsuccessful Remote Desktop Protocol (RDP) login attempts globally. These attempts are made by attackers worldwide on an exposed Windows 10 virtual machine set up in Microsoft Azure. A PowerShell script on the virtual machine extracts failed login events and geolocation data. This information is then processed in the Log Analytics Workspace on Microsoft Azure and visualized within Microsoft Sentinel on the same platform.

## Summary
- Microsoft Sentinel was utilized to monitor repeated failed RDP login attempts originating from global attackers targeting an exposed Windows 10 virtual machine within Microsoft Azure.
- To achieve this, a dedicated log file (`failed_rdp.log`) was generated through a PowerShell script. This script extracted instances of failed login events from the Security Log in Event Viewer. Subsequently, the attacker's IP address extracted is sent to a third-party API to retrieve geolocation data.
- Within the Microsoft Azure Log Analytics Workspace, a custom table (`FAILED_RDP_WITH_GEO_CL`) was established. The table was populated with pertinent fields extracted from the aforementioned log file (`failed_rdp.log`) using a Kusto Query Language (KQL) query.
- To visualize this data effectively, a specialized workbook was created within Microsoft Sentinel. This workbook employed KQL queries to extract and display information from the `FAILED_RDP_WITH_GEO_CL` table. This visualization showcased global attackers involved in RDP login failures on a world map. The map indicated the physical locations of these attacks and their magnitudes (attack count).

## Tools
- **Kali Linux**
  - **Remmina (RDP Client)**
  - **Microsoft Azure**
    - **Windows 10 VM**
      - **PowerShell script**
    - **Log Analytics Workspace**
    - **Microsoft Sentinel**

## Procedure
The procedures to build this lab can be found [here](https://github.com/robsann/AzureSentinelSIEMAttackMap/blob/main/procedure.md). It was adapted from [Josh Madakor](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=1544s&ab_channel=JoshMadakor-Tech%2CEducation%2CCareer).

## Diagram
<img src="images/diagram.png" title="Lab Diagram"/>

# Highlighs

## 1 - The Virtual Machine on Microsoft Azure

### 1.1 - The Virtual Machine Configuration
Here is the information regarding the virtual machine established on Microsoft Azure. The virtual machine, named `honeypot-vm`, operates on the Windows 10 operating system. Details about this virtual machine are available on this screen, including the computer name, operating system, public and private IP addresses, and hardware specifications. Additionally, access to the virtual machine is possible through RDP.

<img src="images/1-honeypot-vm.png" title="honepot-vm"/>

### 1.2 - The Windows Defender Firewall Disabled
The Windows Defender Firewall on the Windows 10 virtual machine was disabled for the Domain, Private, and Public profiles, permitting external connections.

<img src="images/2-windows-firewall.png" title="Windows Defender Firewall"/>

## 2 - The PowerShell Script

### 2.1 - The PowerShell Script Main Sections
Below are various segments of the PowerShell script used to fetch unsuccessful login events from the Security Log within Event Viewer. The acquired IP address is subsequently transmitted to a third-party API to obtain geolocation information. This information, along with event details, is then merged to generate a customized log file.

#### XML Filter
The XML filter utilized in the PowerShell script filters failed login events (Event ID = 4625) in the Event Viewer.

<img src="images/3a-xml-filter.png" title="XML filter"/>

#### Event Viewer Log Gathering
In this section of the code, fields are extracted from the events filtered from the Security Log in Event Viewer using the XML filter.

<img src="images/3b-event-viwer-log-gather.png" title="Event Viwer log gather"/>

#### API Data Gathering
The IP address, extracted from the Security Log within Event Viewer, was utilized to initiate web requests to the geolocation API.

<img src="images/3c-api-data-gather.png" title="API data gather"/>

### 2.2 - The PowerShell Script Extracting Failed Login Attempts
Below is the PowerShell ISE, with the PowerShell script used to extract failed login attempts from Security Log in Event Viewer running and outputting to the terminal the detected failed login attempts. At the right is an example of a failed login attempt event on Event Viewer. Some relevant fields are `Account Name`, the used username, `Source Network Address`, the attacker IP address, `Event ID`, `Logged`, and `Computer`.

Below is the PowerShell ISE, with the PowerShell script used to extract failed login attempts from the Security Log in Event Viewer running and outputting to the terminal the detected failed login attempts. On the right is an example of a failed login attempt event in Event Viewer. Some relevant fields are `Account Name`, which shows the used username, `Source Network Address`, indicating the attacker's IP address, `Event ID`, `Logged`, and `Computer`.

<img src="images/3-powershell-script.png" title="PowerShell script + Event Viwer"/>

### 2.3 - The Custom Log File Generated by the PowerShell Script
This text file, named `failed_rdp.log`, is the custom log file created by the PowerShell script. It contains details about unsuccessful RDP login attempts, including the attempted username, source IP address, and geolocation data obtained from the IP address. This file is processed by the Log Analytics Workspace on Microsoft Azure.

<img src="images/3d-failed_rdp.log.png" title="Failed RDP Log"/>

## 3 - The Log Analytics Workspace Custom Table on Microsoft Azure
The `failed_rdp.log` file was imported in real-time from the virtual machine. Its contents were stored in the `RawData` field of the newly created `FAILED_RDP_WITH_GEO_CL` table. Using a KQL query, new fields were created on the `FAILED_RDP_WITH_GEO_CL` table by extracting data from the `RawData` field, which stores all the `failed_rdp.log` data.

<img src="images/4-law-honeypot1-logs.png" title="law-honeypot1-logs"/>

## 4 - The Microsoft Sentinal Map on Microsoft Azure
The Microsoft Sentinel platform was used to visualize unsuccessful Remote Desktop Protocol (RDP) login attempts worldwide. This visualization involved retrieving the attackers' IP addresses and geolocation data (latitude, longitude, and country) from the `FAILED_RDP_WITH_GEO_CL` table through a Kusto Query Language (KQL) query.

### 4.1 - After 1 Hour of Exposure.
During the initial hour, there were two unsuccessful RDP login attempts made by me and three made by individuals from the United Kingdom.

<img src="images/5a-sentinel-map-1h.png" title="Sentinel map 1h"/>

### 4.2 - After 24 Hours of Exposure
After 24 hours of exposure, the virtual machine experienced 844 unsuccessful RDP login attempts from India, 447 from Palestine, 338 from Japan, and 533 from various other countries.

<img src="images/5b-sentinel-map-24h.png" title="Sentinel map 24h"/>

### 4.3 - After 48 Hours of Exposure
Over a span of 48 hours, there was a notable surge in unsuccessful RDP login efforts on the virtual machine, originating from different countries. Specifically, there were 1.75k failed attempts from the Netherlands, 1.05k from Pakistan, 844 from India, and 3.09k from diverse other countries.

<img src="images/5c-sentinel-map-48h.png" title="Sentinel map 48h"/>

### 4.4 - The Top 10 Usernames Used by Attackers
The bar plot illustrates the top 10 usernames frequently utilized by attackers, extracted from the `FAILED_RDP_WITH_GEO_CL` table via a KQL query. The most prevalent username is `ADMINISTRATOR`, succeeded by `ADMIN`, `administrator`, `Administrator`, and `admin`.

<img src="images/6-TopUsernames.png" title="Top 10 Usernames"/>

</div>
