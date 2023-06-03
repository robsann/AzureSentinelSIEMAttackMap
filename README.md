## Azure Sentinel (SIEM) Attack Map
- Configured an exposed Windows 10 VM in Azure to monitor failed RDP login attempts from Global Attackers using Azure Sentinel (SIEM).
- Configured a Windows 10 VM with Firewall disabled and RDP (3389) port open and used a custom PowerShell script to extract metadata from Windows Event Viewer and forward it to a 3rd party API to get geolocation data.
- Configured a custom log in Log Analytics workspaces on Azure to ingest custom logs containing geographic information (latitude, longitude, state, and country) and extracted the fields using Kusto Query Language (KQL) to map geo data into Azure Sentinel.
- Configured an Azure Sentinel (SIEM) workbook to display global attack data (failed RDP login attempts) on the world map according to physical location and magnitude (count) of attacks.

The procedures to build this lab can be found [here](https://github.com/robsann/AzureSentinelSIEMAttackMap/blob/main/procedure.md).

This lab was adapted from [here](https://www.youtube.com/watch?v=RoZeVbbZ0o0&t=1544s&ab_channel=JoshMadakor-Tech%2CEducation%2CCareer).

## Highlighs
