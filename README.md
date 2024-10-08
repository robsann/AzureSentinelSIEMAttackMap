<div align="justify">

# Attack Monitoring with Microsoft Sentinel (SIEM)

The purpose of this lab is to utilize Microsoft Sentinel to keep track of unsuccessful Remote Desktop Protocol (RDP) login attempts made by attackers worldwide on an exposed Windows 10 virtual machine set up in Microsoft Azure. The failed login events and geolocation data are extracted using a PowerShell script on the virtual machine, processed using the Log Analytics Workspace on Microsoft Azure, and visualized in Microsoft Sentinel on Microsoft Azure.

## Outline

1. [Procedure](#procedure)
2. [Diagram](#diagram)
3. [Setup Overview](#setup-overview)
	1. [Microsoft Azure](#microsoft-azure)
4. [Azure Log Analytics and Microsoft Sentinel](#azure-log-sentinel)
	1. [Azure Log Analytics Workspace](#azure-log-analytics-workspace)
	2. [Mirosoft Sentinel](#microsoft-sentinel)


----------------------------------------------------------------------------------------------------

## Procedure

The procedures to build this lab can be found [here](https://github.com/robsann/AzureSentinelSIEMAttackMap/blob/main/procedure.md), and it was adapted from [Josh Madakor](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=1544s&ab_channel=JoshMadakor-Tech%2CEducation%2CCareer).

## Diagram

<div align="center">
<img src="images/diagram.png" width="80%"/>
</div>


----------------------------------------------------------------------------------------------------


<h1 align="center" id="setup-overview">Setup Overview</h1>

## Microsoft Azure

This lab uses Microsoft Azure, which is a cloud computing platform that offers a wide range of services for building, deploying, and managing applications and services through Microsoft's global network of data centres. It provides tools and resources for businesses to scale and grow their operations in a secure and reliable environment.

<details>
<summary>
<h3>The Virtual Machine on Microsoft Azure</h3>
</summary>
<span style="color:gray">

### The Virtual Machine Configuration

Here is the information about the virtual machine created on Microsoft Azure. The virtual machine was named `honeypot-vm` and runs the Windows 10 operating system. Information about the virtual machine can be found on this screen, such as the computer name, operating system, public and private IP addresses, and the hardware used. Furthermore, the virtual machine can be accessed via RDP.

<img src="images/1-honeypot-vm.png" title="honepot-vm"/>

### The Windows Defender Firewall Disabled

The Windows Defender Firewall of the Windows 10 virtual machine was turned off for the Domain, Private, and Public profiles, allowing connections from outside.

<img src="images/2-windows-firewall.png" title="Windows Defender Firewall"/>

</span>
</details>


<details>
<summary>
<h3>The PowerShell Script</h3>
</summary>
<span style="color:gray">

### The PowerShell Script Main Sections

Here are some chunks of the PowerShell script utilized to retrieve unsuccessful login events from the Security Log in Event Viewer. The retrieved IP address is then sent to a third-party API to get geolocation information that will be combined with the event details to create a custom log file.

#### XML Filter

The XML filter used on the PowerShell script to filter failed login events (ID = 4625) on Event Viewer.

<img src="images/3a-xml-filter.png" title="XML filter"/>

#### Event Viewer Log Gathering

This section of the code extracts fields from the events filtered from the Security Log in Event Viewer by the XML filter.

<img src="images/3b-event-viwer-log-gather.png" title="Event Viwer log gather"/>

#### API Data Gathering

The IP address extracted from the Security Log in Event Viewer was used to make web requests to the geolocation API.

<img src="images/3c-api-data-gather.png" title="API data gather"/>


### The PowerShell Script Extracting Failed Login Attempts

Below is the PowerShell ISE, with the PowerShell script used to extract failed login attempts from Security Log in Event Viewer running and outputting to the terminal the detected failed login attempts. At the right is an example of a failed login attempt event on Event Viewer. Some relevant fields are `Account Name`, the used username, `Source Network Address`, the attacker IP address, `Event ID`, `Logged`, and `Computer`.

<img src="images/3-powershell-script.png" title="PowerShell script + Event Viwer"/>

### The Custom Log File Generated by the PowerShell Script

This text file is the custom log file (`failed_rdp.log`) generated by the PowerShell script. It has information about the failed RDP login attempt, such as the username attempted, the source IP address and the geolocation data gathered using the IP address. It is the file that is ingested by Log Analytics Workspace on Microsoft Azure.

<img src="images/3d-failed_rdp.log.png" title="Failed RDP Log"/>

</span>
</details>


----------------------------------------------------------------------------------------------------


<h1 align="center" id="azure-log-sentinel">Azure Log Analytics and Microsoft Sentinel</h1>

## Azure Log Analytics Workspace

Azure Log Analytics Workspace is a centralized repository for collecting, analysing, and visualizing log data from various sources within the Azure ecosystem. It provides insights into system performance, security threats, and operational efficiency for better decision-making.

<details>
<summary>
<h3>The Log Analytics Workspace Custom Table on Microsoft Azure</h3>
</summary>
<span style="color:gray">

The `failed_rdp.log` was imported in real-time from the virtual machine, and its content was stored in the `RawData` field of the created `FAILED_RDP_WITH_GEO_CL` table. A KQL query was used to extract the data from the `RawData` field and create new fields on the `FAILED_RDP_WITH_GEO_CL` table, named according to the extracted data.

<img src="images/4-law-honeypot1-logs.png" title="law-honeypot1-logs"/>

</span>
</details>


## Microsoft Sentinel

Microsoft Sentinel is a cloud-native security information and event management (SIEM) service that helps organizations detect, investigate, and respond to threats across their entire environment. It provides intelligent security analytics and threat intelligence to help protect against cyberattacks.

<details>
<summary>
<h3>The Microsoft Sentinel Map on Microsoft Azure</h3>
</summary>
<span style="color:gray">

The visualization of failed RDP login attempts on the world map was created on Microsoft Sentinel. The attackers' IP address and geolocation (latitude, longitude, and country) were obtained from the `FAILED_RDP_WITH_GEO_CL` table using a KQL query.

### After 1 Hour of Exposure.

During the first hour, two failed RDP login attempts were registered from me and three from the United Kingdom.

<img src="images/5a-sentinel-map-1h.png" title="Sentinel map 1h"/>

### After 24 Hours of Exposure

After 24 hours of exposure, the virtual machine had 844 failed RDP login attempts from India, 447 from Palestine, 338 from Japan, and 533 from other countries.

<img src="images/5b-sentinel-map-24h.png" title="Sentinel map 24h"/>

### After 48 Hours of Exposure

Over a 48-hour period, there were a significant number of failed RDP login attempts on the virtual machine, originating from various countries. There were 1.75k failed attempts from the Netherlands, 1.05k from Pakistan, 844 from India, and 3.09k from other countries.

<img src="images/5c-sentinel-map-48h.png" title="Sentinel map 48h"/>

### The Top 10 Usernames Used by Attackers

This bar plot displays the top 10 usernames most frequently used by attackers, obtained from the `FAILED_RDP_WITH_GEO_CL` table through a KQL query. The most commonly used username is `ADMINISTRATOR`, followed by `ADMIN`, `administrator`, `Administrator`, and `admin`.

<img src="images/6-TopUsernames.png" title="Top 10 Usernames"/>

</span>
</details>

</div>
